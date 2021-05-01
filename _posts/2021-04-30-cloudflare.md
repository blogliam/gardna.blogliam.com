---
layout: post
title: Setting up Cloudflare and Nginx
date: 2021-04-30 21:59:00 -07:00
categories: [ coding ]
---

Since I'm hosting from my home network I decided it would be a good idea to throw my entire server behind Cloudflare. This was relatively simple--I just had to update my DNS provider in NameCheap. Where things got interesting were with SSL.

By default, Cloudflare will encrypt all proxied traffic using their own SSL certificates however I already had SSL installed through Let's Encrypt. Simple enough, just revoke the Let's Encrypt certificates and remove them from my Nginx config. This worked but weirdly (and I still don't understand why) when inspecting the certificate it was still "Verified by Let's Encrypt." When I moved [gardna.net](https://gardna.net/), which didn't have Let's Encrypt set up already, the Cloudflare SSL worked fine. I imagine this is caused by some form of caching, but I have no idea where. I suspect this is one of those things that will just "go away" one day.

The final step in my Cloudflare transfer was to configure Nginx to only accept Cloudflare IPs. This prevents attackers from bypassing the Cloudflare proxy by simply attacking the host server. This was relatively easy since Cloudflare publishes it's IP ranges [here](https://www.cloudflare.com/ips/).

I put the following in the http section of my `/etc/nginx/nginx.conf` file.

```python
# allow cloudflare ips
allow 173.245.48.0/20;
allow 103.21.244.0/22;
allow 103.22.200.0/22;
allow 103.31.4.0/22;
allow 141.101.64.0/18;
allow 108.162.192.0/18;
allow 190.93.240.0/20;
allow 188.114.96.0/20;
allow 197.234.240.0/22;
allow 198.41.128.0/17;
allow 162.158.0.0/15;
allow 172.64.0.0/13;
allow 131.0.72.0/22;
allow 104.16.0.0/13;
allow 104.24.0.0/14;

deny all;
```

Now when a user hits my home IP address directly they get denied.

![denied](/assets/img/2021-04-30-cloudflare/denied.png)

Overall, I was really impressed with the free services Cloudflare provides. It makes me feel a little more confident in the security of my home server.