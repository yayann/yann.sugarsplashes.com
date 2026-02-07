---
title: "Using State Machines"
date: 2015-05-10
tags: ["laravel", "php", "statemachine", "yii"]
categories: ["php-development"]
summary: "How state machines can save you from workflow hell, with practical tips and library recommendations for PHP and JavaScript."
showToc: true
---

I worked on projects requiring workflows pretty early in my career. You know, that thing supposed to *"streamline work processes"*? but everyone goes *fuck it, I'll send it via email*? Yeah, it didn't start out well, but it got better for me now.

The 1st job I landed in Mauritius, they had just signed up for an SaaS with "workflow" in the name, which I had the *joy* of having to implement, but which turned out to be a major clusterfuck (not because it was a workflow, but because of its implementation).
We tried a few other community tools over a year, until we finally decided to go the BYO (Bring Your Own) way. And that's when I discovered state machines.

### How to know if you need a state machine

If you have an entity which has various "statuses" or "states" or whatever the name of a property whose value is changed by users and drives interactions with your entity, you probably have a workflow in place.
If you don't have a sort of state machine driving all that, you're probably in a world of pain.

### Benefits of a state machine

- Express clearly which states are accessible from a given state
- Programmatically enable access to a state (user role, system state,…)
- Attach properties to a state
- Trigger actions on state transitions

### Some projects to get you started

In PHP, for [Yii 1.x](http://www.yiiframework.com), there is [Raoul's SimpleWorkflow](https://github.com/raoul2000/simpleWorkflow), and [its awesome doc](http://s172418307.onlinehome.fr/project/sandbox/www/index.php?r=simpleWorkflow/page&view=doc).

Another for PHP, installable via composer, is [Yohang's Finite](https://github.com/yohang/Finite), which you can couple with [this trait](https://gist.github.com/tortuetorche/6365575). I've used it successfully with a Laravel 4.x backed API server, it's super effective.

For Javascript, how about the [state-machine](https://github.com/jakesgordon/javascript-state-machine).

### Some tips with workflows

The *divide and conquer* principle is a good one in general, very much in computer science and more so in the case of workflows.

One of the reasons why the workflow project using an SaaS which I mentioned earlier was unsuccessful was that it was one humongous monolithic piece. (The other reasons were overly complex configuration and *fugly* user interface.) There were exceptions all over the place and nobody ever knew for sure what the next step would entail (queue call to the hated dev guy).

So that workflow involved people from 3 departments, plus the client and made provision for 3rd parties/providers. All in one go.
By the time I started coding my own, I had come to breaking it down into smaller parts called "tasks", attached to a "job", which could in turn be attached to a "project", each of these being stateful.
Using a state machine, it's possible to trigger transitions in cascade and in a much more controlled manner, or inversely, prevent a transition due to the state of a related entity.

Here is one of the business rules I had to implement: until a purchase order has been received, final versions of files must not be available for download by clients. With state machine: 10 minutes coding, without: ???
Of course, in that last example, it doesn't stop someone from bypassing the system and sending the files by email without knowing that no purchase order was received.

This takes us to another principle I like, [KISS](http://en.wikipedia.org/wiki/KISS_principle "Keep it simple, stupid"). Breaking things into smaller pieces helps with that, and also separating concern, thus making people much more inclined to use your carefully crafted tool.

Go ahead, give it a spin, you'll thank yourself down the road.
