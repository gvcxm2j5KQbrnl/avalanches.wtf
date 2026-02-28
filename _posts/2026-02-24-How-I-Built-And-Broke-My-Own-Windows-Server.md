---
layout: post
title: "Building and Testing a Custom Windows Server Lab"
date: 2026-02-26
permalink: /blog/how-i-built-and-broke-windows-server/
categories: infrastructure security
---
<link rel="stylesheet" href="/assets/css/blog.css">

<div style="text-align: left;">
  <h1>
    <strong><em>Hey everyone!</em></strong>
    <img src="https://github.com/user-attachments/assets/b4f09079-a433-4035-ac5f-6369de58e62c" 
         style="height: 30px; margin-left: 5px;" 
         alt="emoji">
  </h1> 

  <p style="font-size: 1.1rem; line-height: 1.6;">
    This project covers the entire process: from the initial server deployment and Active Directory configuration to analyzing how specific admin decisions affect the domain's security. Once the foundation is solid, I’ll move into the offensive phase—attempting to break the system and performing a penetration test to see how well it actually holds up.
  </p>
</div>
&nbsp;

## The Objective
The main goal was to gain hands-on experience with **Active Directory**, the backbone of most corporate networks. By building the environment myself, I was able to see how various components integrate and identify common misconfigurations that often lead to security vulnerabilities.

<pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px;">
                               Lab Network Forest     
                       
                      +----------------------------------+
                      | Domain: lab.local                |
                      | (Subnet: 10.0.0.0/24)            |
                      +----------------+-----------------+
                                       |
                 +---------------------+---------------------+
                 |                     |                     |
       +---------+---------+ +---------+---------+ +---------+---------+
       |      [ DC01 ]     | |    [ WKSTN01 ]    | |    [ ATTACKER ]   |
       | Windows Server 22 | | Windows 11 Pro    | | Arch Linux        |
       | Domain Controller | | Target Workstation| | My Host Machine   |
       | IP: 10.0.0.10     | | IP: 10.0.0.20     | | IP: 10.0.0.50     |
       +-------------------+ +-------------------+ +-------------------+
</pre>
&nbsp;
## Lab Architecture
I used a virtualized environment to ensure the lab remains isolated and secure:

<ul style="list-style-type: none; padding: 0;">
  <li style="margin-bottom: 15px;">
    <strong>Hypervisor:</strong> VMware Workstation Pro / VirtualBox
  </li>

  <li style="margin-bottom: 15px; display: flex; align-items: flex-start;">
    <details style="cursor: pointer; width: 100%;">
      <summary style="list-style: none; outline: none; display: flex; align-items: center; justify-content: space-between;">
        <span><strong>The Domain Controller:</strong> Windows Server 2022</span>
        <span style="font-size: 0.8rem; color: #a78bfa; border: 1px solid #a78bfa; padding: 2px 8px; border-radius: 4px; margin-left: 10px;">Details</span>
      </summary>
      <div style="margin-top: 15px; display: flex; gap: 10px; justify-content: flex-start;">
        <a href="https://github.com/user-attachments/assets/e2210227-c3ae-4a7f-a955-c567abbd9068" target="_blank">
          <img src="https://github.com/user-attachments/assets/e2210227-c3ae-4a7f-a955-c567abbd9068" 
               alt="Windows Server 2022 Version Selection" 
               style="width: 250px; border-radius: 8px; border: 1px solid #333;" />
        </a>
        <a href="https://github.com/user-attachments/assets/720145a4-85fe-476e-b530-cccdbebf91b5" target="_blank">
          <img src="https://github.com/user-attachments/assets/720145a4-85fe-476e-b530-cccdbebf91b5" 
               alt="Storage Partitioning" 
               style="width: 250px; border-radius: 8px; border: 1px solid #333;" />
        </a>
      </div>
    </details>
  </li>

  <li style="margin-bottom: 15px;">
    <strong>The Target:</strong> Windows 10 Pro
  </li>

  <li style="margin-bottom: 15px; display: flex; align-items: flex-start;">
    <details style="cursor: pointer; width: 100%;">
      <summary style="list-style: none; outline: none; display: flex; align-items: center; justify-content: space-between;">
        <span><strong>The Attacker:</strong> Arch Linux (Host Machine)</span>
        <span style="font-size: 0.8rem; color: #a78bfa; border: 1px solid #a78bfa; padding: 2px 8px; border-radius: 4px; margin-left: 10px;">Details</span>
      </summary>
      <div style="margin-top: 15px; background: #000; border-radius: 8px; border: 1px solid #333; overflow: hidden; text-align: center;">
        <a href="https://github.com/user-attachments/assets/2a71da84-c60c-4d12-bdb2-04723fdb8f07" target="_blank">
          <img src="https://github.com/user-attachments/assets/2a71da84-c60c-4d12-bdb2-04723fdb8f07"
               alt="Arch Linux Host Configuration" />
        </a>
      </div>
    </details>
  </li>
</ul>

---
&nbsp;
# Phase 1: Identity & Network Configuration
To establish a stable environment, I needed to define a permanent identity for the server. In a domain environment, the Domain Controller must have a fixed IP to ensure reliable DNS resolution for all clients. I used **SConfig** for the initial configuration:

1. **Static IP Assignment:** I assigned <em><strong>10.0.0.10/24</strong></em>. This prevents connectivity issues that would occur if the IP were to change via DHCP.
2. **Hostname Definition:** I renamed the server to <em><strong>DC01</strong></em>. This makes the infrastructure much easier to manage and ensures consistency in the logs.

<div style="display: flex; gap: 10px; margin-top: 15px;">
  <a href="https://github.com/user-attachments/assets/a46a1b92-5a62-44db-93f0-b583869bf1ea" target="_blank">
    <img width="495" alt="Sconfig Computer Name" src="https://github.com/user-attachments/assets/a46a1b92-5a62-44db-93f0-b583869bf1ea" style="border-radius: 8px; border: 1px solid #333;" />
  </a>
  <a href="https://github.com/user-attachments/assets/0cf1a9d1-adaf-4736-b2a3-ee3c73b0aa39" target="_blank">
    <img width="495" alt="Sconfig Network Settings" src="https://github.com/user-attachments/assets/0cf1a9d1-adaf-4736-b2a3-ee3c73b0aa39" style="border-radius: 8px; border: 1px solid #333;" />
  </a>
</div>
&nbsp;

# Phase 2: Active Directory Deployment
