---
title: Locating Signature (Less Effective)
layout: default
parent: Antivirus Evasion
nav_order: 2
---

The theory behind is to split the binary into multiple pieces and perform scan on small pieces until the exact byte flagged is found.
However not very effective because not all antivirus uses the same signature to detect.

1. Import the script:
`Import-Module .\Find-AVSignature.ps1`

2. Specify a start byte of 0, end byte to max, with a large enough interval (eg 10000 for 70000 bytes), and force a output path so we can inspect later.
`Find-AVSignature -StartByte 0 -EndByte max -Interval 10000 -Path C:\Tools\met.exe -OutPath C:\Tools\avtest1 -Verbose -Force`

3. Can then use your AV to scan for every file (ClamAV in this example):
`cd 'C:\Program Files\ClamAV\'`\
`.\clamscan.exe C:\Tools\avtest1`

4. Once you know which one triggered the signature detection, can split the part into smaller part (for example if detected in 10000 to 20000):
`Find-AVSignature -StartByte 10000 -EndByte 20000 -Interval 1000 -Path C:\Tools\met.exe -OutPath C:\Tools\avtest2 -Verbose -Force`

5. Then run the scan again:
`cd 'C:\Program Files\ClamAV\'`\
`.\clamscan.exe C:\Tools\avtest2`

6. If it happens again for example in offset 18000 to 19000, do the split again:
`Find-AVSignature -StartByte 18000 -EndByte 19000 -Interval 100 -Path C:\Tools\met.exe -OutPath C:\Tools\avtest3 -Verbose -Force`

7. Do it until you can pinpoint which one byte is triggering the signature. Then patch it with powershell. If there are multiple different signaure, do it one by one. For example to patch out offset 18867 to 0.
```
$bytes  = [System.IO.File]::ReadAllBytes("C:\Tools\met.exe")
$bytes[18867] = 0
[System.IO.File]::WriteAllBytes("C:\Tools\met_mod.exe", $bytes)
```
