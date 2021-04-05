---
layout: default
title: homepage
---

# Recent activity
___

{% for post in site.posts %}
<div class="pb-4">
    <h3><a href="{{ post.url }}">{{ post.title }}</a></h3>
    <h6 class="monospace">{{ post.date | date: "%D" }} {{ post.time }}</h6>
    <div>
        {{ post.content }}
    </div>
</div>
{% endfor %}