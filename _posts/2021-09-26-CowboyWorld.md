---
title: Writeup for Cowboy World from DownUnder CTF 2021
layout: post
post-image: "https://user-images.githubusercontent.com/33517160/134820324-ed6ff6c0-8379-458a-977c-b07b77353067.png"
description: A writeup for the challenge Cowboy World from DownUnder CTF 2021.
tags:
- writeups
- CTF
- Web
---

- ## Challenge Information:

| - | - |
| ----------- | ----------- |
| Name: | **`Cowboy World`** |
| Category: | **`Web`** |
| Points: | **`100pts`**|

## Description: 
<h4>I heard this is the coolest site for cowboys and can you find a way in?</h4>


![Capture 1](https://user-images.githubusercontent.com/33517160/134733781-d93f214b-a3f5-41d6-928a-122782eb34ee.png)

First thing we get greeted with this normal login form nothing special about it,<br>
before i start doing anything else i decided to check out the `/robots.txt` , i know im on<br> the right track because of the hint provided by the dev

[Hint Link]([https://www.youtube.com/watch?v=fn3KWM1kuAw](https://www.youtube.com/watch?v=fn3KWM1kuAw)).


- ### /robots.txt:

pls no look

User-Agent: regular_cowboys

Disallow: /sad.eml

<br>

we see a `User-Agent` Header but thats just a rabbit hole to try and send requests<br> as `regular_cowboys`, but we also see an email file named `sad.eml` and when we open it with a text editor or outlook we get:

<br>

- ### /sad.eml

Everyone says 'yeee hawwwww'

but never 'hawwwww yeee'  
  
:'(  
  
thats why a 'sadcowboy' is only allowed to go into our website


so now we have a possible username which is `sadcowboy`, to see if the login form has flaws in<br> errors like telling us exactly if the password is wrong or the username, by entering the username `sadcowboy` and a random password i see that we get<br> `Incorrect password` and when i try with another username i get `Incorrect username or password` so now i confirmed the username is correct, and<br> the hint for the password was not clear but i believe it was the `:'(` also when trying `'` we get an<br> `Internal Server Error` so now we know its a sql injection.

so i tried couple sqli login bypass payloads that contains single quotes using the burp repeater trying out payloads in the browser will give an `Internal Server Error` and the payload `'+or+'1'='1` worked with me.


![Pasted image 20210924221939](https://user-images.githubusercontent.com/33517160/134733847-6e8fa801-7aed-4472-9823-cfd38f675f91.png)

## Flag: **`DUCTF{haww_yeeee_downunderctf?}`** 