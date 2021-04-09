---
layout: post
title: Auto deploying from a Github repository
date: 2021-04-06 22:16:00 -07:00
categories: [ coding ]
---

Today I upgraded blogliam.com by writing a tiny NPM project that listens to Github webhooks and automatically runs `git pull`. Basically it's just an HTTP server that you point your Github repo at.

![webhook config](/assets/img/2021-04-06-auto-deploy/webhooks.png)

_no ddos pls ;)_
{: style="margin-left: 5%" }

It's pretty configurable, and allows you to work on other branches without updating the production site. Here's a snippet from the config file:

```json
"repos": [
    {
        "hook": "/lam.blogliam.com",
        "name": "blogliam/lam.blogliam.com",
        "dir": "/var/www/lam.blogliam.com",
        "branch": "refs/heads/main",
        "script": "sh update.sh"
    },
]
```

The webserver can handle as many repos you want, each getting their own endpoint. To get it running yourself:

1. clone [the repository](https://github.com/wg4568/github-auto-deploy)
2. get dependancies with `npm install`
3. write up a config file
4. start the server with `npm start`

I manage my processes using [pm2](https://pm2.keymetrics.io/), which is helpful for ensuring it's always online.
