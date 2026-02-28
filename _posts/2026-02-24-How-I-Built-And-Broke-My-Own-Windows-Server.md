---
layout: post
title: "Building and Breaching: A Custom Windows Server Lab"
date: 2026-02-26
permalink: /blog/how-i-built-and-broke-windows-server/
categories: infrastructure security
---
<link rel="stylesheet" href="/assets/css/blog.css">

<div style="text-align: left;">
  <h1><strong><em>Hello Everyone!</em></strong></h1>
  <p style="font-size: 1.1rem; line-height: 1.6;">
    In this write-up, I will demonstrate how to deploy a Windows Server environment, configure Active Directory from the ground up, and perform a penetration test to exploit common misconfigurations.
  </p>
</div>
&nbsp;

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
&nbsp;
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
        <span style="font-size: 0.8rem; color: #a78bfa; border: 1px solid #a78bfa; padding: 2px 8px; border-radius: 4px; margin-left: 10px;">Details</span>
      </summary>
      <div style="margin-top: 15px; display: flex; gap: 10px; justify-content: flex-end;">
        <img src="https://github.com/user-attachments/assets/e2210227-c3ae-4a7f-a955-c567abbd9068" 
             alt="Windows Server 2022 Versions" 
             style="width: 45%; border-radius: 8px; border: 1px solid #333;" />
             <img src="https://github.com/user-attachments/assets/720145a4-85fe-476e-b530-cccdbebf91b5" 
             alt="60GB Partition" 
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
        <span style="font-size: 0.8rem; color: #a78bfa; border: 1px solid #a78bfa; padding: 2px 8px; border-radius: 4px; margin-left: 10px;">Details</span>
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
&nbsp;
# Configuration: Phase 1 Network & Identity
A Domain Controller requires a predictable identity. Using the **SConfig** utility, I performed two critical steps:

1. **Static IP Assignment:** Set the interface to <em><strong>10.0.0.10/24</strong></em>. This ensures that all future lab machines (Workstations, Attacker) have a reliable DNS target.
2. **Hostname Definition:** Changed the default random string to <em><strong>DC01</strong></em>.
<img width="495" height="208" alt="Sconfig screenshot 1" src="https://github.com/user-attachments/assets/a46a1b92-5a62-44db-93f0-b583869bf1ea" />
<img width="495" height="210" alt="Sconfig screenshot 2" src="https://github.com/user-attachments/assets/0cf1a9d1-adaf-4736-b2a3-ee3c73b0aa39" />



