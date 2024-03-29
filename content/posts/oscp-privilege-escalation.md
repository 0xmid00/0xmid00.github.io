+++
draft = true
date = 2024-03-29T23:40:34+01:00
title = "OSCP Course Privilege Escalation Notes"
description = "Notes on privilege escalation techniques covered in the OSCP course."
slug = ""
authors = []
tags = ["OSCP", "privilege-escalation", "enumeration", "Windows", "Linux"]
categories = ["Cybersecurity", "Penetration Testing"]
externalLink = ""
series = []
+++


# Information Gathering:

## Manual Enumeration (User Information):

### Windows:
- `whoami`
- `net user` (list all users)
- `net user [username]` (get user info)

### Linux:
- `whoami`
- `cat /etc/passwd`
- `id`

## Hostname Enumeration:

### Common Commands:
- Windows: `hostname` or `systeminfo`
- Linux: `uname -a` or `cat /etc/issue`

## List Services and Processes:

- Windows: `tasklist /SVC`
- Linux: `ps aux`

## Network Enumeration:

### IP Address:
- Windows: `ipconfig`
- Linux: `ifconfig`

### Network Routes:
- Windows: `route print`
- Linux: `arp -a`

### Network Connections:
- Windows: `netstat -ano`
- Linux: `ss -anp`

## Firewall Enumeration:

### Windows:
- `netsh advfirewall show currentprofile`
- `netsh advfirewall firewall show rule name=all`

## Scheduled Tasks Enumeration:

### Linux:
- `ls -lah /etc/cron*`

### Windows:
- `schtasks /query /fo LIST /v`

## Enumeration of Applications:

### Linux:
- `dpkg -l` or `dpkg -l | grep ^ii` (installed packages)
- `lsmod` (loaded kernel modules)

### Windows:
- `wmic product get name, version, vendor` (installed software)
- `wmic qfe get Caption, Description, HotFixID, InstalledOn` (installed updates)

## Enumeration of Readable/Writable Files and Directories:

### Windows:
- `Get-ChildItem "C:\Program Files" -Recurse | Get-Acl | Where-Object { $_.AccessToString -match "Everyone\sAllow\sModify" }`

### Linux:
- `find / -writable -type d 2>/dev/null`

## Enumeration of Unmounted Disks:

- Windows: `mountvol`
- Linux: `mount`

## Enumeration of Device Drivers and Kernel Modules:

- Windows: `driverquery.exe /v /fo csv | ConvertFrom-CSV | Select-Object 'Display Name', 'Start Mode', 'Path'`
- Linux: `lsmod`, `modinfo <module>`

## Get Version for a Specific Module (Linux):

- `modinfo <module>`

# Automated Enumeration

- windows-privesc-check
- Watson
- Sherlock
- PowerSploit/Privesc/PowerUp
- Windows-Exploit-Suggester
- JAWS
- WinPEAS .exe and .bat
- linPEAS & LinEnum (for Linux)

# Windows Privilege Escalation

## Insecure File Permissions:

powershell -ep bypass
Import-Module .\powerup.ps1
invoke-AllChecks
icacls.exe '<path service>'


## Escalation via Unquoted Service Paths.

# Linux Privilege Escalation

## Linux Permissions:

To use setuid permission, use:

./<suid_file_here,sh> -p


## Find all commands that you can execute as root with sudo without the password:


sudo -l
sudo vim (no password needed)


## Automated Tools:
- LinPEAS - Linux Privilege Escalation Awesome Script
  [Link to LinPEAS GitHub](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS)

