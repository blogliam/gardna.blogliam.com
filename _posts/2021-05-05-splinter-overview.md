---
layout: post
title: Splinter&colon; Minecraft Performance Enhancements
date: 2021-05-05 15:18:00 -07:00
categories: [ coding, writeup ]
---

Finally getting around to the [splinter](https://github.com/wg4568/splinter/) writeup. It's been a few weeks since I touched this project but I suspect I'll be getting things rolling again shortly so I figured it's about time to get this done. This is going to be a long one.

#### The problem

At the core of Minecraft's optimization issues lies a singular problem: a lack of multithreading. Its single-thread design is so intrinsic to the code that I doubt it's a feature we'll be seeing any time soon--yet it would solve almost all of the major issues with Minecraft server performance. For large server networks, this flaw is overcome by separating players into hundreds of smaller servers, each handling around 50 players. This works fine for minigames, but Minecraft is a survival game. Pure SMP (survival multiplayer) requires that every player is in the same world at the same time. One world, one server, one thread.

This limitation is the greatest obstacle to SMP hosting. Take 2b2t.org for example, an SMP server that has a maximum capacity of around 250 players. It's plagued by performance issues, despite running on relatively powerful hardware. Not only is it consistently laggy, but it often has well over 500 players in queue _waiting_ to join the server. They have the player base to have thousands of concurrent players but are limited by the poor design of Minecraft.

![2b2t](/assets/img/2021-05-05-splinter-overview/2b2t.png)

_The 2b2t spawnpoint, shown from above_

If the single-threaded limitation were to be overcome it would change the face of Minecraft hosting. Imagine thousands of players all in the same survival world without the incredible lag that would come with conventional server design.

#### The solution

Many games built with massive online multiplayer in mind use a solution called sharding. In short, these games will split the world into multiple servers. For example in an MMORPG set in space, each planet could be handled by a different server. When you jump between planets, you are automatically switched from one server to another. Of course, Minecraft has no such mechanism and we are tasked with implementing this ourselves.

Splinter's approach to this is relatively straightforward. We split up the world into regions, our initial prototypes do so as shown in the diagram below. Each region is then handled by its own Minecraft server instance. When a player crosses the borders between the regions they are teleported into the new server using Bungeecord.

![regions](/assets/img/2021-05-05-splinter-overview/world.png)

_I figured that a spawn region would make sense, and then one for each quadrant_

The nice thing about this solution is that it flattens the cost-per-player curve that comes with single-core hosting. As it stands right now, the difference between a $10 server and a $100 server is not a 10x increase in performance. There comes a point where no matter how much money you pour into hosting returns will diminish so much that the increased player count isn't worth the money. This is caused by the single-core limitation. Server performance is tied almost exclusively to CPU clock speed and thread performance. This means that a server can't utilize high-end [multi-core CPUs](https://www.amd.com/en/products/cpu/amd-ryzen-threadripper-3970x) which ironically often have worse single-thread performance than their cheaper counterparts. With Splinter, this becomes significantly more linear.

#### What's been built

So far, I have solo developed a working prototype that handles basic functionality. Below is a brief YouTube video showing a player's perspective on a server using Splinter. I think it's important to note that a player typically wouldn't build so close to a border. The point of this video is to show how seamless the transition is.

{% include tools/embed.html source="youtube" id="qQSTF26Abw8" %}

_There's a weird rendering error around 10 seconds, the stutter isn't caused by Splinter_

As you can see, a lot of features to be implemented. Chat isn't synchronized--nor is the player list. Join and leave messages need to be disabled and certain edge cases such as redstone across the border must be handled. Most importantly, entities and players need to be visible even if they exist on another server. There's a long way to go, but this demo is critical in proving the concept's viability.

#### Splinter community

Now for the big news. I've spent the past few days building a community surrounding the Splinter project in official partnership with [CliqueCraft LLC](https://classic.cliquemc.net/). My role in the company essentially boils down to "open-source community manager." We've created a branch of CliqueCraft titled OpenClique, in which we plan on developing a variety of open-source Minecraft projects with community involvement as a primary focus.

As part of this effort, the CliqueCraft owner who goes by [Brandominos](https://brandominos.com) created a TikTok demonstrating the Splinter project which nicely sums up progress so far. You can find it linked below.

- TikTok Trailer: [https://www.tiktok.com/@brandominos/video/6959254693121887493](https://www.tiktok.com/@brandominos/video/6959254693121887493)
- OpenClique Discord: [https://discord.gg/F93NMyBHda](https://discord.gg/F93NMyBHda)
- GitHub Org: [https://github.com/OpenCliqueCraft](https://github.com/OpenCliqueCraft)

To get involved, join our [Discord](https://discord.gg/F93NMyBHda) server for more information.