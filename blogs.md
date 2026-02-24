---
layout: page
title: Other Blogs
permalink: /blogs/
---

Welcome to my personal Blog.
<br>I occasionally build stuff, and I like to break it too.
<br>Cybersecurity Enthusiast

### Recent Logs 
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> <span>({{ post.date | date: "%b %d, %Y" }})</span>
    </li>
  {% endfor %}
</ul>
