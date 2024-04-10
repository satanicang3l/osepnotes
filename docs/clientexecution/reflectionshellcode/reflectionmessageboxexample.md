---
title: Reflection MessageBox Example
layout: default
parent: Reflection Shellcode
grand_parent: Client Side Execution
nav_order: 3
---

This first perform a lookup of address (refer to [Unsafe Native Methods]), then create the delegate type (refer to [DelegateType Reflection]) and call `GetDelegateForFunctionPointer` to pass the address and the `DelegateType` to call `MessageBox`.

```powershell
function LookupFunc {
  Param ($moduleName, $functionName)

  $assem = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')
  $tmp=@()
  $assem.GetMethods() | ForEach-Object {If($_.Name -eq "GetProcAddress") {$tmp+=$_}}
  return $tmp[0].Invoke($null, @(($assem.GetMethod('GetModuleHandle')).Invoke($null, @($moduleName)), $functionName))
}

$MessageBoxA = LookupFunc user32.dll MessageBoxA
$MyAssembly = New-Object System.Reflection.AssemblyName('ReflectedDelegate')
$Domain = [AppDomain]::CurrentDomain
$MyAssemblyBuilder = $Domain.DefineDynamicAssembly($MyAssembly, [System.Reflection.Emit.AssemblyBuilderAccess]::Run)
$MyModuleBuilder = $MyAssemblyBuilder.DefineDynamicModule('InMemoryModule', $false)
$MyTypeBuilder = $MyModuleBuilder.DefineType('MyDelegateType', 'Class, Public, Sealed, AnsiClass, AutoClass', [System.MulticastDelegate])

$MyConstructorBuilder = $MyTypeBuilder.DefineConstructor('RTSpecialName, HideBySig, Public', [System.Reflection.CallingConventions]::Standard, @([IntPtr], [String], [String], [int]))
$MyConstructorBuilder.SetImplementationFlags('Runtime, Managed')
$MyMethodBuilder = $MyTypeBuilder.DefineMethod('Invoke', 'Public, HideBySig, NewSlot, Virtual', [int], @([IntPtr], [String], [String], [int]))
$MyMethodBuilder.SetImplementationFlags('Runtime, Managed')
$MyDelegateType = $MyTypeBuilder.CreateType()

$MyFunction = [System.Runtime.InteropServices.Marshal]::
GetDelegateForFunctionPointer($MessageBoxA, $MyDelegateType)
$MyFunction.Invoke([IntPtr]::Zero,"Hello World","This is My MessageBox",0)
```

[Unsafe Native Methods]: https://satanicang3l.github.io/osepnotes/docs/clientexecution/reflectionshellcode/unsafenativemethods.html
[DelegateType Reflection]: https://satanicang3l.github.io/osepnotes/docs/clientexecution/reflectionshellcode/delegatetypereflection.html
