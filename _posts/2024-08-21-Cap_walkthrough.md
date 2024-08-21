---
layout: post
title: 'Cap Walkthrough: Conquering Hack The Box Machines "Cap htb"'
date: 20-08-2024
categories: [HackTheBox]
tag: machines
image:
  path: '/55img/cap.png'  
---
## Introduction

 Cap is a Linux machine of easy difficulty that operates an HTTP server for administrative tasks, including network capture functions. Due to improper access controls, it is vulnerable to Insecure Direct Object Reference (IDOR), allowing unauthorized users to access another user's network capture. This capture contains plaintext credentials, which can be exploited to gain an initial foothold. Further, a Linux capability is used to escalate privileges to root.
 
## Reconnaissance
 
 __Network Scanning:__ as always, we will do nmap scan to know what is opened ports and it’s services in this machine.as always, we will do nmap scan to know what is opened ports and it’s services in this machine.

 ```
  -─(hu1k0㉿kali)-[~/Desktop/HTB]
  └─$ nmap -sV -sC -oN scan.txt 10.10.11.18 
  ===========================================
  Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-21 14:14 EEST
  Stats: 0:00:46 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
  Nmap scan report for 10.10.10.245 (10.10.10.245)
  Host is up (0.15s latency).
  Not shown: 997 closed tcp ports (conn-refused)
  PORT   STATE SERVICE VERSION
  21/tcp open  ftp     vsftpd 3.0.3
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
  | ssh-hostkey: 
  |   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
  |   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
  |_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
    80/tcp open  http    gunicorn
  |_http-title: Security Dashboard
  |_http-server-header: gunicorn

 ```
 - first i will try to login in ftp with "anonymous" credentials.

 ![](/Images/cap/ftp1.png)

## Exploration the website

 ![](/Images/cap/2.png)

 - then i explore and browse the site I found this page with this IDOR you can read more about this vulnerability by clicking => [IDOR](https://portswigger.net/web-security/access-control/idor). 

 ![](/Images/cap/3.png)

 - now i will try to find any interesting data by change the number "3" . start with "0"..

 ![](/Images/cap/pocap.png)

 - it download a file with ".pcap" extension after search i found that. 

 ![](/Images/cap/sss.png)

 - so i open it with wireshak. 

 ![](/Images/cap/wiresharck.png)

# User Flag

 - so i will try to use this credentials to login in ```FTP```

 > ftp 10.10.10.245

 ![](/Images/cap/user.txt.png)
 
 - now we will try to use same credentials to connect with ```ssh``` connection.

 ![](/Images/cap/useer.png)

## Privilege Escalation

 i will use "linpeas.sh" . ypu can read more about ```linpeas.sh``` from [here.](https://github.com/XDev05/PEASS-ng)

 1. fire a server to uplad the file on victem machine. 

 ![](/Images/cap/fire-server.png)

 2. wget to download it on victem machine.

 ![](/Images/cap/lin.png)

 3. > ./linpeas.sh

 ![](/Images/cap/linpaes.png)

 - we found a ```cap_suid```. so we can go for the link in linpeas to esclate it or go to this site [GTFOBins](https://gtfobins.github.io/) and search "python"

 ![](/Images/cap/capp.png)

 - now we can copy and paste it in the shell we got . ^^|Don't Forget To Specify Python Version|^^

 ![](/Images/cap/root.png)

---
![](/Images/cap/1.png)
 