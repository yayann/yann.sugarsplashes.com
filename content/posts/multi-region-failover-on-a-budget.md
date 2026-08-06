---
title: "Multi-Region Failover on a Small-Company Budget"
date: 2026-07-23
draft: false
tags: ["infrastructure", "cloudflare", "mysql", "high-availability", "laravel"]
categories: ["infrastructure"]
summary: "How we gave a Laravel API a warm standby on another continent with a Cloudflare Load Balancer, a read replica, and a short list of deliberate rules. It is not active-active, and that's the point."
showToc: true
---

Most multi-region writeups assume a platform team and a seven-figure cloud bill. We have neither. This is the other version: how we gave our production API a warm standby in another region with a Cloudflare Load Balancer, one extra VM, one read replica, and a short list of deliberate operating rules. It survives an app-tier region outage. It does not pretend to be active-active. Both halves of that sentence matter.

## The shape of it

- **Primary region:** a regional load balancer in front of two app VMs, plus the managed MySQL primary and the rest of the data layer (Redis, MongoDB).
- **Standby region, a continent away:** a single app VM running the same codebase on Octane/FrankenPHP, deploying from the same branch, plus a MySQL read-only replica.
- **In front of everything:** a Cloudflare Load Balancer on the public API hostname with two pools, primary and standby, and traffic steering off. That means simple priority failover: all traffic goes to the primary pool while its health monitor passes, and the standby only receives traffic when the primary is down. Failback is automatic.

{{< figure src="/images/blog/failover-topology.svg" alt="Topology diagram: Cloudflare Load Balancer over a primary region pool and a cross-continent standby, with writes crossing the gap to the primary MySQL" caption="The whole thing. Green carries traffic, orange pays the cross-continent toll." >}}

Because the load balancer took over the existing hostname, every WAF rule scoped to it kept working; nothing downstream noticed. (The best kind of migration.) Both origins receive the same Host header, so the framework generates identical URLs in either region.

## The health check is a design decision

The Cloudflare monitor polls a health endpoint on both pools every minute. That endpoint is deliberately boring: the framework's built-in liveness route, which renders a static response and touches no dependency. Our real health checks (database, Redis, queue depth) run elsewhere, on a schedule, feeding a monitoring console.

The distinction matters. A failover check answers "can this origin serve traffic", not "is everything this origin can reach happy". Wire your deep health check into the failover monitor and a database blip in the primary region can fail you over to a standby that has the same problem, plus a cross-continent round trip.

Know the detection lag, too: with a one-minute monitor interval and retries, a real outage means roughly one to three minutes of errors before failover engages. Health-check failover buys you short downtime at low cost, never zero downtime.

## Data: writes go home, reads stay local

The database policy fits in one sentence: writes always go to the primary region, and reads in the standby hit its local replica.

Laravel makes this almost free: separate read/write hosts on one connection, with `sticky` enabled so a request that just wrote reads its own write back. The env vars default to the primary host, so every environment without a replica is untouched by the change.

Without the local replica, every query from the standby pays the ~130–150ms of cross-continent round trip we measured between the two regions; with it, reads are local and only writes pay the toll. Replication is async, so cross-request staleness (save, redirect, list) is possible under lag. We accept that rather than build distributed-systems machinery a four-person company doesn't need.

One gotcha from the trenches: our replica's hostname still carries the region it was created in, not the region it lives in now (managed providers don't rename databases when you migrate them, apparently). Trust the panel, not the hostname. Related: a replica has its own firewall rules and its own CA cert. An "access denied" that works against the primary is usually a credentials problem, because firewall failures time out instead.

## The warm-standby rules (deliberate, do not "fix")

The standby runs web traffic only. These rules look like omissions; each one is there because removing it hurts:

- **No queue workers on the standby.** Workers there would consume production jobs cross-region during normal operation. The standby would be participating when it should be waiting.
- **No scheduler.** Exactly one region runs scheduled tasks. Two regions running the same cron is an incident generator.
- **Local cache.** The standby's cache is its own small Redis rather than the primary region's. The standby must not depend on primary-region infrastructure in the very scenario it exists for, and a cache that costs 150ms per operation defeats its purpose. On failover we lose rate-limiter counters. We'll live.

<img class="meme" src="/images/blog/standby-two-buttons.jpg" alt="Two-buttons meme: a sweating man must choose between the buttons ''fix' the standby by adding queue workers' and 'trust the runbook that says don't'">

## Why failover-only instead of active-active

The data has gravity. With the primary database in one region, actively serving users "closer" from the standby would make their requests slower: every write and every non-replicated read pays the cross-region round trip. Geo-steering only pays off when the data layer has a regional story, and ours doesn't need one yet.

That also defines what this setup is not: if the primary region's database is gone for a prolonged period, that's replica promotion, a disaster-recovery playbook rather than a load-balancer feature. Knowing precisely where your failover ends is as valuable as having it.

## What it costs

One VM, one read replica, and a Cloudflare Load Balancer subscription. Add the discipline to smoke-test the standby regularly (a direct test hostname bypassing the LB makes that a two-minute browser check) and pool-health notifications so we notice if the standby rots. A warm standby you never look at is a cold standby with better marketing.

![Anakin and Padmé four-panel meme. Anakin: we have a warm standby in another region. Padmé, smiling: and you test it regularly, right? Anakin stares. Padmé, worried: you test it regularly, right?](/images/blog/warm-standby-anakin.jpg)
