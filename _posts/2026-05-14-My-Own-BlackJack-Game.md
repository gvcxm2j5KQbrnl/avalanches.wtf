---
layout: post
title: "My Own BlackJack Game (in Python, of course)"
date: 2026-05-14
permalink: /blog/my-own-blackjack-game/
categories: programming
--- 
<head>
<meta property="og:type" content="website">
<meta property="og:url" content="https://avalanches.wtf/blog/my-own-blackjack-game/">
<meta property="og:title" content="My Own BlackJack Game (in Python, of course)">
<meta property="og:description" content="An attempt at making my own version of Blackjack featuring a couple tweaks and mods.">
<meta property="og:image" content="https://avalanches.wtf/assets/web-app-manifest-512x512.png">
<link rel="stylesheet" href="/assets/css/blog.css">

<link rel="stylesheet" href="https://cdn.plyr.io/3.7.8/plyr.css" />
<style>
  .music-widget {
    position: fixed;
    bottom: 24px;
    right: 24px;
    background: rgba(15, 23, 42, 0.8); 
    backdrop-filter: blur(20px);
    -webkit-backdrop-filter: blur(20px);
    border: 1px solid rgba(255, 255, 255, 0.08);
    padding: 12px 20px 12px 12px;
    border-radius: 40px;
    display: flex;
    align-items: center;
    gap: 16px;
    box-shadow: 0 16px 40px rgba(0, 0, 0, 0.5);
    z-index: 9999;
    transition: transform 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  }

  .music-widget:hover {
    transform: translateY(-4px);
  }

  .album-art {
    width: 48px;
    height: 48px;
    border-radius: 50%;
    background: url('https://i.scdn.co/image/ab67616d0000b2736f3adfc82e576b7d55c34ba7') center/cover;
    box-shadow: 0 0 0 4px rgba(255, 255, 255, 0.05);
    animation: spin 16s linear infinite;
    animation-play-state: paused;
  }

  .playing .album-art {
    animation-play-state: running;
  }

  .song-details {
    display: flex;
    flex-direction: column;
    padding-right: 4px;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  }

  .song-title {
    font-size: 14px;
    font-weight: 600;
    color: #f1f5f9;
    margin: 0;
    letter-spacing: -0.01em;
    line-height: 1.2;
  }

  .artist-name {
    font-size: 12px;
    color: #94a3b8;
    margin: 2px 0 0 0;
    text-transform: lowercase;
    line-height: 1.2;
  }

  .play-btn {
    background: #ffffff;
    border: none;
    width: 36px;
    height: 36px;
    border-radius: 50%;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: background 0.2s, transform 0.1s;
    padding: 0;
  }

  .play-btn:hover {
    background: #cbd5e1;
    transform: scale(1.05);
  }

  .play-btn:active {
    transform: scale(0.95);
  }

  .play-btn svg {
    width: 14px;
    height: 14px;
    fill: #0f172a;
  }

  .plyr--audio {
    display: none !important;
  }

  @keyframes spin {
    100% { transform: rotate(360deg); }
  }
</style>
</head>

<div title="Hello Friends 🤑" style="cursor: help; display: inline-block;">
  <h1><strong><em>Привет Друзья!</em></strong>
  <img src="https://raw.githubusercontent.com/gvcxm2j5KQbrnl/avalanches.wtf/refs/heads/main/assets/513.gif" 
       alt="money face gif" 
       style="height: 35px;"> </h1>
</div>

<p style="font-size: 1.1rem; line-height: 1.6;">
  its been three weeks, imma do this when i feel like it 
</p>


<audio id="player" controls>
  <source src="/assets/permafrost.mp3" type="audio/mp3" />
</audio>

<div class="music-widget" id="custom-widget">
  <div class="album-art"></div>
  
  <div class="song-details">
    <p class="song-title">permafrost</p>
    <p class="artist-name">kumosai</p>
  </div>

  <button class="play-btn" id="custom-play-trigger" aria-label="Play/Pause">
    <svg id="play-icon" viewBox="0 0 24 24"><path d="M8 5v14l11-7z"/></svg>
  </button>
</div>


<script src="https://cdn.plyr.io/3.7.8/plyr.js"></script>
<script>
  const player = new Plyr('#player');
  
  const widget = document.getElementById('custom-widget');
  const playBtn = document.getElementById('custom-play-trigger');
  const playIcon = document.getElementById('play-icon');

  const playPath = "M8 5v14l11-7z";
  const pausePath = "M6 19h4V5H6v14zm8-14v14h4V5h-4z";

  playBtn.addEventListener('click', () => {
    player.togglePlay();
  });

  player.on('play', () => {
    widget.classList.add('playing');
    playIcon.querySelector('path').setAttribute('d', pausePath);
  });

  player.on('pause', () => {
    widget.classList.remove('playing');
    playIcon.querySelector('path').setAttribute('d', playPath);
  });
</script>
