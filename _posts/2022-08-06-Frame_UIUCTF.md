---
title: Writeup for Frame from UIUCTF 2022
layout: post
post-image: "https://user-images.githubusercontent.com/33517160/183234305-6bcd4448-cacf-43e9-9df6-9fd167253cc9.png"
description: Writeup for the web challenge Frame from UIUCTF 2022
tags:
- Writeups
- CTF
- Web
---


UIUCTF is a capture the flag competition run by [SIGPwny](https://sigpwny.com/) from the University of Illinois at Urbana-Champaign.

## Description:
We made it easy to add a frame to your digital art!

[https://frame-web.chal.uiuc.tf/](https://frame-web.chal.uiuc.tf/)

**authors**: Emma + Minh


## Source-Code:
```php
<?php
ini_set('display_errors', 1);
ini_set('display_startup_errors', 1);
error_reporting(E_ALL);
          if (isset($_POST["submit"])) {
            $allowed_extensions = array(".jpg", ".jpeg", ".png", ".gif");
            $filename = $_FILES["fileToUpload"]["name"];
            $tmpname = $_FILES["fileToUpload"]["tmp_name"];
            $target_file = "uploads/" . bin2hex(random_bytes(8)) . "-" .basename($filename);

            $has_extension = false;
            foreach ($allowed_extensions as $extension) {
              if (strpos(strtolower($filename), $extension) !== false) {
                $has_extension = true;
              }
            }
            
            if ($_FILES["fileToUpload"]["size"] < 2000000) {
              if (getimagesize($tmpname) && $has_extension) {
                if (move_uploaded_file($tmpname, $target_file)) {     
                  echo "<div id='frame'><img src='$target_file' alt='Your image failed to load :(' id='submission'></div>";
                } else {
                  echo "There was an error uploading your file. Please contact an admin.";
                }
              } else {
                echo "Your picture is not a picture and could not be framed.";
              }
            } else {
              echo "Your picture is too large for us to process.";
            }
          }
        ?>
```

## Solution:
`Frame` had the tags of (`web, php, beginner`) and it was an easy challenge, this challenge had 146 Solves at the time of writing this post though i solved it much earlier, although i didn't notice there was a source code which took me sometime to get the flag.

![Pasted image 20220731183223](https://user-images.githubusercontent.com/33517160/183234151-6df9df66-29a8-41ad-811e-b0b01f002297.png)


this php webapp is just an upload form that lets users upload an image and display the image but only for the extentions (`.jpg, .jpeg, .png, .gif`) within a frame, so the first thing every player will do is try and upload a webshell, at first i tried uploading a php webshell(`Shell.php`) but there is a checker on the file extention and the content it self, after doing some bypass techniques such as (`Shell.png.php`) etc. 

but than i uploaded a webshell which worked, using the` GIF89a;` header, if the content is being scanned sometimes it can be fooled by putting this header on top of the shellcode, though because of the use of the function `getimagesize()` there might be another way to upload a shell.

this is how my request looked:

![Pasted image 20220802201102](https://user-images.githubusercontent.com/33517160/183234234-65fc575c-fb94-4c4a-84e8-464dc9f2e41c.png)


i changed the `Content-Type` as well just to make it look more like a gif but it might not have been required to upload the shell

and BOOM! we got a shell, after looking for a minute i found the flag in the `/` directory.

![Pasted image 20220802201621](https://user-images.githubusercontent.com/33517160/183234268-f7398145-e91a-4731-a150-1c9b5d90b526.png)


Ref:
https://vulp3cula.gitbook.io/hackers-grimoire/exploitation/web-application/file-upload-bypass

## Flag: `uiuctf{th1nk1ng_0uts1de_th3_fr4m3}`
*Thank You For Reading ♥*
