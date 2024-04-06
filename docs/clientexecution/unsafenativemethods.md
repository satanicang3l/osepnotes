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
