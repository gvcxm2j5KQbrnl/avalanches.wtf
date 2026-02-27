---
layout: post
title: "Building and Breaching: A Custom Windows Server Lab"
date: 2026-02-26
permalink: /blog/how-i-built-and-broke-windows-server/
categories: infrastructure security
---
<link rel="stylesheet" href="/assets/css/blog.css">

<div style="text-align: left;">
  <h1>Hello Everyone!</h1>
  <p style="font-size: 1.1rem; line-height: 1.6;">
    In this write-up, I will demonstrate how to deploy a Windows Server environment, configure Active Directory from the ground up, and perform a penetration test to exploit common misconfigurations.
  </p>
</div>

## The Mission
The objective of this project was to analyze the core of corporate infrastructure: Keyword **Active Directory**. By building the domain from scratch, I gained firsthand experience in the administrative configurations required for stability—and the security oversights that lead to total domain compromise.

<pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px;">
                          Graphical Diagram of Forest
                      
                     +----------------------------------+
                     | Domain: lab.local                |
                     | (Static IP Network: 10.0.0.0/24) |
                     +----------------+-----------------+
                                      |
                +---------------------+---------------------+
                |                     |                     |
      +---------+---------+ +---------+---------+ +---------+---------+
      |      [ DC01 ]     | |    [ WKSTN01 ]    | |    [ ATTACKER ]   |
      | Windows Server 22 | | Windows 11 Pro    | | Arch Linux        |
      | Role: Domain Ctrl | | Role: Target      | | Role: Host        |
      | IP: 10.0.0.10     | | IP: 10.0.0.20     | | IP: 10.0.0.50     |
      +-------------------+ +-------------------+ +-------------------+
</pre>

## Lab Architecture
To ensure a secure and isolated testing environment, I utilized a virtualized local lab:

<ul style="list-style-type: none; padding: 0;">
  <li style="margin-bottom: 15px;">
    <strong>Hypervisor:</strong> VMware Workstation Pro / VirtualBox
  </li>

  <li style="margin-bottom: 15px; display: flex; align-items: flex-start;">
    <details style="cursor: pointer; width: 100%;">
      <summary style="list-style: none; outline: none; display: flex; align-items: center; justify-content: space-between;">
        <span><strong>The Domain Controller:</strong> Windows Server 2022 Standard Evaluation</span>
        <span style="font-size: 0.8rem; color: #a78bfa; border: 1px solid #a78bfa; padding: 2px 8px; border-radius: 4px; margin-left: 10px;">Setup Details</span>
      </summary>
      <div style="margin-top: 15px; display: flex; gap: 10px; justify-content: flex-end;">
        <img src="https://github.com/user-attachments/assets/e2210227-c3ae-4a7f-a955-c567abbd9068" 
             alt="Windows Server 2022 Versions" 
             style="width: 45%; border-radius: 8px; border: 1px solid #333;" />
            <img width="512" height="305" alt="image" src="https://github.com/user-attachments/assets/8677dc6d-bc30-4e9f-9400-41447128fc5f" />
            alt="60GB Drive Partition" 
             style="width: 45%; border-radius: 8px; border: 1px solid #333;" />
      </div>
    </details>
  </li>

  <li style="margin-bottom: 15px;">
    <strong>The Victim:</strong> Windows 10 Pro
  </li>

  <li style="margin-bottom: 15px; display: flex; align-items: flex-start;">
    <details style="cursor: pointer; width: 100%;">
      <summary style="list-style: none; outline: none; display: flex; align-items: center; justify-content: space-between;">
        <span><strong>The Attacker:</strong> Arch Linux</span>
        <span style="font-size: 0.8rem; color: #a78bfa; border: 1px solid #a78bfa; padding: 2px 8px; border-radius: 4px; margin-left: 10px;">Play Demo</span>
      </summary>
      <div style="margin-top: 15px; background: #000; border-radius: 8px; border: 1px solid #333; overflow: hidden; text-align: center;">
        <img src="https://github.com/user-attachments/assets/2a71da84-c60c-4d12-bdb2-04723fdb8f07"
             alt="Asciicinema Gif of fastfetch"
             style="width: 80%; border-radius: 8px;" />
      </div>
    </details>
  </li>
</ul>

---

## Technical Hurdles
During the initial deployment of the Domain Controller, I encountered a "Compatibility Report" loop while booting from the ISO. This typically happens when selecting the "Upgrade" option on a fresh virtual disk. By selecting **Custom: Install Windows only (advanced)** and targeting the unallocated 60GB drive, I successfully initiated the base OS installation.
