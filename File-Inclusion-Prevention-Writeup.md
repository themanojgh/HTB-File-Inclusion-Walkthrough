# File Inclusion Prevention Writeup

This repository documents the exploitation of a File Inclusion Prevention. Here is the step-by-step process of how to do it in the Hack The Box controlled environment.

## Steps

### Step 1: Access the Server
- Use the provided credentials to SSH into the target
```bash
ssh htb-student@10.129.29.112 -p 22
```
- Enter the password: `HTB_@cademy_stdnt!`

### Step 2: Locate the php.ini File
- Apache's php.ini is often located in /etc/php/<version>/apache2/php.ini.
```bash
find / -type f -name 'php.ini' 2>/dev/null | grep apache
```
![image](https://github.com/user-attachments/assets/7dcf3be1-bc99-4ff1-9985-1b7cd1940973)

**Important Considerations**:

- Permissions: Ensure you have the necessary permissions to access the directory and file.

- Multiple Installations: If multiple PHP versions are installed, there may be multiple php.ini files. Make sure you find the one used by Apache.

- Configuration Variations: The exact path may vary depending on the distribution and configuration of the server.

### Step 3: Edit/Add php.ini to disable system()

```bash
sudo nano /etc/php/7.4/apache2/php.ini
disable_functions = system
```
### Step 4: Restart Apache Server
```bash
sudo systemctl restart apache2
```
### Step 5: Create a PHP webshell
```bash
echo "<?php system(\$_GET['cmd']); ?>" | sudo tee /var/www/html/shell.php
```

### Step 6: Attempt to Execute the Webshell
```bash
curl "http://localhost/shell.php?cmd=id"
```

### Step 7: Check the error file to get the answer
```bash
cat /var/log/apache2/error.log
```
![Screenshot 2025-04-15 161909](https://github.com/user-attachments/assets/9ca8c3cc-e1fb-425d-9198-e1d6a61dc656)
