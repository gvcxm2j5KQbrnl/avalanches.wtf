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
    background: rgba(15, 23, 42, 0.85); 
    backdrop-filter: blur(20px);
    -webkit-backdrop-filter: blur(20px);
    border: 1px solid rgba(255, 255, 255, 0.08);
    padding: 12px 16px 12px 12px;
    border-radius: 40px;
    display: flex;
    align-items: center;
    gap: 12px;
    box-shadow: 0 16px 40px rgba(0, 0, 0, 0.5);
    z-index: 9999;
    transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
    max-width: 320px;
  }

  .music-widget:hover {
    transform: translateY(-4px);
  }

  .album-art {
    width: 44px;
    height: 44px;
    border-radius: 50%;
    background: url('https://i.scdn.co/image/ab67616d00001e02d0b8bc13b08ab92d691d64d4') center/cover;
    box-shadow: 0 0 0 4px rgba(255, 255, 255, 0.05);
    animation: spin 16s linear infinite;
    animation-play-state: paused;
    flex-shrink: 0;
  }

  .playing .album-art {
    animation-play-state: running;
  }

  .song-details {
    display: flex;
    flex-direction: column;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    width: 100px;
  }

  .song-title {
    font-size: 13px;
    font-weight: 600;
    color: #f1f5f9;
    margin: 0;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .artist-name {
    font-size: 11px;
    color: #94a3b8;
    margin: 1px 0 0 0;
    text-transform: lowercase;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .volume-container {
    display: flex;
    align-items: center;
    width: 0px;
    overflow: hidden;
    transition: width 0.3s ease, margin 0.3s ease;
  }

  .music-widget:hover .volume-container,
  .volume-container:focus-within {
    width: 70px;
    margin-left: 4px;
  }

  .volume-slider {
    -webkit-appearance: none;
    width: 100%;
    height: 4px;
    border-radius: 2px;
    background: rgba(255, 255, 255, 0.2);
    outline: none;
    cursor: pointer;
  }

  .volume-slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background: #ffffff;
    transition: transform 0.1s;
  }

  .volume-slider::-webkit-slider-thumb:hover {
    transform: scale(1.3);
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
    flex-shrink: 0;
  }

  .play-btn:hover {
    background: #cbd5e1;
    transform: scale(1.05);
  }

  .play-btn svg {
    width: 14px;
    height: 14px;
    fill: #0f172a;
  }

  .youtube-hide-wrapper {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    border: 0;
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

<div class="youtube-hide-wrapper">
  <div id="player" data-plyr-provider="youtube" data-plyr-embed-id="sd9kHahPjp4"></div>
</div>

<div class="music-widget" id="custom-widget">
  <div class="album-art"></div>
  
  <div class="song-details">
    <p class="song-title">danmaku</p>
    <p class="artist-name">kumosai</p>
  </div>

  <div class="volume-container">
    <input type="range" class="volume-slider" id="custom-volume" min="0" max="1" step="0.05" value="0.4">
  </div>

  <button class="play-btn" id="custom-play-trigger" aria-label="Play/Pause">
    <svg id="play-icon" viewBox="0 0 24 24"><path d="M8 5v14l11-7z"/></svg>
  </button>
</div>

<script src="https://cdn.plyr.io/3.7.8/plyr.js"></script>
<script>
  const player = new Plyr('#player', { volume: 0.4 });
  
  const widget = document.getElementById('custom-widget');
  const playBtn = document.getElementById('custom-play-trigger');
  const playIcon = document.getElementById('play-icon');
  const volumeSlider = document.getElementById('custom-volume');

  const playPath = "M8 5v14l11-7z";
  const pausePath = "M6 19h4V5H6v14zm8-14v14h4V5h-4z";

  playBtn.addEventListener('click', () => {
    player.togglePlay();
  });

  volumeSlider.addEventListener('input', (e) => {
    player.volume = e.target.value;
  });

  player.on('volumechange', () => {
    volumeSlider.value = player.volume;
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
