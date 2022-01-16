---
title: Discord Phishing Link Analysis
layout: post
post-image: "https://user-images.githubusercontent.com/33517160/149678693-7b1dd64a-61d8-4f45-8a3a-fb972352f141.png"
description: Analysis of a phishing link being shared in discord currently 
tags:
- DFIR
- Phishing
- Analysis
---

# Analysis of (http[:]//disccrdapp [.]com/newyears) Phishing Link

## Info:
- Target: discord.com
- Phish Domain: disccrdapp[.]com
- IP: 190.115.18.199
- DNS Record: AS262254 - DDOS-GUARD CORP
- Domain Registrar: REG.RU LLC
- Location: Belize City, Belize

## Victim Clicked on Link Using:
- **Device: Android Phone**

## Details:
This phishing attack scenario happens when the victim clicks on the link accidentally or otherwise
it spreads it self by sending the link to the discord friends & joined servers, the link did not ask
the victim to enter credintionls in order to send the link or steal any tokens or credentials,
this phishing attack is meant to give the users(victims) a free month of discord nitro which costs
9.99$ USD Dollars monthly as if it was brought to you by steam,

This way they can spread the link by discord and or stealing your discord credentials and
your steam account credentials and potentially making profit from your steam inventory
, also from the [urlscan.io] the link uses nextcord which is a discord api used to make
discord bots so maybe it uses it to make an http request to the server and send the harvested
creds via discord, but thats just a possibility.

## How do they fool victims:
- Using a domain name really close to the real domain or related.
- Cloning the real discord nitro page.
- Making them think they don't have to pay and promising them something they want(nitro).
- Using the exact same embed photo for links as the real discord.

## Scans & Screenshots:
### Phish link homepage:<br>
<img height="300px" width="500px" src="https://user-images.githubusercontent.com/33517160/149651330-82ce903d-80ab-4c16-b6e0-3955575ea447.png">

----------

### Embeds:
<img align="left" src="https://user-images.githubusercontent.com/33517160/149650997-1591d545-7e82-4afd-98e1-d1d3944de84f.png">
<img align="center" src="https://user-images.githubusercontent.com/33517160/149651055-01f9be29-edef-4d2f-900f-4cbad22a6bd6.png">

----------

### urlscan:
<img align="center" src="https://user-images.githubusercontent.com/33517160/149654873-8a4d9662-ed21-44e0-9ff4-6aa01b2560a6.png">

----------

### Virus Total:
<img align="left" height="300px" width="500px" src="https://user-images.githubusercontent.com/33517160/149655310-5bbd6436-5265-4ea5-aa76-b22b0bbde349.png">
<img align="center" height="300px" src="https://user-images.githubusercontent.com/33517160/149655390-72b2a895-c7f0-4a77-9549-e8eafa88f56a.png">

---------
[Virus Total]: https://www.virustotal.com/gui/url/509f8ab5426cd5fdf54c3289e99f3e49f3275748e1b3793c6c80f97f8d838540
[urlscan.io]: https://urlscan.io/result/f6f39dbe-bfb7-4d8a-95ff-c0800bc8885d/

### Scan Results Links:
- VT: [Virus Total]
- urlscan: [urlscan.io]

If you want to learn how to to protect yourself you can read this post on the malwarebytes blog:
https://blog.malwarebytes.com/scams/2021/10/discord-scammers-lure-victims-with-promise-of-free-nitro-subscriptions/
