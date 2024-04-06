---
title: Unsafe Native Methods
layout: default
parent: Client Side Execution
---

1. To avoid writing to disk, we need to perform dynamic lookup of function address instead of `Add-Type` and `DllImport`. 
2. This can be achieved by using Win32 API `GetModuleHandle` and `GetProcAddress`.
3. Since we cannot use them directly to avoid writing to disk, need to find assembly that we can use to invoke these API.

```powershell
$Assemblies = [AppDomain]::CurrentDomain.GetAssemblies()
$Assemblies |
  ForEach-Object {
    $_.GetTypes()|
      ForEach-Object {
        $_ | Get-Member -Static| Where-Object {
          $_.TypeName.Contains('Unsafe')
        }
    } 2> $null
  }
```

4. Next, perform a filter and search for the location that these APIs are located by adding `$_.Location`. Based on the results, they are `static` and the typename is `Microsoft.Win32.UnsafeNativeMethods` (Use `Contains` instead of `Equals` to avoid error).

```powershell
$Assemblies = [AppDomain]::CurrentDomain.GetAssemblies()
$Assemblies |
  ForEach-Object {
    $_.Location
    $_.GetTypes()|
      ForEach-Object {
        $_ | Get-Member -Static| Where-Object {
          $_.TypeName.Contains('Microsoft.Win32.UnsafeNativeMethods')
        }
    } 2> $null
  }
```

5. Finally after we finally identify that it is coming from System.dll, we can use the following to search for the required API (in this case `GetModuleHandle`).

```powershell
$systemdll = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object {
$_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') })
$unsafeObj = $systemdll.GetType('Microsoft.Win32.UnsafeNativeMethods')
$GetModuleHandle = $unsafeObj.GetMethod('GetModuleHandle')
```

6. Finally we can use the following search for the DLL address (for example user32.dll):

```powershell
$GetModuleHandle.Invoke($null, @("user32.dll"))
```
