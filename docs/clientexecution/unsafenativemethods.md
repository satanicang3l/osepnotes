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

5. After we identify that it is coming from System.dll, we can use the following to search for the required API (in this case `GetModuleHandle`).

```powershell
$systemdll = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') })
$unsafeObj = $systemdll.GetType('Microsoft.Win32.UnsafeNativeMethods')
$GetModuleHandle = $unsafeObj.GetMethod('GetModuleHandle')
```

6. Then we can use the following search for the DLL base address (for example user32.dll):

```powershell
$GetModuleHandle.Invoke($null, @("user32.dll"))
```

7. If you try to do a `$GetProcAddress = $unsafeObj.GetMethod('GetProcAddress')` you will get an error "Ambiguous match found.". Instead we need to do this for multiple occurence:

```powershell
$user32 = $GetModuleHandle.Invoke($null, @("user32.dll"))
$tmp=@()
$unsafeObj.GetMethods() | ForEach-Object {If($_.Name -eq "GetProcAddress") {$tmp+=$_}}
$GetProcAddress = $tmp[0]
$GetProcAddress.Invoke($null, @($user32, "MessageBoxA"))
```

8. Finally to reuse this multiple times, wrap them in function:

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
```

9. To use the function above, do this:

```powershell
LookupFunc "user32.dll" "MessageBoxA"
```
