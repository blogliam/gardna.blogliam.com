---
layout: post
title: Burp Plugins! (curtail)
date: 2021-04-07 23:58:00 -07:00
categories: [ coding ]
---

Finally started work on my Burp plugin for my [HTTP minifier](/2021/04/05/bb-and-http/) project. I've gone with the name Curtail, because you _curtail_ the HTTP request. The hardest part so far has been UI design in Java. Burp uses all the JPanel stuff, not really sure what that API is called but it seems IntelliJ has a built in editor which I managed to get working.

![intellij editor](/assets/img/2021-04-07-curtail-minifier/intellij.png)

_intellij built in editor_
{: style="margin-left: 5%" }

I also drafted a quick UI mockup. I'm thinking that the user will eventually be able to right click on an HTTP request, and send to curtail.

![the mockup](/assets/img/2021-04-07-curtail-minifier/mockup.png)

_ms paint superiority_
{: style="margin-left: 5%" }

When it comes to the auto-minifier I've decided against the as-a-service approach, since Burpsuite already has built in tools for sending HTTP requests. Unfortunately I think the cURL conversion will have to be done remotely, since the h2c script is written in Perl.

For now I'm going to be grinding through UI development, since it's not something I'm very familiar with. Hopefully once that's out of the way porting the functionality from my first draft will be easy.

No Github repo yet--things are still in the very early stages--but expect one soon.
