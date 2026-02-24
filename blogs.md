---
layout: page
title: My Projects
permalink: /blogs/
---
<link rel="stylesheet" href="/assets/css/blog.css">
Welcome to my personal Blog.
<br>Here you can find projects I am, or have been working on.
<br> I break to build, and build to break.

<p></p>
### Recent Logs 
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> <span>({{ post.date | date: "%b %d, %Y" }})</span>
    </li>
  {% endfor %}
</ul>
