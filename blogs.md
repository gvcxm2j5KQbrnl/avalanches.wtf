---
layout: page
title: My Projects
permalink: /blog/
---
<head>
<meta property="og:type" content="website">
<meta property="og:url" content="https://avalanches.wtf/blog/">
<meta property="og:title" content="A Journey into Infrastructure & Security">
<meta property="og:description" content="Welcome to my personal blog. I build to break and break to build. documenting my projects across all areas of cybersecurity..">
<meta property="og:image" content="https://avalanches.wtf/assets/web-app-manifest-512x512.png">
<link rel="stylesheet" href="/assets/css/blog.css">
</head>

<div style="display: flex; justify-content: space-between; align-items: flex-start; margin-top: -80px;">
  
  <div style="margin-top: 85px;">
    <div style="line-height: 1.5; font-size: 1.1rem; text-align: left;">
      <p style="margin: 0;">Welcome to my personal Blog.</p>
      <p style="margin: 0;">Here you can find projects I am, or have been working on.</p>
      <p style="margin: 0;">I break to build, and build to break.</p>
    </div>
  </div>
  
  <a href="https://avalanches.wtf" rel="noopener noreferrer" style="text-decoration: none; border: none;">
    <img src="https://avalanches.wtf/assets/web-app-manifest-512x512.png" alt="funny anime girl" width="180" style="flex-shrink: 0; margin-top: 85px; border: none; outline: none;">
  </a>
</div>

<p style="margin: 15px 0;">&nbsp;</p>

<h3 style="font-size: 1.6rem; margin-bottom: 10px;">Recent Logs</h3> 
<ul style="margin-top: 0; font-size: 1.05rem; line-height: 1.7;">
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a> <span>({{ post.date | date: "%b %d, %Y" }})</span>
    </li>
  {% endfor %}
</ul>
