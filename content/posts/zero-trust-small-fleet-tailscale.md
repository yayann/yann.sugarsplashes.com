---
title: "Zero-Trust Networking for a Small Multi-Provider Fleet"
date: 2026-07-21
draft: false
tags: ["infrastructure", "tailscale", "security", "networking", "devops"]
categories: ["infrastructure"]
summary: "How a Tailscale mesh turned servers on four hosting providers into one private network, and the operational rules that keep it clean: tags, key expiry, explicit public holes."
showToc: true
---

Our servers live on four different hosting providers. On purpose (exchange API whitelisting, price/performance, not marrying one vendor), but it means there is no shared VPC, no private backbone, no security group that spans the fleet. The default posture for a fleet like that is grim: every admin surface ends up on a public IP with a firewall rule and hope.

Hope is not a security control. <img class="gif-inline gif-lg" src="/images/blog/hope.gif" alt="Red from Shawshank: hope is a dangerous thing. Hope can drive a man insane." title="Hope is a dangerous thing. Hope can drive a man insane.">

We replaced that with a Tailscale mesh. Every server joins a WireGuard-based private network, internal traffic flows over it, and public exposure becomes an exception you grant explicitly instead of a default you forget to revoke. A year in, here's what I'd tell someone setting this up. Mostly the operational rules, because the tech itself is the easy part.

## What it looks like

Every node gets a stable private address and a DNS name (MagicDNS), regardless of provider. The API servers in one region talk to the trading bots at two other providers as if they shared a rack. Internal service auth still exists on top, since the mesh is transport rather than authorization, but nothing internal listens on a public interface anymore.

{{< figure src="/images/blog/tailscale-nodes-everywhere.jpg" alt="Buzz Lightyear to Woody: tailscale nodes, tailscale nodes everywhere" caption="Eleven years after [the SSH keys edition](/posts/automating-deployment-chatbot/). Some templates never expire, unlike node keys." >}}

The pieces that matter beyond the basics:

- **Tags are the org chart.** Servers carry ACL tags (`api`, `rebalancing`, `monitoring`, `staging`, a provider tag); people's laptops and phones join untagged. That one convention makes everything downstream scriptable. Our monitoring console discovers fleet machines by "has a tag" and never maintains a list of names.
- **Internal-only web apps use `tailscale serve`.** Our operations console is served with TLS on the tailnet only: no public DNS record, no public port, no VPN client config to distribute. For internal admin panels I know of nothing with a better convenience-to-exposure ratio.
- **Public ingress is an explicit, narrow hole.** The same console needs to receive webhooks from external SaaS providers. Rather than exposing the box, a Cloudflare Tunnel publishes exactly one path prefix, the webhook route, to the outside world. Everything else stays private. Private by default; then punch holes you can enumerate.

## The rules that keep it from decaying

The mesh itself is nearly maintenance-free. The decay happens at the edges, and each of these rules exists because something bit us.

**Disable key expiry on every server node.** By default a node's key expires about 180 days after joining, and when it does the machine silently drops off the network. For a laptop that's a login prompt; for a server it's a 2am mystery. Disabling expiry is one click per machine, and the trick is remembering to click it. We audited the fleet once, brought every node into compliance, and our monitoring now warns when any node's key expires within 14 days. In practice that warning means someone joined a server and skipped the checklist.

**Auth keys and API tokens are different species.** Auth keys enroll devices; API access tokens talk to the admin API. Automation needs the latter, they expire on their own schedule, and confusing the two costs you an afternoon. Put token rotation on the calendar. Our console flags the collector going `unknown`, which is the polite way to find out.

**Write the joining steps down.** Join the mesh, apply tags, disable key expiry, register in monitoring, install the heartbeat agent. Five steps, two minutes, and every skipped step surfaces weeks later as a gap somebody trips over. We keep known gaps visible on the console: machines not yet on the mesh are listed rather than forgotten.

**The mesh doesn't replace the firewall.** Public SSH on some boxes, cloud-provider firewalls, WAF rules: all still there, all still audited. Zero-trust networking shrinks the attack surface without granting immunity. Treat "it's on the tailnet" as one layer of several.

## The trade-offs

You're trusting a coordination-plane vendor. The data plane is peer-to-peer WireGuard, but device auth and key distribution run through their control plane. Some features sit behind paid plans; public funneling wasn't available on ours, which pushed us to the Cloudflare Tunnel pattern, and in hindsight I prefer the explicitness. And DNS gets a second namespace to keep in your head.

Against that: we deleted every "admin panel on a public IP" from the estate, cross-provider traffic stopped transiting the public internet, and onboarding a new machine takes two minutes. For a small team running infrastructure across multiple providers, I don't know a higher-leverage security move.

Give it a spin. Your future 2am self will thank you.
