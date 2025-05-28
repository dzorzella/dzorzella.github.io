---
date: '2025-05-25T21:06:08+02:00'
draft: false
title: 'Hello World!'
author: Zorzella Davide
categories: ["CTF"]

cover:
  image: "kali-layers.png"
  alt: "<alt text>"
  caption: "<text>"
---

Find your way home

# Enumeration

Add the host to the list of known hosts

```
$ sudo -- sh -c -e "echo '127.0.0.1   home' >> /etc/hosts"
```

Test reachability with local hosts file

```
$ ping home
```

```
PING home (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.077 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.092 ms
64 bytes from localhost (127.0.0.1): icmp_seq=3 ttl=64 time=0.089 ms
64 bytes from localhost (127.0.0.1): icmp_seq=4 ttl=64 time=0.074 ms
^C
--- home ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3104ms
rtt min/avg/max/mdev = 0.074/0.083/0.092/0.007 ms
```

# Exploitation

Check the content of the web page

```
$ curl http://home/
```

```
<!DOCTYPE HTML>
<html>
    <head>
        <title>Home</title>
    </head>
    <body>
        flag{Y0u_Ar3_w3lc0mE}
    </body>
</html>
```

We obtained the flag!