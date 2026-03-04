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
I’m running everything in a virtual environment to keep it safe and separate from my main files:

<ul style="list-style-type: none; padding: 0;">
  <li style="margin-bottom: 15px; display: flex; align-items: flex-start;">
    <details style="cursor: pointer; width: 100%;">
      <summary style="list-style: none; outline: none; display: flex; align-items: center; justify-content: space-between;">
        <span><strong>The Domain Controller:</strong> Windows Server 2022</span>
        <span style="font-size: 0.8rem; color: #a78bfa; border: 1px solid #a78bfa; padding: 2px 8px; border-radius: 4px; margin-left: 10px;">Details</span>
      </summary>
      <div style="margin-top: 15px; display: flex; gap: 10px; justify-content: flex-start;">
        <a href="https://github.com/user-attachments/assets/c0e9423b-0dca-44bc-87af-6639927500c1" target="_blank">
          <img src="https://github.com/user-attachments/assets/c0e9423b-0dca-44bc-87af-6639927500c1" 
               alt="Windows Server 2022 Setup" 
               style="width: 500px; border-radius: 8px; border: 1px solid #333;" />
        </a>
      </div>
    </details>
  </li>
  
  <li style="margin-bottom: 15px; display: flex; align-items: flex-start;">
    <details style="cursor: pointer; width: 100%;">
      <summary style="list-style: none; outline: none; display: flex; align-items: center; justify-content: space-between;">
        <span><strong>The Victim:</strong> Windows 11 Pro</span>
        <span style="font-size: 0.8rem; color: #a78bfa; border: 1px solid #a78bfa; padding: 2px 8px; border-radius: 4px; margin-left: 10px;">Details</span>
      </summary>
      <div style="margin-top: 15px; display: flex; gap: 10px; justify-content: flex-start;">
        <a href="https://github.com/user-attachments/assets/c6893e2e-3da9-4d32-b305-31fad852f967" target="_blank">
          <img src="https://github.com/user-attachments/assets/c6893e2e-3da9-4d32-b305-31fad852f967" 
               alt="Windows 11 Pro Setup" 
               style="width: 250px; border-radius: 8px; border: 1px solid #333;" />
        </a>
      </div>
    </details>
  </li>

  <li style="margin-bottom: 15px; display: flex; align-items: flex-start;">
    <details style="cursor: pointer; width: 100%;">
      <summary style="list-style: none; outline: none; display: flex; align-items: center; justify-content: space-between;">
        <span><strong>The Attacker:</strong> Arch Linux (Host Machine)</span>
        <span style="font-size: 0.8rem; color: #a78bfa; border: 1px solid #a78bfa; padding: 2px 8px; border-radius: 4px; margin-left: 10px;">Details</span>
      </summary>
      <div style="margin-top: 15px; background: transparent; text-align: left;">
        <a href="https://github.com/user-attachments/assets/2a71da84-c60c-4d12-bdb2-04723fdb8f07" target="_blank">
          <img src="https://github.com/user-attachments/assets/2a71da84-c60c-4d12-bdb2-04723fdb8f07" 
               alt="Arch Linux Info" 
               style="max-width: 600px; width: 100%; border-radius: 4px; border: 1px solid #333;" />
        </a>
      </div>
    </details>
  </li>
  <li style="margin-bottom: 15px;"><strong>Hypervisor:</strong> VMware Workstation</li>
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
&nbsp;

# Phase 4: Setting up the Workstation 
<div style="width: 100%; border: none; padding: 0;">
  <div style="font-size: 1.1rem; line-height: 1.6; margin-bottom: 20px;">
    <strong>The Victim:</strong> Windows 11 Pro (WKSTN01)
    <p>Now that we have a system in which we can integrate and implement the users in, its time to set up a Workstation.<br>
       For this I will simulate another virtual environment using Windows 11 Pro. This machine will be joined to the lab.local domain.</p>
  </div>
  
  <div style="text-align: center;">
    <a href="https://github.com/user-attachments/assets/c6893e2e-3da9-4d32-b305-31fad852f967" target="_blank">
      <img src="https://github.com/user-attachments/assets/c6893e2e-3da9-4d32-b305-31fad852f967" alt="Windows 11 Pro Setup" style="width: 650px; border-radius: 8px; border: 1px solid #333;" />
    </a>
  </div>
</div>

&nbsp;

### Step 1: Networking & DNS Configuration
The most critical part of joining a domain is DNS. Without pointing the Workstation to our Domain Controller, it will never find <strong><em>lab.local</em></strong>. I manually configured the IPv4 settings to use **10.0.0.10** as the primary DNS server.

<div style="text-align: center; margin-top: 15px;">
  <a href="https://github.com/user-attachments/assets/55e61ff0-4fdf-4ca1-b1b7-882b268ac611" target="_blank">
    <img src="https://github.com/user-attachments/assets/55e61ff0-4fdf-4ca1-b1b7-882b268ac611" alt="IPv4 Configuration" style="width: 650px; border-radius: 8px; border: 1px solid #333;" />
  </a>
</div>

&nbsp;

### Step 2: Troubleshooting the Connection
Initially, I couldn't ping the Domain Controller. This was due to two issues:
1. **Network Isolation:** One VM was set to NAT and the other to Host-Only. I had to move both to the same **Host-Only** adapter to allow them to communicate.
2. **Firewall Blocking:** Windows Server blocks ICMP (Ping) by default. I used the following command on the DC to enable it:

<pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px; overflow-x: auto;">
PS C:\> Enable-NetFirewallRule -DisplayName "File and Printer Sharing (Echo Request - ICMPv4-In)"
</pre>

Once the firewall was open, the pings started flowing perfectly!

<div style="text-align: center; margin-top: 15px;">
  <a href="https://github.com/user-attachments/assets/d59f0664-c23a-4908-af53-b699a65514ec" target="_blank">
    <img src="https://github.com/user-attachments/assets/d59f0664-c23a-4908-af53-b699a65514ec" alt="Successful Ping" style="width: 650px; border-radius: 8px; border: 1px solid #333;" />
  </a>
</div>

&nbsp;

### Step 3: Joining the Domain
With connectivity established, it was time for the final step. I went into the system settings, changed the membership from a Workgroup to the **lab.local** domain, and authenticated with the Administrator credentials.

<div style="text-align: center; margin-top: 15px;">
  <a href="https://github.com/user-attachments/assets/6ed3ea52-d78f-475c-bb4d-3b9c2c31d462" target="_blank">
    <img src="https://github.com/user-attachments/assets/6ed3ea52-d78f-475c-bb4d-3b9c2c31d462" alt="Domain Join Settings" style="width: 650px; border-radius: 8px; border: 1px solid #333;" />
  </a>
</div>

&nbsp;

### Success!
The machine is now officially part of the lab. A quick reboot and a "Welcome" message later, I am ready to start testing my Active Directory users on a real target machine.

<div style="text-align: center; margin-top: 15px;">
  <img src="https://github.com/user-attachments/assets/231eb4cd-987e-46a1-91e2-51c338dafe9a" alt="Welcome to Domain" style="width: 650px; border-radius: 8px; border: 1px solid #333;" />
</div>

&nbsp;

# Phase 5: Vulnerability Engineering
Now that the foundation is solid and the workstation is joined, the "Building" part is over. It’s time to intentionally weaken the security of this domain to simulate real-world misconfigurations. My first target: **Kerberoasting**.

### Setting the Bait (SPN)
Hacker look for accounts with a **Service Principal Name (SPN)** because these accounts can be "roasted" to request Kerberos tickets, which can then be cracked offline. I assigned a fake SQL service name to the <strong><em>sql_svc</em></strong> user I created earlier:

<pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px; overflow-x: auto;">
PS C:\> setspn -a MSSQLSvc/sql01.lab.local:1433 sql_svc
</pre>

<div style="text-align: center; margin-top: 15px;">
  <a href="https://github.com/user-attachments/assets/27f104f3-7fd7-4901-9951-25bdc46660ae" target="_blank">
    <img src="https://github.com/user-attachments/assets/27f104f3-7fd7-4901-9951-25bdc46660ae" alt="Set SPN Command" style="width: 650px; border-radius: 8px; border: 1px solid #333;" />
  </a>
  <p style="margin-top: 10px; font-style: italic; color: #666;">
    Successfully registered the SPN. The lab is now officially "vulnerable".
  </p>
</div>

&nbsp;

### Why this matters
In a professional environment, service accounts often have weak passwords. By setting this SPN, I've created a path for an attacker to move from a standard user to cracking a service account password without even touching the Domain Controller directly.

---
&nbsp;

# Phase 6: Reconnaissance & Connectivity
Now that the Windows environment is "vulnerable," I need to step into the shoes of the attacker. Since my main OS is **Arch Linux**, I’m using it as the attack platform. However, there was one final hurdle: getting my host machine to talk to the isolated **Host-Only** network.

### Bringing the Attacker into the Network
My VMs were living in the <strong>10.0.0.0/24</strong> subnet, but my Arch host's virtual interface (<strong>vmnet1</strong>) was on a completely different range. To bridge this gap, I manually assigned an IP to my host within the lab's subnet:

<pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px; overflow-x: auto;">
$ sudo ip addr add 10.0.0.50/24 dev vmnet1
</pre>

<div style="text-align: center; margin-top: 15px;">
  <a href="https://github.com/user-attachments/assets/770d448d-e37e-464a-a44a-5325c49dcc81" target="_blank">
    <img src="https://github.com/user-attachments/assets/770d448d-e37e-464a-a44a-5325c49dcc81" alt="Host IP Config" style="width: 550px; border-radius: 8px; border: 1px solid #333;" />
  </a>
</div>

&nbsp;

### The Handshake
With the IP set, I performed a simple connectivity test. Pinging the Domain Controller (<strong>10.0.0.10</strong>) from my Arch terminal confirmed that the "Attacker" now has a direct line of sight to the target.

<div style="text-align: center; margin-top: 15px;">
  <a href="https://github.com/user-attachments/assets/8085fe9b-3446-409a-85fb-09449d35b901" target="_blank">
    <img src="https://github.com/user-attachments/assets/8085fe9b-3446-409a-85fb-09449d35b901" alt="Ping Success" style="width: 600px; border-radius: 8px; border: 1px solid #333;" />
  </a>
</div>

&nbsp;

### Mapping the Target (Nmap Scan)
With connectivity confirmed, it was time for the first real "hacker" move: **Network Scanning**. I ran <strong>nmap</strong> against the Domain Controller to see exactly what services were exposed.

<pre style="font-family: monospace; line-height: 1.2; background: #1e1e1e; padding: 20px; color: #a78bfa; border: 1px solid #333; border-radius: 5px; overflow-x: auto;">
$ nmap -Pn -sV 10.0.0.10
</pre>

<div style="text-align: center; margin-top: 15px;">
  <a href="https://github.com/user-attachments/assets/adb6991a-5ba1-43bb-b30c-d656d589f444" target="_blank">
    <img src="https://github.com/user-attachments/assets/adb6991a-5ba1-43bb-b30c-d656d589f444" alt="Nmap Scan Results" style="width: 800px; border-radius: 8px; border: 1px solid #333;" />
  </a>
</div>

&nbsp;

### The Loot: Important Ports
The scan confirmed these main entry points for the upcoming AD attacks:

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

---
&nbsp;

## What's Next?
The reconnaissance is done. I have a map of the target, and I know exactly where the vulnerabilities lie. In the next post, I’ll finally pull the trigger on the **Kerberoasting** attack using Impacket to grab that service account hash. Stay tuned!
