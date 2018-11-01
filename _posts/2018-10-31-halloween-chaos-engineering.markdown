---
title: "Chaos Engineering @ Velocity"
layout: post
date: 2018-10-31 18:48
image: /assets/images/markdown.jpg
headerImage: false
tag:
- markdown
- components
- extra
category: blog
author: Lefteris Tatakis
description: Key points of Chaos Engineering talk @ Velocity
# jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="https://assets.github.com/images/icons/emoji/unicode/1f35c.png" height="20" width="20" align="absmiddle">'
---

Talk given by @ from Gremlin.

# What is Chaos Engineering?

Thoughtful, planned experiments designed to reveal the weakness in our systems.

Where we would try to:

> Inject something harmful to build an immunity

# Why do it?

- Downtime is really expensive
- Our dependencies will fail
- Pager fatigue for John and Marcus
- Reveal weak points in your systems

# Prerequisites

1. Dashboard and metrics (New Relic, Kibana) ✅
2. Alerting ✅
3. What is our cost per hour of outage? ❓

# Principles of Chaos Engineering ([http://principlesofchaos.org](http://principlesofchaos.org/) )

These experiments follow four steps:

1. Start by defining ‘steady state’ as some measurable output of a system that indicates normal behavior.
2. Hypothesize that this steady state will continue in both the control group and the experimental group.
3. Introduce variables that reflect real world events like servers that crash, hard drives that malfunction, network connections that are severed, etc.
4. Try to disprove the hypothesis by looking for a difference in steady state between the control group and the experimental group.

> Don’t approach it with a random strategy, instead approach it like a scientific experiment, thoughtful and planned.

# What experiments can you run?

• Reproduce outage conditions
• Unpredictable circumstances
• Large traffic spikes
• Race conditions
• Datacenter failure
• Time travel - system clocks to be out of sync
• Network errors
• CPU overloads

However, ensure we restrict the **blast radius** of our experiments.

And after each experiment, reiterate the experiment to see if we are resilient to it.

## Where we could we use it?

We already have it in the pipeline! 👷

**Hint: War games**

Or at least in my intepritation of it. 🤓

# Tools

Tools that could be used:

- Chaos Monkey; [https://github.com/Netflix/chaosmonkey](https://github.com/Netflix/chaosmonkey)
- Simian Army: [https://github.com/Netflix/SimianArmy](https://github.com/Netflix/SimianArmy)
- Litmus: [github.com](http://github.com/)/openebs/Litmus
- Powerful Seal: [https://github.com/bloomberg/powerfulseal](https://github.com/bloomberg/powerfulseal)

## Other

Post mortem URL: (different outages)
[github.com/danluu/post-mortems](http://github.com/danluu/post-mortems)