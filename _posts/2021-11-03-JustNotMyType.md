---
title: WriteUp for Just Not My Type from Killer Queen 2021
layout: post
post-image: "https://user-images.githubusercontent.com/33517160/140023283-c2890a86-2539-45ea-869e-b11140f8acde.png
"
description: A Writeup for the challenge "Just Not My Type" from Killer Queen CTF 2021.
tags:
- writeups
- Web
- CTF
---


[ZeroDayTea]: https://twitter.com/ZeroDayTea

- ## Challenge Information:

| Name             | Category | Difficulty | Points | Dev        |
|------------------|----------|------------|--------|------------|
| Just Not My Type | Web      | Easy       | 248    |[ZeroDayTea]|


# Description: 
I really don't think we're compatible


# Solution:
First thing i thought it was an sqli, but then i remembered they already gave us the source code<br> for the challenge.

### Source-Code:

```php
<h1>I just don't think we're compatible</h1>
<?php
$FLAG = "shhhh you don't get to see this locally";
if ($_SERVER['REQUEST_METHOD'] === 'POST') 
{
    $password = $_POST["password"];
    if (strcasecmp($password, $FLAG) == 0) 
    {
        echo $FLAG;
    } 
    else 
    {
        echo "That's the wrong password!";
    }
}
?>
<form method="POST">
    Password
    <input type="password" name="password">
    <input type="submit">
</form>
```

The twist of the challenge is first we didn't have any link to the webapp, at first 
so the `$FLAG` variable is just a fake flag, so i look and nothing really wrong with 
the code but maybe the function `strcasecmp()` has some kind of vulnerability 
or not used in secure way, after googling a bit and reading the strcasecmp   
php [Documentation](https://www.php.net/manual/en/function.strcasecmp.php)

Turns out that `strcasecmp()` is a single-byte function , after searching what that 
means and how to exploit it found that if you don't use it in a secure way it can lead
to **Authentication Bypass** , the idea is to turn the password param into an empty array and the value to %22%22

Example: `http://vulntarget.com/type.php?password[]=%22%22`

and that gave me the flag :) 

![Pasted image 20211030045525](https://user-images.githubusercontent.com/33517160/139555131-39686fe2-8548-404a-a845-9aa5e97af02b.png)

## Flag: **`flag{no_way!_i_took_the_flag_out_of_the_source_before_giving_it_to_you_how_is_this_possible}`**
