---
title: "MetaCTF"
date: 2024-09-16T01:20:25+09:00
draft: false
---

## What's a CTF? (229 solves)

問題文に書いてある flag を入力します

## My Head Hurts (226)

> Binary is a numbering system where that uses only two characters: 1s and 0s. At the lowest level of logic, this is the language spoken by computers (in the form of high and low electrical signals).I was going to give you the flag, but it got converted to binary. Can you help me bring it back?
> 01001101 01100101 01110100 01100001 01000011 01010100 01000110 01111011 01101100 01100001 01101110 01100111 01110101 01100001 01100111 01100101 01011111 01101111 01100110 01011111 01101110 01100101 01110010 01100100 01110011 01111101

cyberchef で OK

## Portal Under Inspection (223)

> You need to urgently log into the facility admin portal to reactivate the security system.Can you figure out a way to login?

通信を見るとパスワードを比較する js ファイルが見える．
この比較部分がモロに出ているので flag として出す

## Informing the Soldiers (212)

> Julius Caesar is wanting to relay some vital information to his front line troops. He sent this message: ZrgnPGS{napvrag_pvcuref_sgj).

Caesar暗号で OK

## All your file are mine (184)

>During a breach investigation, you find an odd file: C:\Windows\Temp\abc.xyz that was created while a threat actor was accessing the system.That file type doesn't seem right...can you figure out what kind of file this is and if there's anything interesting in it?

```file``` コマンドで zip だと分かるので ```unzip```

## Captured in Transit (180)

> We sniffed some network traffic from a suspect computer, can you extract the flag from it?

``` text
GET /flag.txt HTTP/1.1
Host: 172.17.0.3
User-Agent: curl/7.81.0
Accept: */*

HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.10.6
Date: Fri, 05 May 2023 13:41:26 GMT
Content-type: text/plain
Content-Length: 40
Last-Modified: Fri, 05 May 2023 13:38:56 GMT

MetaCTF{0bjectiv3ly_subj3ctive_p4ckets}
```

## Unknown Architect (152)

> How strange. This binary won't run on any of my computers. What kind of system was it built for?
> The answer is not in flag format; rather, it's just the name of the architecture this program is meant to run on. You don't need to run the program. Note that you only have 5 attempts on this challenge.

```file``` コマンド

## Mmm, Birthday Cake(142)

> Inside this ZIP file is a delicious birthday cake. Would you like a slice? The password is, of course, the six digits of my birthday (MMDDYY, no dashes or anything). Since you're such a good friend, you remember it, right?

総当たり

fcrackzip でもいいがせっかくなのでコードを書く

``` python
import zipfile
zf = zipfile.ZipFile("birthdaycake.zip")


for month in range(13):
  for day in range(32):
    for year in range(100):
      tmp_password = str(month).zfill(2) + str(day).zfill(2) + str(year).zfill(2)
      #print(tmp_password)
      try:
        zf.extract('flag.txt', pwd = tmp_password.encode('utf-8'))      
        password = 'Password found: %s' % tmp_password
        print("OK")
        print(password)
        break
      except:
        pass
        
#print(tmp_password)
```

## Server Sleuthing(139)

> You've been tasked with testing the security of this Electricity Substation management portal. Can you figure out what framework/programming language and version are used to run this app?

通信を見るとレスポンスに書いてある

## Unintended Reveal (135)

> Your coworker posted this photo of their room at a ranch they are currently vacationing at. Can you figure out the name of the ranch?

```exiftool``` で緯度経度が書いてあるので google map で見る

## Duke of the Kingdom (131)

> I love this little guy so much. Check out the program I made for them

まずはデコンパイル ```java -jar cfr-0.152.jar Duke.class```

重要なところだけ抜き出すと以下

```Java
System.out.println(string);
        System.out.println("Duke is waving at you! Say Hi to Duke.");
        Scanner scanner = new Scanner(System.in);
        System.out.print("> ");
        String string2 = scanner.nextLine().toLowerCase();
        if (string2.contains("hi duke")) {
            System.out.println("Aw, Duke liked that! Hi!");
            byte[] byArray = string2.getBytes();
            int[] nArray = new int[]{986, 995, 338, 950, 1103, 1013, 959, 338, 986, 959, 1076, 959, 401, 1085, 338, 923, 338, 968, 1022, 923, 977, 338, 968, 1049, 1076, 338, 1139, 1049, 1103, 338, 1031, 959, 1094, 923, 941, 1094, 968, 1157, 950, 1103, 1013, 509, 905, 923, 1094, 905, 1094, 986, 509, 905, 1094, 482, 1058, 905, 482, 968, 905, 1094, 986, 509, 995, 1076, 905, 941, 1022, 518, 1085, 1085, 1175};
            if (byArray.length != nArray.length) {
                return;
            }
            for (int i = 0; i < byArray.length; ++i) {
                if (byArray[i] * 9 + 50 == nArray[i]) continue;
                return;
            }
            System.out.println("And Duke LOVES the flag! Go submit it!");
        } else if (string2.contains("hi")) {
            System.out.println("Hi!");
        } else {
            System.out.println(":(");
        }
```

ある一定の入力で flag を表示してくれるらしい．
```if (byArray[i] * 9 + 50 == nArray[i])``` ということなので，これを逆算すればいい．

```python
words = [986, 995, 338, 950, 1103, 1013, 959, 338, 986, 959, 1076, 959, 401, 1085, 338, 923, 338, 968, 1022, 923, 977, 338, 968, 1049, 1076, 338, 1139, 1049, 1103, 338, 1031, 959, 1094, 923, 941, 1094, 968, 1157, 950, 1103, 1013, 509, 905, 923, 1094, 905, 1094, 986, 509, 905, 1094, 482, 1058, 905, 482, 968, 905, 1094, 986, 509, 995, 1076, 905, 941, 1022, 518, 1085, 1085, 1175]

for i in range(len(words)):
    print(chr((words[i] - 50) // 9), end='')

```

## Subsea Searching (80)

急に solves 落ちたな

> A Google cable lands here. What town does the other end of the cable land in?
> This answer is not in flag format, you should just submit the name of the town. You only have 10 attempts for this one.

google の光ケーブルの陸揚げ地の写真がもらえる．
マンホールを見ると ```VBDA-COMM``` と書いてある．
色々調べてみると Virginia Beach Development Authority のマンホールっぽい？
光ファイバーの[敷設状況](https://www.yesvirginiabeach.com/key-industries/digital-port)を調べてみると Virginia Beach Development Authority からいくつかの街に飛んでいる．
これをいくつか選ぶと Saint-Hilaire-de-Riez が答えだった．

## Escalator (69)

> Help! We need someone to climb from the bottom to the top. Can you find the way?Escalate privileges to the root user and read the flag file in /flag.txt

権限昇格すればいい，ということで，ログインしてまず SUID を見つける

```shell
find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
```

```find``` が管理者権限で使えるらしい．

``` shell
find / -exec cat /flag.txt \; -quit
```

## Head Cannon (67)

> To find the flag, find what.three.words for the location of where the person taking this picture is standing. The flag is in the format of X.X.X where the Xs are the words needed for the location.

噂には聞いていたが what.three.words を初めて使った．

google photo で調べると bull street とか Savannah がヒットする．
この情報をもとにストリートビューで歩き回ってると同じような大砲を見つけるので，what.three.words の方で一致しそうなマスを総当たりする．

## Keypad(66)

> Welcome to the most critical and clandestine operation of your hacking career. You’ve just intercepted a highly sensitive communication regarding the security of a nuclear power plant. The facility is on high alert due to a recent security breach, and you’ve been tasked with infiltrating the most guarded system—the reactor’s keypad.
> Successfully complete these security checks to generate the decryption key that will unlock the final layer of protection. What you uncover next will take you deeper into the heart of the facility, where the stakes rise even higher. The reactors are not just a source of power; they are the gateway to unlocking the plant’s most hidden secrets.

デコンパイル結果は以下．

``` C  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  puts("Welcome to the Nuclear Power Plant Keypad Challenge!");
  puts("Security Check 1: Access Code Verification");
  local_478 = 0;
  local_470 = 0;
  local_468 = 0;
  local_460 = 0;
  local_458 = 0;
  local_450 = 0;
  fgets((char *)&local_478,0x30,stdin);
  sVar2 = strcspn((char *)&local_478,"\n");
  *(undefined *)((long)&local_478 + sVar2) = 0;
  local_4d8 = "power";
  local_4a8 = 0x4131d09130e00;
  xor_encrypt(&local_478,"power");
  iVar1 = strcmp((char *)&local_4a8,(char *)&local_478);
  if (iVar1 == 0) {
    puts("Access Code Verified!");
    puts("Security Check 2: Numeric Keypad Entry");
    __isoc99_scanf("%d %d %d %d",&local_504,&local_500,&local_4fc,&local_4f8);
    local_4c8[0] = 9;
    local_4c8[1] = 7;
    local_4c8[2] = 1;
    local_4c8[3] = 3;
    local_4b8[0] = local_504;
    local_4b8[1] = local_500;
    local_4b8[2] = local_4fc;
    local_4b8[3] = local_4f8;
    local_4d0 = 4;
    local_4f0 = 4;
    add_encrypt(local_4b8,4,4);
    for (local_4e0 = 0; local_4e0 < local_4d0; local_4e0 = local_4e0 + 1) {
      if (local_4c8[local_4e0] != local_4b8[local_4e0]) {
        puts("Access Denied: Numeric Entry Incorrect.");
        uVar3 = 1;
        goto LAB_00101a5d;
      }
    }
    do {
      local_4ec = getchar();
      if (local_4ec == -1) break;
    } while (local_4ec != 10);
    puts("Numeric Keypad Entry Verified!");
    puts("Security Check 3: Reversed Passphrase");
    local_448 = 0;
    local_440 = 0;
    local_438 = 0;
    local_430 = 0;
    local_428 = 0;
    local_420 = 0;
    fgets((char *)&local_448,0x30,stdin);
    sVar2 = strcspn((char *)&local_448,"\n");
    *(undefined *)((long)&local_448 + sVar2) = 0;
    local_4a0 = 0x72657665727365;
    reverse_encrypt(&local_448);
    iVar1 = strcmp((char *)&local_4a0,(char *)&local_448);
    if (iVar1 == 0) {
      puts("Passphrase Verified!");
      puts("Security Check 4: Lookup Table Validation");
      __isoc99_scanf(&DAT_001021c2,&local_4f4);
      local_4e8 = 9;
      local_4e4 = lookup_encrypt(local_4f4);
      if (local_4e4 == local_4e8) {
        puts("Lookup Table Validated!");
        snprintf(local_418,0x400,"%s%d%d%d%d%s%d","padlock",(ulong)local_504,(ulong)local_500,
                 (ulong)local_4fc,(ulong)local_4f8,&local_448,(ulong)local_4f4);
        local_498 = 0x65102d202f303222;
        local_490 = 0x343a19100a555352;
        local_488 = 0x120801120301021c;
        local_480 = 0;
        xor_encrypt(&local_498,local_418);
        printf("Flag: %s\n",&local_498);
        uVar3 = 0;
      }
      else {
        puts("Access Denied: Lookup Table Mismatch.");
        uVar3 = 1;
      }
    }
    else {
      puts("Access Denied: Incorrect Passphrase.");
      uVar3 = 1;
    }
  }
  else {
    puts("Access Denied: Incorrect Code.");
    uVar3 = 1;
  }
```

### Security Check 1

```local_4a8 = 0x4131d09130e00;``` というデータと比較するが，最初が null bytes なのでそのまま Enter する

### Security Check 2

```*(int *)(local_10 * 4 + param_1) = (*(int *)(param_1 + local_10 * 4) + param_3) % 10;``` を満たす入力を調べる．
でもこれ複数パターンあるよな～とか思っているが，後々ハマる

### Security Check 3

```0x72657665727365``` というデータに対して，逆順にした入力を正しいかどうかを調べている．
なので reverse

### Security Check 4

lookup table が見えるので，出力が 9 になる入力を知れればいい．

というわけだが，最終的に flag をデコードしてくれる部分が，これまでの入力も鍵に使っている．
なので，Security Check2 に対して複数あり得る入力の全てが鍵になり得るが，このうち正しいものは 1 つしかない．
RSTCON{} というようなフォーマットに沿う出力が得られれば OK ということで総当たりするか～とか思っていたら最小の入力で OK だった

## Griddown(59)

> One of our substations went down in a cyberattack. We captured this network trace from the server.

wireshark で見ると，MMS というプロトコルが大量に見える．キャリアメールの通信方式らしいがよく分かっていない．
[ここ](https://ask.wireshark.org/question/18251/mms-protocol-is-captured/)の情報をもとに wireshark をカスタマイズすると MMS が見えるようになる．
wireshark が デコードしてくれた MMS の通信を見ると，FileName なるデータが見える．
これを連結させると flag になる．

## Semper Fi(54)

> To find the flag, find what.three.words for the location of the image. The flag is in the format of X.X.X where the Xs are the words needed for the location

また OSINT か．．．と思っていたが全然前の OSINT で調べた場所に近かったので一瞬

## Play It On the Radio(25)

> We captured some radio waves and put them in this file for you. Plucked them right out of the air, onto the disk. Isn't software awesome?

iq ファイルが渡される．
gqrx で解析するが，サンプリングレートだけ注意する．
今回は 18e4 にした．

gqrx の方で再生すると音声になるので，flag にする．
関係ないけど Fourier を 48 と聞き間違えており全然フラグが通らず泣いていた

ここまでで時間切れ．
問題量に圧倒されてしまった感があるが面白い問題も多かった．
公式 writeup も問題リポジトリもないのは寂しいし復習できないのがつらい
