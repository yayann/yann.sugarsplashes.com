---
title: "One in Three Tickets, Zero Human Touch: Lessons From Building an AI Support Agent"
date: 2026-07-09
draft: false
tags: ["ai", "llm", "rag", "automation", "support"]
categories: ["ai"]
summary: "Lessons from a year of running a production AI support agent at a regulated fintech: architecture, confidence scoring, and a 4x ticket surge."
showToc: true
---

At CapStellar I built and run an AI customer-support agent for a regulated digital-assets platform. As of this summer it resolves roughly one in three tickets end-to-end with no human touching them, and it drafts a first reply for everything else. A bot answering your customers about their money: what could possibly go wrong? Quite a lot, which is why this post is mostly about the guardrails.

## The architecture, briefly

The agent is built on Claude with two additions that do most of the heavy lifting:

- **Retrieval over a curated knowledge base.** Not the whole wiki: a deliberately maintained set of support articles, product facts, and policy answers, embedded and searched at question time. Curation is the unglamorous 80% of the job. When the agent answers badly, the fix is almost always in the knowledge base, not the prompt.
- **Tool access with hard boundaries.** The agent can look up the state of the customer's account (subscription, recent activity, onboarding progress) through a small set of read-only tools. It answers "why hasn't X happened?" with the customer's actual data instead of a shrug. That's the difference between an agent and a chatbot. It cannot change anything, and that's a feature (more on that later).

Orchestration runs through workflow automation (n8n in our case), which handles the ticket lifecycle: classification, retrieval, drafting, and the one decision that actually matters. Send it, or show it to a human first?

{{< figure src="/images/blog/hector-brain-blurred.jpg" alt="A sprawling n8n workflow canvas of blurred nodes and connections" caption="The agent's brain, an n8n canvas. Same blur policy as everything else in production." >}}

## Confidence is the product

The most important design decision was refusing to make "reply automatically" the default. Every draft gets a confidence score, and only high-confidence answers on recognized topics go out without review. Everything else becomes a draft that a human approves, edits, or bins.

That threshold is what makes the ~31% figure one I can defend with a straight face. The other ~69% of tickets still involve a person, but the agent has already read the ticket, pulled the account context, and written the reply they start from. The human cost per ticket collapses even where automation doesn't fully take over.

If you take one thing from this post, take this: chase calibrated confidence rather than the automation rate. The rate follows.

<img class="meme" src="/images/blog/chase-calibrated-confidence.jpg" alt="Drake meme: Drake recoils from 'chase automation rate' and approves of 'chase calibrated confidence'">

## The stress test we didn't schedule

In mid-2026, European crypto regulation forced a large exchange migration for our customers, and ticket volume roughly quadrupled in a month (~185/month to 644 at the peak). Our auto-reply had gone live days before the wave hit. 

Great timing, zero planning involved. <img class="gif-inline" src="/images/blog/wow-omg.gif" alt="Wow OMG reaction GIF">

Two things happened that taught me more than any benchmark:

1. The support team absorbed the surge without adding headcount, on the automation rather than on heroics.
2. The agent's confidence went down during the surge. The migration produced ticket types it had never seen, so it escalated more and auto-sent less. That's a calibrated system doing its job.

If your agent's confidence never drops on novel questions, the score is decoration.

## Lessons I'd give anyone building one

- **Ground answers in customer state, not just documents.** Retrieval answers "what is the policy"; tools answer "what happened to this account." Most frustrated tickets are the second kind.
- **Make escalation cheap and shameless.** The agent gets no reward for guessing. An escalated ticket that arrives with a good draft attached is a win.
- **Measure per topic, not globally.** Our global number hides the truth: mature topics resolve far above 31%, novel topics near zero. Per-topic metrics show where the next knowledge-base investment pays.
- **In a regulated business, read-only tools are a feature.** Auditors and executives both sleep better knowing the agent can see everything and change nothing. (Told you we'd get back to that.)
- **Expect the win to show up as leverage, not layoffs.** Nobody was replaced. The team handled four times the volume and now spends its time on the tickets that need a human.

## What's next

The current frontier for us is evaluation: regression suites for answer quality, so knowledge-base and prompt changes ship with the same confidence as code. Year one was making the agent useful. Year two is making it boring, in the best possible sense.

Go build one. Just wire the escalation path before the auto-send button, and your support team might even like it.
