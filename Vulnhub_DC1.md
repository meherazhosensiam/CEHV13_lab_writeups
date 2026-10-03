# VulnHub DC:1 Lab Report

**Name:** Meheraz Hossen Siyam
**Lab Name:** DC:1
**Target:** Drupal
**Target IP:** 192.168.0.105
**Target MAC Address:** F0:B6:1E:A9:98:8B
**Open Ports:** 22, 80, 111

## Introduction

For this lab, I worked on the VulnHub DC:1 machine. The main goal was to find the vulnerabilities in the machine, get initial access, and then escalate my privileges until I could get root access.

I used **Nmap** for scanning, **Metasploit** for gaining access, and **GTFOBins** for researching the privilege-escalation technique.

I did not follow a fixed exploitation path from the beginning. I used the information from each flag and the system itself to decide what to investigate next.

## 1. Scanning the Target

First, I scanned the target using Nmap to find the available ports and services.

The target IP was:

```text
192.168.0.105
```

The scan showed these open ports:

```text
22/tcp    SSH
80/tcp    HTTP
111/tcp   RPC
```

The HTTP service on port 80 was the most interesting because the target was running **Drupal**, which gave me a possible entry point.

I used the information from the Nmap scan to investigate the web application and its vulnerabilities.

## 2. Getting Initial Access

After identifying Drupal as the target application, I used Metasploit to find and use the relevant Drupal exploit.

The exploit gave me access to the target machine.

At this point, I had a foothold on the system, but I still needed to investigate the machine further and find the flags.

## 3. Finding the First Flag

After getting access, I started looking through the system and found the first flag.

The first flag contained the message:

> "Every good CMS contains a configuration file."

This was a hint rather than just a flag. It suggested that I should investigate the Drupal configuration files.

I followed the hint and checked the Drupal configuration.

## 4. Finding Database Information

Inside the Drupal configuration, I found database-related information.

This was useful because the database contained information belonging to the Drupal installation.

I then investigated the database and its tables.

While examining the database, I found information related to the flags and the Drupal users.

This part of the lab showed me why application configuration files are important during post-exploitation. A configuration file may contain database credentials or other information that can lead to further access.

## 5. Investigating the Drupal Database

After accessing the database, I examined the available tables and the information stored inside them.

I found the relevant flag information in the database.

I also found information related to the Drupal administrator account.

The database investigation helped me understand how the web application, its configuration, and its database are connected.

## 6. Changing the Administrator Credential

While examining the Drupal user information, I found the administrator account.

I modified the administrator credential and set the password to:

```text
admin
```

I then used the updated credential to access the administrator account.

This allowed me to continue the investigation and find the next flag.

This part of the lab demonstrated the security impact of having access to application databases. If an attacker gets database access and the application does not properly protect authentication information, they may be able to take over user accounts.

## 7. The Misconfiguration Hint

After obtaining the next flag, I received another clue:

> "Check the system for misconfigurations."

This changed the direction of my investigation.

Instead of continuing to look only at the Drupal application, I started looking at the Linux system itself.

I searched for files and binaries that had special privileges.

One of the commands I used to investigate the filesystem was:

```bash
find . -exec /bin/sh \; -quit
```

This command resulted in a shell, which helped me continue investigating the system.

I also investigated privileged binaries and used **GTFOBins** to understand whether a binary with elevated privileges could be abused for privilege escalation.

## 8. Privilege Escalation

The important thing I learned from this stage was that getting a normal shell is not the same as having full control of the machine.

After obtaining initial access, I had to look for ways to move from the current user to a higher-privileged user.

I found a binary with elevated privileges and checked it against GTFOBins.

GTFOBins provided information about how certain Unix binaries can be abused when they have inappropriate privileges or are available in a privileged context.

Using the technique I found, I was able to escalate my privileges.

I verified the final privilege level with:

```bash
whoami
```

The result was:

```text
root
```

This confirmed that I had successfully obtained root access to the DC:1 machine.

## 9. Attack Path

The overall path I followed was roughly:

```text
Nmap
  ↓
Open ports discovered
  ↓
Drupal identified
  ↓
Metasploit
  ↓
Initial access
  ↓
First flag
  ↓
Drupal configuration file
  ↓
Database information
  ↓
Database/table enumeration
  ↓
Administrator credentials
  ↓
Administrator access
  ↓
Next flag
  ↓
"Check for misconfigurations"
  ↓
Privileged binaries
  ↓
GTFOBins
  ↓
Privilege escalation
  ↓
Root
```

## 10. What I Learned

The main thing I learned from this lab is that one vulnerability or one piece of information can lead to another part of the system.

At first, I only knew that the machine had several open ports. After investigating the web service, I found Drupal and used it to gain access.

The first flag then gave me a direction to investigate the CMS configuration. The configuration led me to database information, and the database gave me more information about the Drupal installation and administrator account.

Later, another flag pointed me toward system misconfigurations. That moved the investigation from the web application to the Linux system itself.

The privilege-escalation part was especially useful because it showed me how privileged binaries can become a security problem when they are incorrectly configured.

## Conclusion

The DC:1 lab gave me practical experience with a complete attack chain.

I started by scanning the machine with Nmap and found ports 22, 80, and 111. I focused on the HTTP service because it was running Drupal. I then used Metasploit to gain initial access.

After getting access, I followed the clues from the flags. The first flag led me to the Drupal configuration file, where I found database information. I then investigated the database and its tables and found information related to the administrator account. I changed the administrator password to `admin` and continued the investigation.

The later flag told me to check for misconfigurations. I investigated privileged binaries and used GTFOBins as a reference for privilege escalation. Finally, I was able to obtain root access.

Overall, this lab helped me understand how reconnaissance, exploitation, application enumeration, database investigation, and Linux privilege escalation can be connected together during a penetration test.
