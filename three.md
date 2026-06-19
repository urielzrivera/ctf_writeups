# Three

**Platform**: Hack The Box \
**Difficulty**: Very Easy\
**Date**: 2026-05-25\
**OS**: Linux\
**IP**: 10.129.19.25

# Machine Tasks

Task 1
 > How many TCP ports are open?
 
`2`
 
```bash
Nmap scan report for 10.129.19.25
Host is up (0.30s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 17:8b:d4:25:45:2a:20:b8:79:f8:e2:58:d7:8e:79:f4 (RSA)
|   256 e6:0f:1a:f6:32:8a:40:ef:2d:a7:3b:22:d1:c7:14:fa (ECDSA)
|_  256 2d:e1:87:41:75:f3:91:54:41:16:b7:2b:80:c6:8f:05 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: The Toppers
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Task 2
 > What is the domain of the email address provided in the "Contact" section of the website?

`thetoppers.htb`

After running the Nmap scan, we can see there is a webpage being served on Port 80, so we can visit it and scroll to the bottom to see the Contact section to find email address: `mail@thetoppers.htb`.

Task 3
 > In the absence of a DNS server, which Linux file can we use to resolve hostnames to IP addresses in order to be able to access the websites that point to those hostnames?
 
 We've ran into this issue in previous machines. When we need to force DNS to resolve to hostnames so we can access sites, we can place the IP address and domain name in our machine's `/etc/hosts/` file.
 
Task 4
 > Which sub-domain is discovered during further enumeration?
 
 From here, the task is guiding us to perform some subdomain enumeration. There are plenty of tools to do this with, but I typically like to use assetfinder to do this. We'll build our command using assetfinder similar to the below:
 
`assetfinder --subs-only $IP > subs.txt`
`amass enum -d example.com -o subdomains.txt`
`gobuster vhost -r --ad -u thetoppers.htb -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark.txt`

```bash
Starting gobuster in VHOST enumeration mode
===============================================================
s3.thetoppers.htb Status: 404 [Size: 21]
```

Task 5
 > Which service is running on the discovered sub-domain?
 
`Amazon S3`


Task 6
 > Which command line utility can be used to interact with the service running on the discovered sub-domain?
 `awscli`

Task 7
 > Which command is used to set up the AWS CLI installation?

Visiting AWS Documentation we can get the answers for Tasks 7 & 8

 > Configuring using AWS CLI commands
 > For general use, the `aws configure` or `aws configure sso` commands in your preferred terminal are the fastest way to set up your AWS CLI installation. 
 
Task 8
 > What is the command used by the above utility to list all of the S3 buckets?
 `aws s3 ls`

Task 9
 > This server is configured to run files written in what web scripting language?
 `PHP` 

---

# Flags

#### User
```text
[REDACTED]
```

#### Root/Admin
```text
flag_here
```

---

# Lessons Learned

**Key takeaway**:\
**Enumeration lesson**:\
**Missed clues**:\
**New/Useful Commands**:
