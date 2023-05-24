---
title: Writeups for Ducky1 & RevEnv from BYUCTF 2023 
layout: post
post-image: "https://github.com/0xRar/0xrar.github.io/assets/33517160/ff77ab9f-d0a3-46e7-8d5f-18062af3a297"
description: 
tags:
- Writeups
- CTF
- Rev
---



This is our second year of `BYUCTF` last time we were placed 5th,
I don't normally solve `Rev` challenges but for this ctf i tried to solve at least the easy ones
because of rev players shortage in our team and ended up liking it, so hopefully in future i would be solving
these kind of challenges easily!


![placement](https://github.com/0xRar/0xrar.github.io/assets/33517160/4982946e-7552-460b-859b-cb3c55f11062)

---

## Ducky1
- Category: Rev
- Difficulty: Easy

## Description
I recently got ahold of a Rubber Ducky, and have started automating 
ALL of my work tasks with it! You should check it out!

## Solution 
For this challenge we are presented with a file: `inject.bin`, `inject.bin` is a binary payload made with 
DuckyScript for the Malicious USB RubberDucky and one of the useful tools for rubberducky users is the 
<a href="https://ducktoolkit.com/">ducktoolkit</a> you can try to get what commands and text that were
used to create the `inject.bin` by uploading the binary.
<br>
![ducky1](https://github.com/0xRar/0xrar.github.io/assets/33517160/42689b84-eb97-468e-8d62-1150ba30fc4c)

## Flag: `byuctf{this_was_just_an_intro_alright??}`

--- 

## RevEng
- Category: Rev
- Difficulty: Easy

## Description
See if you can find the flag!

## Solution
This challenge is a classic type rev challenge we are givin an executable/ELF64,
running the executable will ask for a passphrase to give us the flag this could be really
easy and solved just by looking into strings and decoding the passphrase. 


```shell
┌──(kali㉿rar)-[~/Desktop/byuctf/rev]
└─$ strings ./gettingBetter 
[...]
Incorrect passphrase. Please try again.
Please enter the correct passphrase to get the flag: 
Congratulations! The flag is %s
;*3$"
Xmj%yzwsji%rj%nsyt%f%sj|y
```

figuring out what kind of cipher it is takes like 10 seconds using <a href="https://www.dcode.fr/cipher-identifier">
dcode.fr's Cipher Identifier</a>

![RevEng](https://github.com/0xRar/0xrar.github.io/assets/33517160/282284bb-42c5-4ab8-8428-6d21e6c8dc8f)
![RevEng-pass](https://github.com/0xRar/0xrar.github.io/assets/33517160/166aa671-6386-490e-b1f3-c6a4b1e9a26f)

- Using `gdb(pwndbg)`:

```shell
pwndbg> disassemble main
[...]
   0x00000000000011bf <+70>:	call   0x1319 <check_passphrase>

pwndbg> break check_passphrase
Breakpoint 1 at 0x131d

pwndbg> run
[...]
	*RDX  0x7fffffffdc50 ◂— 'She turned me into a newt'

pwndbg> x/s $rdx
0x7fffffffdc40:	"She turned me into a newt"
```

<br>
```shell
┌──(kali㉿rar)-[~/Desktop/byuctf/rev]
└─$ ./gettingBetter
Please enter the correct passphrase to get the flag: She turned me into a newt
Congratulations! The flag is byuctf{i_G0t_3etTeR!_1975}
```

## Flag: `byuctf{i_G0t_3etTeR!_1975}`
Thank You For Reading ❤
