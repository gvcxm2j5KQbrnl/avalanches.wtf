---
layout: page
title: My Projects
permalink: /blogs/
---
<link rel="stylesheet" href="/assets/css/blog.css">

<div style="display: flex; justify-content: space-between; align-items: flex-start; gap: 30px; text-align: left; margin-top: 10px;">
  <div style="margin: 0; line-height: 1.6; font-size: 1.2rem;">
    <p style="margin: 0;">Welcome to my personal Blog.</p>
    <p style="margin: 0;">Here you can find projects I am, or have been working on.</p>
    <p style="margin: 0;">I break to build, and build to break.</p>
  </div>
  <img src="https://avalanches.wtf/assets/web-app-manifest-512x512.png" alt="funny anime girl" width="240" style="flex-shrink: 0; margin-top: -10px;">
</div>

<p style="margin: 20px 0;">&nbsp;</p>

<h3 style="font-size: 1.8rem; margin-bottom: 15px;">Recent Logs</h3> 
<ul style="margin-top: 0; font-size: 1.1rem; line-height: 1.8;">
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a> <span>({{ post.date | date: "%b %d, %Y" }})</span>
    </li>
  {% endfor %}
</ul>
