---
title: "Rebuilding a Platform Under Itself: Monolith to Microservices Without Stopping the Business"
date: 2026-08-05
draft: false
tags: ["architecture", "laravel", "microservices", "migration", "event-sourcing"]
categories: ["architecture"]
summary: "Notes from a live V2→V3 re-platforming, moving a tightly-coupled Node/Python duolith to Laravel microservices while the old system keeps serving customers."
showToc: true
---

For the past year I've been leading a full re-platforming at CapStellar: moving a production system to a Laravel microservices architecture (V3) while the old one keeps serving customers and moving real money. The old system (V2) usually gets called a monolith, and that's always bothered me, because it was two apps: a Node.js one and a Python one, coupled by crons firing API calls into each other, through endpoints that trusted their callers far more than they should have. Two apps, zero boundaries. A duolith.

![Two identical Spider-Men pointing at each other, one labeled Node.js app, the other Python app](/images/blog/duolith-spiderman.jpg)

The migration itself: no rewrite-in-a-cave, no big-bang cutover, no heroics. (Fine, one weekend of heroics. We'll get there.) These are my working notes on what's carried it.

## The destination, and why Laravel

V3 is a core API plus a small set of focused services (portfolio execution, job dispatch and routing, market index data, and gamification) fronted by a family of React SPAs, a mobile app, and a browser extension. Four services, each existing for an operational reason; "microservices" as a lifestyle was never the goal.

Choosing Laravel for a crypto platform raises eyebrows. Let them rise: a three-person team ships fastest on a batteries-included framework with a mature ecosystem, and that means queues and workers (Horizon), server management (Forge), serverless (Cloud), admin panels, testing. The scarce resource in a small company is engineer-hours and operational attention, never CPU. Boring technology the team knows cold is a performance optimization.

## Strangler, enforced

The strategy is the classic strangler pattern with one hard rule that made it real: V2 is maintenance-only. No new features land on the duolith, full stop. Every product request becomes a forcing function for V3: either the flow exists there, or building it there is the work. Without that rule, "migration" turns into "two products forever". Ask anyone who's been there.

Flows move one at a time: build in V3, run alongside, cut over, leave V2's version dormant until confidence is high. Meanwhile the systems interoperate, with shared identities and events bridging old and new, because customers do not care which stack served their request.

{{< figure src="/images/blog/strangler-fig.jpg" alt="A strangler fig engulfing a temple ruin at Wat Phra Si Sanphet, Ayutthaya" caption="The pattern's namesake doing its thing to a temple in Ayutthaya. V2 is the temple. ([Wikimedia Commons](https://commons.wikimedia.org/wiki/File:Ayudhaya_wat_pra_sri_sanphet_005.JPG), public domain)" >}}

## Service boundaries follow operations

Every one of our service boundaries earns its network hop operationally:

- **Portfolio execution** is separate because it holds the most sensitive credentials and must run from specific egress IPs for exchange whitelisting. It's an isolation and blast-radius boundary.
- **Dispatch/routing** is separate because it's the traffic cop between the API and execution, a choke point you want independently deployable and independently scalable.
- **Index data** is separate because it's a scheduled pipeline with a different failure profile. It can be down for an hour without customer impact, and it must not be able to take the API down with it.
- **Gamification** is separate because it's optional. It should be killable on the worst day without a second thought. (Sorry, gamification.)

Nothing was extracted because a domain diagram said so. "Would I want to deploy, scale, or lose this independently?" is the only boundary test that has held up.

## The money path is event-sourced

Portfolio operations, the part that touches customer funds, is event-sourced: every state change is an immutable event, and current state is a projection. In a regulated financial context this pays for itself three ways. The audit trail is complete by construction instead of by logging discipline. "Why is this account in this state?" can be answered by replaying history. And new read models can be added without touching the write path.

The migration's most valuable side effect came from exactly this. V3's event history and observability surfaced data-hygiene problems that were invisible under V2: stale accounts that should have been archived, implicit state that existed only in a support tool's memory. A rebuild doesn't just carry your data forward. It makes your data tell the truth, and some of those truths are chores you're finally forced to schedule.

## Write the decisions down

The migration produced an architecture docs hub well past a hundred documents: ADRs for every consequential decision, migration runbooks, service maps. The ADRs earn their keep every time someone asks "why don't we just..." and a two-year-old decision record answers with the trade-offs already weighed. In a small team, the alternative to writing it down is re-litigating the same argument every quarter.

Continuous delivery is the other half of that discipline: deploy-on-merge, feature flags, and automated tests mean the migration advances in hundreds of small production-visible steps per month rather than a terrifying quarterly release. Small steps make a risky project boring, and boring is the goal.

## The unplanned final exam

Remember that weekend of heroics? No plan survives contact with reality, and ours met it early. External events (a regulatory deadline in our market) forced parts of V3 into production months ahead of the hardening schedule. The new execution engine's first production run happened under emergency conditions, over a weekend, mid-infrastructure-migration.

<img class="meme" src="/images/blog/no-option-test-in-production.jpg" alt="A Chernobyl control-room engineer, resigned: I see no other option but to test in production">

It held. Hundreds of customer portfolios were processed with zero incorrect operations.

I'd love to claim foresight. The lesson is narrower: the engine held because the migration had been continuous instead of big-bang. The code had been shipping for months, the events were flowing, and the observability already existed. A strangler migration means your new system is never more than one hard week from being real. Plan as if the cutover date is yours to choose; build as if it isn't.

## If you're staring at a similar migration

Freeze the old system first; every exception you grant extends the timeline by more than it seems. Draw boundaries where operations demand them, not where the domain diagram looks prettiest. Event-source the flows you'll be audited on, and only those. Write decisions down while they're cheap. And get the new system into production earlier than feels comfortable, because the date it must work by may not be on your calendar.

Go ahead, start strangling. You'll thank yourself down the road.
