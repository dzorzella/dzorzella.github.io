---
date: '2025-06-03T21:34:48+02:00'
draft: false
title: 'Port Scanner'
author: Zorzella Davide

tags: ["enumeration"]
categories: ["Tools"]

cover:
  image: "port-scanner.jpeg"
  alt: "port-scanner"
  caption: "port-scanner - AI generated image"
---

## Port Scanning with Bash: A simple Script for Network Diagnostics
When working with networks, sometimes you just need a quick way to check if a range of ports is open on a given host. While tools like nmap are powerful and feature-rich, they can be overkill for simple tasks—or not available in minimal environments. That’s where a lightweight Bash script like this comes in handy.  

```bash
#!/bin/bash

host=${1:?"missing arg 1 for host"}
port=${2:?"missing arg 2 for port. Accepted values: [port] | [start-end]"}

if [[ $port =~ ([0-9]+)-([0-9]+) ]]; then
  start=${BASH_REMATCH[1]}
  end=${BASH_REMATCH[2]}
fi

log_message() {
  echo "`date "+%Y-%m-%d %H:%M:%S"` $1" | tee -a log
}

for p in $(seq $start $end);
do
    if timeout 1 bash -c "</dev/tcp/$host/$p" 2>/dev/null; then
      log_message "Port $p is open on $host"
    else
      log_message "Port $p is closed on $host"
    fi
done 
```

You can find the [GitHub project here](https://github.com/dzorzella/port-scanner)

## What it does
This script performs a basic TCP port scan on a given host and port range. It checks each port in the range to see if it’s open and logs the result with a timestamp.

## Key feaures
**Argument Validation**: Ensures both host and port range are provided.  
**Regex Matching**: Extracts start and end ports from a range like 20-25.  
**TCP Check**: Uses Bash’s /dev/tcp pseudo-device to attempt a connection.  
**Timeout Handling**: Prevents hanging on unresponsive ports.  
**Logging**: Outputs results to both the console and a log file with timestamps.  

## Example usage

```bash
./main.sh localhost 8000-8010
```
This will check ports 8000 to 8010 on localhost (127.0.0.1)

```
2025-06-04 09:20:10 Port 8000 is open on localhost
2025-06-04 09:20:10 Port 8001 is closed on localhost
2025-06-04 09:20:10 Port 8002 is closed on localhost
2025-06-04 09:20:10 Port 8003 is closed on localhost
2025-06-04 09:20:10 Port 8004 is closed on localhost
2025-06-04 09:20:10 Port 8005 is closed on localhost
2025-06-04 09:20:10 Port 8006 is closed on localhost
2025-06-04 09:20:10 Port 8007 is open on localhost
2025-06-04 09:20:10 Port 8008 is closed on localhost
2025-06-04 09:20:10 Port 8009 is closed on localhost
2025-06-04 09:20:10 Port 8010 is closed on localhost
```

## Why I wrote this
While this is not as robust or stealthy as dedicated tools like nmap, I created this script as a lightweight, dependency-free way to check port availability during internal network diagnostics. It’s a great example of how powerful Bash can be for quick automation tasks.