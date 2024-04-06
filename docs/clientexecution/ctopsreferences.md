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

After that need to use `Add-Type` to use it from Powershell. 

# Example 1:
An example to use `MessageBox`:

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

[User32]::MessageBox(0, "This is an alert", "MyBox", 0)
```

Note:

 * `@` is for Here-String, which allows you to enter multiple lines of code like a line
 * The last line is to invoke the method MessageBox and passing the arguments to it

# Example 2:
Another example is on `GetUserName`:

```powershell
$random321 = @"
using System;
using System.Runtime.InteropServices;
using System.Text;
public class any321 {
  [DllImport("Advapi32.dll", SetLastError = true)]
  public static extern bool GetUserName(StringBuilder lpBuffer, ref uint nSize);
}
"@
Add-Type $random321

$usernameBuilder = New-Object System.Text.StringBuilder 256
$bufferSize = $usernameBuilder.Capacity
[any321]::GetUserName($usernameBuilder, [ref]$bufferSize)
$username = $usernameBuilder.ToString().TrimEnd([char]::NULL)
$username
```

Note:

* We add `using System.Text` here because if not, PowerShell will complaint about `StringBuilder` could not be found (It is in `System.Text`)
* `$usernameBuilder` is a reference to the `StringBuilder` object, and passing it to `GetUserName` allows the function to directly manipulate the data stored in that object.

[PInvoke Alternative 1]: https://www.p-invoke.net/
[PInvoke Alternative 2]: https://www.pinvoke.dev/
