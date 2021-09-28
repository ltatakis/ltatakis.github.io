---
title: "Liberty Cars - Iteration 2"
layout: post
date: 2020-10-1 22:10
tag: 
- Jekyll
- netlify
image: # TODO
headerImage: true
projects: true
hidden: true # don't count this post in blog pagination
description: "Keeping it simple"
category: project
author: LefterisTatakis
externalLink: false
---


As of early 2020, the AWS lambdas I was using for version one where being decommissioned due to Node 8 EOL.  Plus several other technologies I was used were way out of date. So in April 2020 I decided to simplify the website.

The site does not actually do any business, its primarily my testing ground website / domain. 
For this reason I decided to a make it a static website and deploy it with [Netlify](https://netlify.com).
I loved it, super simple and the generation of the website with Jekyll was a few hours work. The website lost a lot of its functionality that had all the advanced technical features and the technical debt, but I decided it was not needed. It only added complexity at this point and stress. 
Instead I experimented with static website form submission systems that send emails if actually used. Just in case some person is actually interested. In the months is been live only a few bots have actually got through the form. ✌️


For example some of the functionality it lost:

The old homepage:
![Markdown Image][1]

The old date range selection:
![Markdown Image][2]

The old booking page:
![Markdown Image][3]

As always I will admit my frontend skills are not the best. I am a Full Stack Engineer that focuses on DevOps and Backend work, that can change a CSS colour if needed 🤓

Check out the new [liberty cars here](https://www.libertycars.gr).

[1]: /assets/images/libertycars/home.png
[2]: /assets/images/libertycars/select.png
[3]: /assets/images/libertycars/book.png
