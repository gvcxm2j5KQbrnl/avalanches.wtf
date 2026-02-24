---
layout: page
title: My Projects
permalink: /blogs/
---
<link rel="stylesheet" href="/assets/css/blog.css">

<div style="display: flex; justify-content: space-between; align-items: center; gap: 40px; text-align: left;">
  <div>
    Welcome to my personal Blog.<br>
    Here you can find projects I am, or have been working on.<br>
    I break to build, and build to break.
  </div>
  <img src="https://github.com/user-attachments/assets/1536c07f-3801-453f-83d9-3da5d0c5b997" alt="annoyed anime girl" width="235" style="flex-shrink: 0;">
</div>

<p>&nbsp;</p>

### Recent Logs 
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a> <span>({{ post.date | date: "%b %d, %Y" }})</span>
    </li>
  {% endfor %}
</ul>
