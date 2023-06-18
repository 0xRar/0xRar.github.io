---
title: Writeup for Zombie from NahamCon CTF 2023 
layout: post
post-image: "https://github.com/0xRar/0xrar.github.io/assets/33517160/bb1f6efb-f058-488f-810e-4074f6e294e7"
description: 
tags:
- Writeups
- CTF
- Misc
---

NahamCon CTF 2023 ! it was a great CTF where we were fighting for the 5th place for couple of hours, but
ended getting tied with the 6th place, and got 7th out of 2518 teams.

<br>

![place-image](https://github.com/0xRar/0xrar.github.io/assets/33517160/40a8d8f7-64ef-457e-8284-ced7a61184c2)


---

## Zombie 
- Category: Misc
- Difficulty: Easy

## Description
Author: @JohnHammond#6971

Oh, shoot, I could have sworn there was a flag here. Maybe it's still alive out there?

## Solution
For this challenge, we are given the credentials to connect to a machine via ssh
and we have to find the flag somehow, so there is a file that had the flag in it
but we can't access it in the meantime.

After connecting to the machine listing all the files in the home directory will
allow us to see a hidden shell script (`.user-entrypoint.sh`)

```shell
user@zombie:~$ ls -la
total 24
drwxr-sr-x    1 user     user          4096 Jun 16 12:37 .
drwxr-xr-x    1 root     root          4096 Jun 14 17:52 ..
-rwxr-xr-x    1 user     user          3846 Jun 14 17:52 .bashrc
-rw-r--r--    1 user     user            17 Jun 14 17:52 .profile
-rwxr-xr-x    1 root     root           131 Jun 14 17:52 .user-entrypoint.sh
```

`.user-entrypoint.sh` content:
```shell
#!/bin/bash

nohup tail -f /home/user/flag.txt >/dev/null 2>&1 & # 
disown

rm -f /home/user/flag.txt 2>&1 >/dev/null

bash -i
exit
```

After searching about some of the commands used here i got interested in the `nohup`
command which is a command used to keep processes running even after exiting the shell or terminal

and listing the proceses will confirm that:

```shell
user@zombie:~$ ps
PID   USER     TIME   COMMAND
    1 root       0:00 /usr/sbin/sshd -D -e
    7 root       0:00 sshd: user [priv]
    9 user       0:00 sshd: user@pts/0
   10 user       0:00 {.user-entrypoin} /bin/bash /home/user/.user-entrypoint.sh
   11 user       0:00 tail -f /home/user/flag.txt
   13 user       0:00 bash -i
   21 user       0:00 ps
```

So now its clearly something to do with a process so i did some reading on Process-Specific Subdirectories
(`/proc`)

Quoting [docs.kernel.org](https://docs.kernel.org/filesystems/proc.html): 
"The directory /proc contains (among other things) one subdirectory for each process running on the system,
which is named after the process ID (PID)."

And Looking into the Process specific entries there is `fd`: `Directory, which contains all file descriptors`,
A file descriptor is an unsigned integer used by a process to identify an open file so now that we know
the pid and what entry to look at we can read `tail -f /home/user/flag.txt` from the file descriptor
for that process.

```shell
user@zombie:~$ cd /proc
user@zombie:/proc$ cd 11
user@zombie:/proc/11$ cd fd
user@zombie:/proc/11/fd$ cat *
flag{6387e800943b0b468c2622ff858bf744}
```

References:
- [https://docs.kernel.org/filesystems/proc.html](https://docs.kernel.org/filesystems/proc.html)
- [https://en.wikipedia.org/wiki/File_descriptor](https://en.wikipedia.org/wiki/File_descriptor) 
- [https://www.ibm.com/docs/en/aix/7.2?topic=n-nohup-command](https://www.ibm.com/docs/en/aix/7.2?topic=n-nohup-command)

## Flag: `flag{6387e800943b0b468c2622ff858bf744}` 
Thank You For Reading ?

