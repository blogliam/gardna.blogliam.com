---
layout: post
title: Bug bounty and messy HTTP
time: 9:06 PM
---

Recently I've been running into the same problem. Security researchers use HTTP requests but engineers use cURL. Almost daily, I will open a bug bounty report to be greeted with a wall of text, only half of which is even necessary for the request to function. The researcher just copies and pastes the entire HTTP request from burp into a code block, and calls it a day. Of course along with the important info related to the exploit, they also bring with them a whole entourage of cookies, tracking data, and general junk that does nothing.

![if purgatory was an http request](https://www.dropbox.com/s/np2g5f3bu2i9nlh/Screenshot%202021-04-05%20212750.png?dl=1)

_Pictured, HTTP hell_
{: style="margin-left: 5%" }

The task:

1. Create an HTTP request parser
2. Extend that parser to programatically remove headers
3. Convert the resulting HTTP request into a cURL command

So far I have completed steps 1 and 2. Here's what that looks like:

![http-minifier](https://www.dropbox.com/s/563ira4n8njvqhu/image.png?dl=1)

_I wrote a little interface in Bootstrap_
{: style="margin-left: 5%" }

The messy request gets pasted on the left, and broken down by the parser. Filter rules can be set in the bottom right, and Content-Length is automatically calculated and inserted. Then the right hand textarea updates in real time with the appropriate headers removed.

I have a few ideas for the next steps. For HTTP to cURL I plan on using a little script called [h2c](https://curl.se/h2c/). I'll convert the TypeScript client app into an endpoint with h2c integration so that you can just hit it like any regular API. I also want to add auto-minify. This would iteratively remove headers, each time testing the request to make sure it hasn't broken. I also think that a BurpSuite plugin would be neat. I could either reimplement entirely in Python or just call the API I mentioned. Both have pros and cons worth considering.

That's all for today. You can find the source code [here](https://github.com/wg4568/http-minify)
