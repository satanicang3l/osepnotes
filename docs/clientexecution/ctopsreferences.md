---
title: C to Powershell References
layout: default
parent: Client Side Execution
---

# References:
[PInvoke Alternative 1]
[PInvoke Alternative 2]

First need to convert it to CSharp, a quick way is use PInvoke. Every use of PInvoke in CSharp needs to imported with the following:

```csharp
using System;
using System.Runtime.InteropServices;
```

After that need to use Add-Type to use it from Powershell. An example to use `MessageBox`: (`@` is for Here-String, which allows you to enter multiple lins of code like a line)

```powershell
$User32 = @"
using System;
using System.Runtime.InteropServices;
public class User32 {
  [DllImport("user32.dll", CharSet=CharSet.Auto)]
  public static extern int MessageBox(IntPtr hWnd, String text, String caption, int options);
}
"@
Add-Type $User32
```

[PInvoke Alternative 1]: https://www.p-invoke.net/
[PInvoke Alternative 2]: https://www.pinvoke.dev/
