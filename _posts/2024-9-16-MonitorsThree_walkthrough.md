---
layout: post
title: 'MonitorsThree Walkthrough: Conquering Hack The Box Season 6 "MonitorsThree htb"'
date: 16-09-2024
categories: [HackTheBox]
tag: [season 6,machines]
image:
  path: '/Images/moteiresthree/1.png'
---
## Introduction 

__MonitorsThree on HackTheBox is a challenging machine that truly tests your skills. For beginners, tackling MonitorsThree can be both daunting and rewarding. This blog will guide you through the essential steps to conquer this machine, using techniques from hacking and penetration testing. Get ready to dive into the world of CTF challenges and sharpen your hacking abilities. Let’s explore the intricacies of MonitorsThree and uncover strategies for successfully hacking it. Stay tuned for expert tips and tricks to capture that elusive user flag. Good luck on your hacking journey!__ 

## Reconnaissance

Network Scanning: as always, we will do nmap scan to know what is opened ports and it’s services in this machine.as always, we will do nmap scan to know what is opened ports and it’s services in this machine.

![](/Images/moteiresthree/nmap.png)

## Exploration the website

```
Visiting the IP address in a browser redirects us to a website named “monitorsthree.htb”, presenting a form and various navigation options.
```
![](/Images/moteiresthree/2.png)

![](/Images/moteiresthree/3.png)

![](/Images/moteiresthree/4.png)

![](/Images/moteiresthree/5.png)

i will try SQLmap

> sqlmap -r sql.txt --dbs   || then y  .. n 

![](/Images/moteiresthree/databases.png)

> sqlmap -r sql.txt -D monitorsthree_db --tables

![](/Images/moteiresthree/tables.png)

>  sqlmap -r sql.txt -D monitorsthree_db -T users --dump

![](/Images/moteiresthree/passwords.png)

- Now crack the md5 hash 

![](/Images/moteiresthree/cracks.png)

- After i login i didn't find any thing credentials.. 

    so i try to find any sub domains ?

> ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -u http://10.10.11.30 -H "Host: FUZZ.monitorsthree.htb" -mc 200,302 -fs 13556

![](/Images/moteiresthree/cactii.png)

## exploit

- make some search about "cacti". i will use "searchsploit"

![](/Images/moteiresthree/searchsploit.png)

![](/Images/moteiresthree/metas.png)

![](/Images/moteiresthree/1111metas.png)


-now we will search about any important credentials or any good data .

![](/Images/moteiresthree/config11.png)

![](/Images/moteiresthree/download%20config.png)

![](/Images/moteiresthree/config-db.png)

* here try to connect to maraiadb 

![](/Images/moteiresthree/maria1.png)

* i found hash for marcous user 

![](/Images/moteiresthree/maria%202.png)

![](/Images/moteiresthree/john.png)

* i try to su marcus and take the id_rsa form .ssh folder and connect to ssh connection.

> ssh -i id_rsa marcus@10.10.11.30

## Privilege Escalation

- notice the open port 8200 in localhost i do port forwarding  then i found a website.

![](/Images/moteiresthree/open%20ports.png)


![](/Images/moteiresthree/port_fowrwarad.png)

![](/Images/moteiresthree/duplicate.png)

* try to login with credentials i have and i can't so i search about [credentials].
- [Duplicati: Bypassing Login Authentication With Server-passphrase](https://medium.com/@STarXT/duplicati-bypassing-login-authentication-with-server-passphrase-024d6991e9ee)

and i follow the exploit and it work for me .

* download "SQLLitebrowser" and download from the machine this file "Duplicati-server.sqlite".

![](/Images/moteiresthree/sqlite.png)

![](/Images/moteiresthree/e1.png)

![](/Images/moteiresthree/e2.png)

![](/Images/moteiresthree/e3.png)

// disable automatically run backup 

![](/Images/moteiresthree/e5.png)

![](/Images/moteiresthree/e6.png)

![](/Images/moteiresthree/e7.png)

![](/Images/moteiresthree/e8.png)

```
marcus@monitorsthree:~$ ls -al

drwxr-xr-x 2 root   root   4096 Aug 28 00:53 root.txt
-rw-r----- 1 root   marcus   33 Aug 27 23:35 user.txt

marcus@monitorsthree:~$ cd root.txt
marcus@monitorsthree:~/root.txt$ ls
root.txt
```