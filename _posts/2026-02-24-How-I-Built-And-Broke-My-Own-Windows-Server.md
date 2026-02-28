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
<meta property="og:image" content="https://github.com/user-attachments/assets/7cdd01e9-4afb-4db8-b73f-0c4eb3db1447">
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
Active Directory is pretty much everywhere in the corporate world, so I wanted to get some hands-on time with it. By building the whole thing myself, I can see how everything connects and, more importantly, what kind of small mistakes lead to a network getting compromised.

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
I’m running everything in a virtual environment to keep it safe and separate from my main files:

<ul style="list-style-type: none; padding: 0;">
  <li style="margin-bottom: 15px; display: flex; align-items: flex-start;">
    <details style="cursor: pointer; width: 100%;">
      <summary style="list-style: none; outline: none; display: flex; align-items: center; justify-content: space-between;">
        <span><strong>The Domain Controller:</strong> Windows Server 2022</span>
        <span style="font-size: 0.8rem; color: #a78bfa; border: 1px solid #a78bfa; padding: 2px 8px; border-radius: 4px; margin-left: 10px;">Details</span>
      </summary>
      <div style="margin-top: 15px; display: flex; gap: 10px; justify-content: flex-start;">
        <a href="https://github.com/user-attachments/assets/e2210227-c3ae-4a7f-a955-c567abbd9068" target="_blank">
          <img src="https://github.com/user-attachments/assets/e2210227-c3ae-4a7f-a955-c567abbd9068" 
               alt="Windows Server 2022 Setup" 
               style="width: 250px; border-radius: 8px; border: 1px solid #333;" />
        </a>
        <a href="https://github.com/user-attachments/assets/720145a4-85fe-476e-b530-cccdbebf91b5" target="_blank">
          <img src="https://github.com/user-attachments/assets/720145a4-85fe-476e-b530-cccdbebf91b5" 
               alt="Storage" 
               style="width: 250px; border-radius: 8px; border: 1px solid #333;" />
        </a>
      </div>
    </details>
  </li>
  <li style="margin-bottom: 15px;"><strong>The Victim:</strong> Windows 10 Pro</li>
  <li style="margin-bottom: 15px; display: flex; align-items: flex-start;">
    <details style="cursor: pointer; width: 100%;">
      <summary style="list-style: none; outline: none; display: flex; align-items: center; justify-content: space-between;">
        <span><strong>The Attacker:</strong> Arch Linux (Host Machine)</span>
        <span style="font-size: 0.8rem; color: #a78bfa; border: 1px solid #a78bfa; padding: 2px 8px; border-radius: 4px; margin-left: 10px;">Details</span>
      </summary>
      <div style="margin-top: 15px; background: #000; border-radius: 8px; border: 1px solid #333; overflow: hidden; text-align: center;">
        <a href="https://github.com/user-attachments/assets/2a71da84-c60c-4d12-bdb2-04723fdb8f07" target="_blank">
          <img src="https://github.com/user-attachments/assets/2a71da84-c60c-4d12-bdb2-04723fdb8f07" alt="Arch Linux Info" />
        </a>
      </div>
    </details>
  </li>
  <li style="margin-bottom: 15px;"><strong>Hypervisor:</strong> VMware Workstation / VirtualBox</li>
</ul>

---
&nbsp;

# Phase 1: Basic Config
The first thing I had to do was give the server a permanent IP and a clear name. I used **SConfig** to handle the basics:

<ol>
  <li><strong>Static IP:</strong> I assigned <em><strong>10.0.0.10/24</strong></em> so the server always stays in the same spot.</li>
  <li><strong>Hostname:</strong> I renamed the server to <em><strong>DC01</strong></em> so it’s easy to recognize.</li>
</ol>

<div style="display: flex; gap: 10px; margin-top: 15px;">
  <a href="https://github.com/user-attachments/assets/a46a1b92-5a62-44db-93f0-b583869bf1ea" target="_blank">
    <img width="495" alt="Sconfig Name" src="https://github.com/user-attachments/assets/a46a1b92-5a62-44db-93f0-b583869bf1ea" style="border-radius: 8px; border: 1px solid #333;" />
  </a>
  <a href="https://github.com/user-attachments/assets/0cf1a9d1-adaf-4736-b2a3-ee3c73b0aa39" target="_blank">
    <img width="495" alt="Sconfig IP" src="https://github.com/user-attachments/assets/0cf1a9d1-adaf-4736-b2a3-ee3c73b0aa39" style="border-radius: 8px; border: 1px solid #333;" />
  </a>
</div>
&nbsp;

# Phase 2: Active Directory Setup

Now that the name and IP are set, it’s time to actually turn the server into a Domain Controller. I used PowerShell for this because it’s a lot quicker than clicking through a bunch of menus.

<ol>
  <li><strong>Installing AD Tools:</strong> I jumped into PowerShell (Option 15 in SConfig) and ran this to get the Active Directory files installed:
    <pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px; overflow-x: auto;">PS C:\> Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools</pre>
  </li>
  <li><strong>Promoting to Forest:</strong> Once the tools were ready, I promoted the server to a Domain Controller for <em><strong>lab.local</strong></em>.
    <pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px; overflow-x: auto;">PS C:\> Install-ADDSForest -DomainName "lab.local" -InstallDns:$true -Force:$true</pre>
  </li>
</ol>

<div style="margin-top: 20px; text-align: center;">
  <a href="https://github.com/user-attachments/assets/2ca0371e-b478-4b37-8af6-bd6c3d5d9663" target="_blank">
    <img width="89%" alt="AD Setup Progress" src="https://github.com/user-attachments/assets/2ca0371e-b478-4b37-8af6-bd6c3d5d9663" style="border-radius: 8px; border: 1px solid #333;" />
  </a>
  <p style="margin-top: 10px; font-style: italic; color: #666;">
    After the command finished, the server rebooted itself to finalize everything.
  </p>
</div>

&nbsp;

# Phase 3: Populating the Domain

A Domain Controller without users is pretty boring, so I needed to add some life to the network.

<ol>
  <li><strong>Organizational Units:</strong> I created a folder called <strong><em>Lab_Users</em></strong> to keep things organized.
    <pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px; overflow-x: auto; margin-top: 10px;">PS C:\> New-ADOrganizationalUnit -Name "Lab_Users" -Path "DC=lab,DC=local"</pre>
  </li>

  <li style="margin-top: 20px;"><strong>Creating Test Users:</strong> I added a few different roles. To make things easier, I stored the password in a variable first:
    <pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px; overflow-x: auto; margin-top: 10px;"># Storing the password securely in a variable
PS C:\> $password = ConvertTo-SecureString "Password123!" -AsPlainText -Force</pre>
    <p style="margin-top: 15px;">Then, I used that variable to create the actual accounts in one go:</p>
    <pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px; overflow-x: auto; margin-top: 10px;"># Creating the accounts using the variable
PS C:\> New-ADUser -Name "SQL Service" -SamAccountName "sql_svc" -UserPrincipalName "sql_svc@lab.local" -Path "OU=Lab_Users,DC=lab,DC=local" -AccountPassword $password -Enabled $true
PS C:\> New-ADUser -Name "IT Admin" -SamAccountName "it_admin" -UserPrincipalName "it_admin@lab.local" -Path "OU=Lab_Users,DC=lab,DC=local" -AccountPassword $password -Enabled $true
PS C:\> New-ADUser -Name "HR Manager" -SamAccountName "hr_user" -UserPrincipalName "hr_user@lab.local" -Path "OU=Lab_Users,DC=lab,DC=local" -AccountPassword $password -Enabled $true</pre>
  </li>

  <li style="margin-top: 20px;"><strong>Setting Up Targets:</strong> I added the <strong><em>it_admin</em></strong> to the <strong><em>Domain Admins</em></strong> group for future testing.
    <pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px; overflow-x: auto; margin-top: 10px;">PS C:\> Add-ADGroupMember -Identity "Domain Admins" -Members "it_admin"</pre>
  </li>
</ol>

<div style="margin-top: 20px; text-align: center;">
  <a href="https://github.com/user-attachments/assets/7cdd01e9-4afb-4db8-b73f-0c4eb3db1447" target="_blank">
    <img width="89%" alt="AD Users and Roles" src="https://github.com/user-attachments/assets/7cdd01e9-4afb-4db8-b73f-0c4eb3db1447" style="border-radius: 8px; border: 1px solid #333;" />
  </a>
  <p style="margin-top: 10px; font-style: italic; color: #666;">
    Successfully populated the Lab_Users OU with different roles.
  </p>
</div>
