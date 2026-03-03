---
layout: post
title: "How I Built (And Broke) My Own Windows Server"
date: 2026-02-26
permalink: /blog/how-i-built-and-broke-windows-server/
categories: infrastructure security
---
<head>
<meta property="og:type" content="website">
<meta property="og:url" content="https://avalanches.wtf/blog/how-i-built-and-broke-windows-server/">
<meta property="og:title" content="How I Built (And Broke) My Own Windows Server">
<meta property="og:description" content="An inside look at how I deployed a Windows Server environment from scratch just to see if I could compromise it.">
<meta property="og:image" content="https://avalanches.wtf/assets/web-app-manifest-512x512.png">
<link rel="stylesheet" href="/assets/css/blog.css">
</head>

<div style="text-align: left;">
  <h1>
    <strong><em>Hey everyone!</em></strong>
    <img src="https://github.com/user-attachments/assets/b4f09079-a433-4035-ac5f-6369de58e62c" 
         style="height: 30px; position: relative; bottom: 2px; margin-left: 2px;" 
         alt="emoji">
  </h1> 

  <p style="font-size: 1.1rem; line-height: 1.6;">
    I decided to build a Windows Server lab from scratch to see how Active Directory actually works under the hood. This post covers the basic setup and getting the domain running. Once the foundation is solid, I’m going to try and find some weak spots and see if I can break into it.
  </p>
</div>

&nbsp;

## The Goal
Active Directory is pretty much everywhere in the corporate world. By building the whole thing myself, I can see how everything connects and, more importantly, what kind of small mistakes lead to a network getting compromised.

<pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px;">
                                The Lab Layout     
                        
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
       | Domain Controller | | Target PC         | | My Main Machine   |
       | IP: 10.0.0.10     | | IP: 10.0.0.20     | | IP: 10.0.0.50     |
       +-------------------+ +-------------------+ +-------------------+
</pre>
&nbsp;

## Lab Architecture
I’m running everything in a virtual environment to keep it safe and separate from my main files. I'm using a **Host-Only** network, which acts as a sandbox so I don't accidentally mess with my actual home network.

<ul style="list-style-type: none; padding: 0;">
  <li style="margin-bottom: 15px; display: flex; align-items: flex-start;">
    <details style="cursor: pointer; width: 100%;">
      <summary style="list-style: none; outline: none; display: flex; align-items: center; justify-content: space-between;">
        <span><strong>The Domain Controller:</strong> Windows Server 2022</span>
        <span style="font-size: 0.8rem; color: #a78bfa; border: 1px solid #333; padding: 2px 8px; border-radius: 4px; margin-left: 10px;">Details</span>
      </summary>
      <div style="margin-top: 15px; display: flex; gap: 10px; justify-content: flex-start;">
        <a href="https://github.com/user-attachments/assets/439a70eb-71a8-4729-b230-3b21db09d276" target="_blank">
          <img src="https://github.com/user-attachments/assets/439a70eb-71a8-4729-b230-3b21db09d276" 
               alt="Windows Server 2022 Setup" 
               style="width: 400px; border-radius: 8px; border: 1px solid #333;" />
        </a>
      </div>
    </details>
  </li>
  
  <li style="margin-bottom: 15px; display: flex; align-items: flex-start;">
    <details style="cursor: pointer; width: 100%;">
      <summary style="list-style: none; outline: none; display: flex; align-items: center; justify-content: space-between;">
        <span><strong>The Victim:</strong> Windows 11 Pro</span>
        <span style="font-size: 0.8rem; color: #a78bfa; border: 1px solid #333; padding: 2px 8px; border-radius: 4px; margin-left: 10px;">Details</span>
      </summary>
      <div style="margin-top: 15px; display: flex; gap: 10px; justify-content: flex-start;">
        <a href="https://github.com/user-attachments/assets/c6893e2e-3da9-4d32-b305-31fad852f967" target="_blank">
          <img src="https://github.com/user-attachments/assets/c6893e2e-3da9-4d32-b305-31fad852f967" 
               alt="Windows 11 Pro Setup" 
               style="width: 400px; border-radius: 8px; border: 1px solid #333;" />
        </a>
      </div>
    </details>
  </li>

  <li style="margin-bottom: 15px; display: flex; align-items: flex-start;">
    <details style="cursor: pointer; width: 100%;">
      <summary style="list-style: none; outline: none; display: flex; align-items: center; justify-content: space-between;">
        <span><strong>The Attacker:</strong> Arch Linux </span>
        <span style="font-size: 0.8rem; color: #a78bfa; border: 1px solid #333; padding: 2px 8px; border-radius: 4px; margin-left: 10px;">Details</span>
      </summary>
      <div style="margin-top: 15px; background: transparent; text-align: center;">
        <a href="https://github.com/user-attachments/assets/2a71da84-c60c-4d12-bdb2-04723fdb8f07" target="_blank">
          <img src="https://github.com/user-attachments/assets/2a71da84-c60c-4d12-bdb2-04723fdb8f07" 
               alt="Arch Linux Info" 
               style=" width: 60%; border-radius: 4px; border: 1px solid #333;" />
        </a>
      </div>
    </details>
  </li>
  <li style="margin-bottom: 15px;"><strong>Hypervisor:</strong> VMware Workstation</li>
</ul>

---
&nbsp;

# Phase 1: Basic Config
The first thing I had to do was give the server a permanent IP and a clear name. In AD, if your IP shifts around, everything breaks. I used **SConfig** to handle the basics:

<ol>
  <li><strong>Static IP:</strong> I assigned <em><strong>10.0.0.10/24</strong></em> so the server is always found at the same address.</li>
  <li><strong>Hostname:</strong> I renamed it to <em><strong>DC01</strong></em> to keep things clean.</li>
</ol>

<div style="display: flex; gap: 10px; margin-top: 15px;">
  <img width="495" alt="Sconfig Name" src="https://github.com/user-attachments/assets/a46a1b92-5a62-44db-93f0-b583869bf1ea" style="border-radius: 8px; border: 1px solid #333;" />
  <img width="495" alt="Sconfig IP" src="https://github.com/user-attachments/assets/0cf1a9d1-adaf-4736-b2a3-ee3c73b0aa39" style="border-radius: 8px; border: 1px solid #333;" />
</div>
&nbsp;

# Phase 2: Active Directory Setup
Now for the actual core: turning the server into a Domain Controller. I used PowerShell for this because it's way faster than clicking through the GUI wizards. I promoted the server to a new Forest for <em>lab.local</em>.

<pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px; overflow-x: auto;">
PS C:\> Install-ADDSForest -DomainName "lab.local" -InstallDns:$true -Force:$true
</pre>

<div style="margin-top: 20px; text-align: center;">
  <img width="89%" alt="AD Setup Progress" src="https://github.com/user-attachments/assets/2ca0371e-b478-4b37-8af6-bd6c3d5d9663" style="border-radius: 8px; border: 1px solid #333;" />
  <p style="margin-top: 10px; font-style: italic; color: #666;">
    After a reboot, the server officially became the master of lab.local.
  </p>
</div>

&nbsp;

# Phase 3: Populating the Domain
A DC without users is a ghost town. I created an **Organizational Unit (OU)** called <em>Lab_Users</em> and populated it with different roles: an IT admin, a manager, and a SQL service account.

<pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px; overflow-x: auto;">
# Creating the accounts and a Domain Admin for future testing
PS C:\> New-ADUser -Name "IT Admin" -SamAccountName "it_admin" ...
PS C:\> Add-ADGroupMember -Identity "Domain Admins" -Members "it_admin"
</pre>

<div style="margin-top: 20px; text-align: center;">
  <img width="89%" alt="AD Users and Roles" src="https://github.com/user-attachments/assets/7cdd01e9-4afb-4db8-b73f-0c4eb3db1447" style="border-radius: 8px; border: 1px solid #333;" />
</div>

&nbsp;

# Phase 4: Setting up the Workstation 
<div style="width: 100%; border: none; padding: 0;">
  <div style="font-size: 1.1rem; line-height: 1.6; margin-bottom: 20px;">
    <strong>The Victim:</strong> Windows 11 Pro (WKSTN01)
    <p>Now I needed a machine to join to the domain. If DNS isn't pointed exactly at the DC (10.0.0.10), the join fails immediately. After some troubleshooting with the firewall and network adapters, it finally worked.</p>
  </div>
  
  <div style="text-align: center;">
    <img src="https://github.com/user-attachments/assets/55e61ff0-4fdf-4ca1-b1b7-882b268ac611" alt="IPv4 Configuration" style="width: 650px; border-radius: 8px; border: 1px solid #333;" />
  </div>
</div>

&nbsp;

### Joining & Success
Joining the domain feels like the final "Build" boss. Once I authenticated with the Admin credentials, I got the welcome message. The lab was officially alive.

<div style="text-align: center; margin-top: 15px;">
  <img src="https://github.com/user-attachments/assets/231eb4cd-987e-46a1-91e2-51c338dafe9a" alt="Welcome to Domain" style="width: 650px; border-radius: 8px; border: 1px solid #333;" />
</div>

&nbsp;

# Phase 5: Vulnerability Engineering
With the foundation solid, the "Building" part is over. Now I have to intentionally weaken the security to simulate real-world mistakes. My first target: **Kerberoasting**. I assigned a **Service Principal Name (SPN)** to the SQL account.

<pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px; overflow-x: auto;">
PS C:\> setspn -a MSSQLSvc/sql01.lab.local:1433 sql_svc
</pre>

<div style="text-align: center; margin-top: 15px;">
  <img src="https://github.com/user-attachments/assets/27f104f3-7fd7-4901-9951-25bdc46660ae" alt="Set SPN Command" style="width: 650px; border-radius: 8px; border: 1px solid #333;" />
  <p style="margin-top: 10px; font-style: italic; color: #666;">
    Registered the SPN. The lab is now officially "vulnerable".
  </p>
</div>

&nbsp;

# Phase 6: Reconnaissance & Entry
Time to step into the boots of an attacker. Since my main OS is **Arch Linux**, I’m using it as the attack platform. I had to manually assign an IP to my host's virtual interface to bridge into the lab's subnet.

<pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px; overflow-x: auto;">
$ sudo ip addr add 10.0.0.50/24 dev vmnet1
</pre>

### Mapping the Target
I ran a full **Nmap** scan against the DC to see what services were exposed. 

<div style="text-align: center; margin-top: 15px;">
  <img src="https://github.com/user-attachments/assets/adb6991a-5ba1-43bb-b30c-d656d589f444" alt="Nmap Scan Results" style="width: 800px; border-radius: 8px; border: 1px solid #333;" />
</div>

&nbsp;

### The Loot: Important Ports
The scan results confirmed that the DC is broadcasting its roles. These are the main entry points for AD attacks:

<table style="width: 100%; border-collapse: collapse; margin-top: 20px; border: 1px solid #333; font-size: 0.95rem;">
  <tr style="background-color: #1e1e1e; color: #a78bfa;">
    <th style="padding: 10px; border: 1px solid #333; text-align: left;">Port</th>
    <th style="padding: 10px; border: 1px solid #333; text-align: left;">Service</th>
    <th style="padding: 10px; border: 1px solid #333; text-align: left;">Pentest Use</th>
  </tr>
  <tr>
    <td style="padding: 8px; border: 1px solid #333;"><strong>53</strong></td>
    <td style="padding: 8px; border: 1px solid #333;">DNS</td>
    <td style="padding: 8px; border: 1px solid #333;">Enumerating hosts in the domain.</td>
  </tr>
  <tr>
    <td style="padding: 8px; border: 1px solid #333;"><strong>88</strong></td>
    <td style="padding: 8px; border: 1px solid #333;">Kerberos</td>
    <td style="padding: 8px; border: 1px solid #333;">Kerberoasting & AS-REP Roasting.</td>
  </tr>
  <tr>
    <td style="padding: 8px; border: 1px solid #333;"><strong>445</strong></td>
    <td style="padding: 8px; border: 1px solid #333;">SMB</td>
    <td style="padding: 8px; border: 1px solid #333;">Lateral movement & file shares.</td>
  </tr>
  <tr>
    <td style="padding: 8px; border: 1px solid #333;"><strong>389</strong></td>
    <td style="padding: 8px; border: 1px solid #333;">LDAP</td>
    <td style="padding: 8px; border: 1px solid #333;">AD database queries.</td>
  </tr>
</table>

&nbsp;

---
&nbsp;

## What's Next?
The reconnaissance is done. I have a map of the target, and I know exactly where the vulnerabilities lie. In the next post, I’ll finally pull the trigger on the **Kerberoasting** attack using Impacket to grab that service account hash. Stay tuned!
