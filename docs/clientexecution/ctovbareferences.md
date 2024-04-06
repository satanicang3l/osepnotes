---
title: C to VBA References
layout: default
parent: Client Side Execution
---

# References:
* LPSTR -> ByVal, As String
* LPDWORD -> By Ref, As Long

# Example:

First look at the corresponding Microsoft site to find out the DLL it resides in, for example to use GetUserName API:

[MSDN GetUserNameA]

From website above it is possible to know that the API resides in Advapi32.dll, and the C++ code is:

```cpp
BOOL GetUserNameA(
  [out]     LPSTR   lpBuffer,
  [in, out] LPDWORD pcbBuffer
);
```

Then proceed to declare and import the API name and DLL location (**Add PtrSafe before the Function in first line for 64-bit**):

```vb
Private Declare Function GetUserName Lib "advapi32.dll" Alias "GetUserNameA" (ByVal lpBuffer As String, ByRef nSize As Long) As Long
Function MyMacro()
  Dim res As Long
  Dim MyBuff As String * 256
  Dim MySize As Long
  Dim strlen As Long
  MySize = 256
  res = GetUserName(MyBuff, MySize)
  strlen = InStr(1, MyBuff, vbNullChar) - 1
  MsgBox Left$(MyBuff, strlen)
End Function
```

In C, string is terminated by null byte, so we can use InStr to find out which character is the null byte character, then get the length of character. The syntax is:

```
Instr(START FROM, STRING TO SEARCH, WHAT CHARACTER TO SEARCH) - 1
```

The code:

* Declares variables res (result of the function call), MyBuff (string buffer to store the username), MySize (size of the buffer), and strlen (length of the username).
* Sets the size of the buffer (MySize) to 256.
* Calls the GetUserName function passing the buffer MyBuff and its size MySize as parameters to retrieve the username. The result of the function call is stored in the variable res.
* Calculates the length of the retrieved username by finding the position of the null character (vbNullChar) in the buffer (MyBuff) and subtracting 1.
* Displays a message box showing the username retrieved from the buffer, using Left$ to extract the substring containing the username based on its length (strlen).



[MSDN GetUserNameA]: https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-getusernamea
