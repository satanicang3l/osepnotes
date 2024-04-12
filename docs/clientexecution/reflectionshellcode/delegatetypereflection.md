---
title: DelegateType Reflection
layout: default
parent: Reflection Shellcode
grand_parent: Client Side Execution
nav_order: 2
---

1. To invoke the Win32 API, in CSharp it is done using `GetDelegateForFunctionPointer` method. However in PowerShell this is not available.
2. `Add-Type` can help you to do this, but since we want to do it in memory we need to manually the assembly and populate it with content.
3. Create a new assembly object through the `AssemblyName` class by creating a new variable `$MyAssembly` and set it to the instantiated assembly object with the name "ReflectedDelegate".

   ```powershell
   $MyAssembly = New-Object System.Reflection.AssemblyName('ReflectedDelegate')
   ```
    
4. Configure access mode(permission) to be executable and not saved to disk. Use `DefineDynamicAssembly` method, supplying name in previous step, and the Run access mode.

    ```powershell
    $Domain = [AppDomain]::CurrentDomain
    $MyAssemblyBuilder = $Domain.DefineDynamicAssembly($MyAssembly, [System.Reflection.Emit.AssemblyBuilderAccess]::Run)
    ```

5. Create the main block of the assembly, which is Module. Use `DefineDynamicModule` to and pass the following to it: a module name and false, to not include symbol information.

    ```powershell
    $MyModuleBuilder = $MyAssemblyBuilder.DefineDynamicModule('InMemoryModule', $false)
    ```

6. Create a custom type to substitute the delegate type. Use the `DefineType` method and pass: custom name, combined list of attributes for the type, type it builds on top of.

    ```powershell
    $MyTypeBuilder = $MyModuleBuilder.DefineType('MyDelegateType', 'Class, Public, Sealed, AnsiClass, AutoClass', [System.MulticastDelegate])
    ```

7. Put the function prototype into inside the type and let it become our custom delegate type. **(The last argument depends on the API you would like to call! The example below is for `MessageBox` which got IntPtr, String, String, int)**

    ```powershell
    $MyConstructorBuilder = $MyTypeBuilder.DefineConstructor('RTSpecialName, HideBySig, Public', [System.Reflection.CallingConventions]::Standard, @([IntPtr], [String], [String], [int]))
    ```

8. Setting implementation flag for the constructor by using `SetImplementationFlags` method.

    ```powershell
    $MyConstructorBuilder.SetImplementationFlags('Runtime, Managed')
    ```

9. Define an `Invoke` method to tell .NET framework the delegate type to be used in a function, by using `DefineMethod`. **(The third argument is the return type of the API you want to call, and forth is argument type for the API)**

    ```powershell
    $MyMethodBuilder = $MyTypeBuilder.DefineMethod('Invoke', 'Public, HideBySig, NewSlot, Virtual', [int], @([IntPtr], [String], [String], [int]))
    ```

10. Set the implementation flag, this time for the `Invoke` method.

    ```powershell
    $MyMethodBuilder.SetImplementationFlags('Runtime, Managed')
    ```

11. Instantiate the delegate type by calling the constructor through `CreateType` method.

    ```powershell
    $MyDelegateType = $MyTypeBuilder.CreateType()
    ```
