---
title: PowerShell Shellcode
layout: default
parent: Client Side Execution
---

Similar to CSharp, the theory is to:
1. Allocate memory with `VirtualAlloc(0, sizeOfShellcodeArray, &H3000, &H40)`
2. Copy shellcode using `System.Runtime.InteropServices.Marshal`
3. Execute with `CreateThread(0, 0, startAddress, 0, 0, 0)`. startAddress here is the return value of `VirtualAlloc()` in step 1.
