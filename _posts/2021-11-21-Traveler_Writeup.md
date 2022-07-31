---
title: Writeup for Traveler from AtHack CTF Quals 2021
layout: post
post-image: "https://user-images.githubusercontent.com/33517160/142748375-b5c7f1ce-05bd-4f28-94e9-a6cffa822a96.png"
description: A writeup for the web challenge Traveler from AtHack CTF Quals 2021.
tags:
- writeups
- CTF
- Web
- AtHack
---

# Writeup for the challenge **_`Traveler`_** from AtHack CTF Quals 2021
----
- ## Challenge Information:

| Name        | Category | Difficulty | Points |
|-------------|----------|------------|--------|
| Traveler    | Web      | Easy       | 100pts |

----

# Description: 
Ready for a quick external assessment?

----

# Solution:
When we go to the link we get:

![traveler-web athack-ctf com](https://user-images.githubusercontent.com/33517160/142747644-3f591d86-7a04-4f49-a22b-3c8deb763fd8.png)

From first sight we knew it was a bypass challenge, because also the admins confirmed it they said "do not waste your time by directory bruteforcing" so we need to bypass the 403 Forbidden status code  somehow.

what i did at first is i tried couple of ways to bypass the 403 Status Code this github repo was of good use ["How To Hunt:Status_Code_Bypass"](https://github.com/KathanP19/HowToHunt/tree/master/Status_Code_Bypass) until i stumpled upon this ["403 & 401 Bypasses CheatSheet"](https://book.hacktricks.xyz/pentesting/pentesting-web/403-and-401-bypasses) and i already tried changing the request methods and the basic http headers like `X-Forwarded-For:` and the the site had path protection so adding `X-Original-URL: /admin` would be a good choice as the bypasses CheatSheet says.

so the bypass worked by adding the header `X-Original-URL: /admin` to our request
initialy giving us access to an input that requests a filename by the looks of it with the parameter:   `filename=` so for this part of the challenge its kinda guessable that the input is vulnerable to LFI the real question is there any filtering?

and the answer to that is yes there is a filter that replaces `../` appearnatly so any payloads that can bypass this filter can work, after trying multiple locations i tried `/home/flag.txt`.

Payload:` ....//....//....//....//....//....//....//....//....//....//home/flag.txt`

![](https://user-images.githubusercontent.com/33517160/142747695-31ce1e6c-dcff-4441-bf3e-6d1be5d99ccd.png)


## Flag: **`AtHackCTF{E@SY_!S_V3RY_E@SY}`**
