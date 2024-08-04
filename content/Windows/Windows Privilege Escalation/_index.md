---
title: Checklist - Local Windows Privilege Escalation
aliases:
---
# Checklist - Local Windows Privilege Escalation

##  Kernel Exploits
- [ ] [[Kernel Exploits]] - `python wes.py systeminfo.txt -i 'Elevation of Privilege' --exploits-only | le`

***
## Services
 -  `.\winPEASany.exe quiet servicesinfo`
   
* [ ] [[Insecure Service Properties|Insecure Service Properties]] 
* [ ] [[Unquoted Service Paths]]
* [ ] [[Services registry modify permissions]]
* [ ] [[Services binaries weak permissions]]
* [ ] [[PATH DLL Hijacking|PATH DLL Hijacking]]
***
##  Registry

- [ ] [[Run at startup]] - `.\winPEASany.exe quiet applicationsinfo`
- [ ] [[AlwaysInstallElevated]] - `.\winPEASany.exe quiet windowscreds`
***
## Passwords

- [ ] [[Searching the Registry for Passwords]] - `.\winPEASany.exe quiet filesinfo userinfo`