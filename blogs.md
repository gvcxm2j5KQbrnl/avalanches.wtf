---
layout: page
title: My Projects
permalink: /blogs/
---
<link rel="stylesheet" href="/assets/css/blog.css">

<div style="display: flex; justify-content: space-between; align-items: flex-start; gap: 20px; text-align: left;">
  <div style="margin: 0; line-height: 1.4;">
    <p style="margin: 0;">Welcome to my personal Blog.</p>
    <p style="margin: 0;">Here you can find projects I am, or have been working on.</p>
    <p style="margin: 0;">I break to build, and build to break.</p>
  </div>
  <img src="https://avalanches.wtf/assets/web-app-manifest-512x512.png" alt="funny anime girl" width="200" style="flex-shrink: 0; margin: 0;">
</div>

<p style="margin: 10px 0;">&nbsp;</p>

### Recent Logs 
<ul style="margin-top: 0;">
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a> <span>({{ post.date | date: "%b %d, %Y" }})</span>
    </li>
  {% endfor %}
</ul>
