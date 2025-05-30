---
date: '2025-05-30T14:08:30+02:00'
draft: false
title: 'Ffuf'
author: Zorzella Davide
tags: ["enumeration", "fuzzers", "virtual hosts", "subdomains"]
categories: ["Tools", "CBBH"]

cover:
  image: "ffuf.png"
  alt: "ffuf"
  caption: "ffuf"
---

**Ffuf (Fuzz Faster U Fool)** is a fast and flexible web fuzzer written in Go. It is primarily used for discovering hidden resources on web servers, such as directories, files, and virtual hosts.

>Tip:
>Taking a look at this wordlists we will notice that they contain copyright comments at the beginning, which can be considered as part of the wordlist and clutter the results. We can use the following in `ffuf` to get rid of these lines with the `-ic` flag.

>Tip:
>To filter results by response size in **Ffuf**, you can use the `-fs` (filter size) option. This allows you to exclude responses that match a specific size. The same applies to the other parameters.

# Public Sub-domains Fuzzing
This technique allows you to identify publicly accessible subdomains associated with a main domain. By using a wordlist of common subdomain names, ffuf performs DNS requests for each entry, dynamically replacing the FUZZ placeholder. This approach is useful for discovering hidden endpoints, test environments, admin panels, or undocumented services that may present opportunities for further security analysis.
```bash
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://FUZZ.$DOMAIN$:$PORT
```

---

# Virtual Host enumeration
This technique leverages the virtual hosting feature of web servers to identify configured but undocumented virtual hosts. In this context, ffuf sends HTTP requests while modifying the Host header with values from a wordlist, looking for different responses that may indicate active virtual hosts. It is especially useful in environments where multiple sites share the same IP address, such as shared hosting or cloud-based setups.
```bash
ffuf -u http://$DOMAIN:$PORT -H "Host: FUZZ.$DOMAIN" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
```

---

# Page fuzzing
Page fuzzing is a core technique in web reconnaissance used to discover hidden or unlinked resources on a web server. By systematically testing different paths and file names, this method helps uncover directories, scripts, and files that may not be directly accessible through the website’s navigation. It can reveal sensitive endpoints, misconfigurations, or forgotten development artifacts. This chapter explores two key sub-techniques: extension fuzzing and recursive page fuzzing, each targeting different aspects of web content enumeration.

## extension fuzzing
This technique focuses on discovering hidden or unlisted files by fuzzing common file extensions. By appending values from a wordlist to a known path (e.g., indexFUZZ), ffuf attempts to identify accessible files such as backups, source code, or configuration files. It's particularly useful for discovering the technologies used to develop the application (e.g. .asp and .aspx for Microsoft IIS, .php for Apache etc.)
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://$DOMAIN:$PORT/indexFUZZ
```

## page fuzzing
Page fuzzing aims to enumerate accessible paths and directories on a web server. Using a wordlist of common directory and file names, ffuf recursively explores the target, optionally appending extensions like .php or .phps. This method helps identify hidden pages, admin panels, or entry points that could be leveraged for further testing. Options like -fs (filter by response size) and -recursion-depth enhance the precision and depth of the scan.
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://$DOMAIN:$PORT/FUZZ -ic -recursion -recursion-depth 3 -e ".php,.phps" -v -fs xxx
```

---

# Directory fuzzing
Directory fuzzing is a technique used to discover hidden or unlisted directories on a web server. By sending HTTP requests with common directory names from a wordlist, ffuf helps identify areas of the site that may not be linked or visible through the user interface. With options like recursion and depth control, this method can explore nested structures, revealing admin panels, configuration folders, or development environments that may be exposed unintentionally.
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://$DOMAIN:$PORT/FUZZ -ic -recursion -recursion-depth 3 -v
```

Additional parameters are:
- `-H`: Add headers (e.g., `-H "Cookie: sessionid=12345"`).
- `-X`: HTTP method (GET, POST, etc.).
- `-d`: POST data (e.g., `-d "username=test&password=FUZZ"`).
- `-t`: Number of concurrent threads.
- `-o`: Output results to a file in various formats (csv, json, html).
- `-mc`: Match response by HTTP status code.
- `-fs`: Filter by response size.
- `-recursion -recursion-depth x`: recursive fuzzing.

---

# Parameter fuzzing
Parameter fuzzing is a technique used to discover hidden or undocumented parameters that a web application may accept. These parameters can influence application behavior, reveal sensitive data, or expose potential vulnerabilities. This chapter explores three key approaches: fuzzing parameter names in GET and POST requests, and fuzzing parameter values to test how the application responds to different inputs.

## GET
This method targets query string parameters in URLs. ffuf replaces the FUZZ placeholder with potential parameter names from a wordlist, helping identify parameters that trigger different responses or reveal hidden functionality.
```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://$DOMAIN:$PORT/?FUZZ=key -fs xxx
```

## POST
Similar to GET fuzzing, this technique focuses on POST requests. By injecting parameter names into the request body, ffuf can uncover backend logic or form fields that are not exposed in the frontend but are still processed by the server.
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://$DOMAIN:$PORT/admin.php -X POST -d "FUZZ=key" -H "Content-Type: application/x-www-form-urlencoded" -fs xxx
```

## Value
Value fuzzing tests how a known parameter (e.g., id) behaves when different values are supplied. This is useful for identifying IDOR (Insecure Direct Object Reference) vulnerabilities, privilege escalation paths, or input validation issues.
```bash
ffuf -w ids.txt:FUZZ -u http://$DOMAIN:$PORT/admin.php -X POST -H 'Content-type: application/x-www-form-urlencoded' -d 'id=FUZZ' -fs xxx
```