---
layout: post
title: Curtail updates
date: 2021-04-11 10:12:00 -07:00
categories: [ coding ]
---

_Note: see [Bug bounty and messy HTTP](https://gardna.blogliam.com/2021/04/05/bb-and-http/) for context_

This weekend, I've spent a decent amount of time converting [Curtail](https://github.com/wg4568/curtail) to a Burpsuite plugin. The hardest part has been the UI, but now I have that almost down it's just a matter of implementing features. I have basic scanning functionality down, here's a demo.

{% include tools/embed.html source="youtube" id="9xIxO88ikik" %}

On the todo list:

1. Fix HTTP parsing bugs (there are a few)
2. Improve scan detection, ignore things like timestamps
3. Implement the "output" tab
4. Add cURLification

Full writeup coming soon with the goal of better documenting the Burpsuite plugin development process. Will include lessons learned along with practical advice and documentation. 