---
title: "ACSC 2024 upsolve"
date: 2024-04-02T22:43:36+09:00
draft: true
---

前回は ACSC 2024 の writeup でしたが，hardware 以外の問題を解かなかったのでここで消化します．
分からないものは適宜 writeup を見て色々やることにします．
初心者なので試行錯誤の跡が残っています

### Web: Login!

username と password を入力するシンプルなシステムが与えられます．
ソースコードも渡されました．

```javascript
const express = require('express');
const crypto = require('crypto');
const FLAG = process.env.FLAG || 'flag{this_is_a_fake_flag}';

const app = express();
app.use(express.urlencoded({ extended: true }));

const USER_DB = {
    user: {
        username: 'user', 
        password: crypto.randomBytes(32).toString('hex')
    },
    guest: {
        username: 'guest',
        password: 'guest'
    }
};

app.get('/', (req, res) => {
    res.send(`
    <html><head><title>Login</title><link rel="stylesheet" href="https://cdn.simplecss.org/simple.min.css"></head>
    <body>
    <section>
    <h1>Login</h1>
    <form action="/login" method="post">
    <input type="text" name="username" placeholder="Username" length="6" required>
    <input type="password" name="password" placeholder="Password" required>
    <button type="submit">Login</button>
    </form>
    </section>
    </body></html>
    `);
});

app.post('/login', (req, res) => {
    const { username, password } = req.body;
    if (username.length > 6) return res.send('Username is too long');

    const user = USER_DB[username];
    if (user && user.password == password) {
        if (username === 'guest') {
            res.send('Welcome, guest. You do not have permission to view the flag');
        } else {
            res.send(`Welcome, ${username}. Here is your flag: ${FLAG}`);
        }
    } else {
        res.send('Invalid username or password');
    }
});

app.listen(5000, () => {
    console.log('Server is running on port 5000');
});
```

guest/guest でログインできるが，flag はもらえない．
guest 以外でログインすると flag が返ってくる．
別のユーザである user でログインすると(というか guest 以外で)フラグがもらえそうだけど，user のパスワードはランダム化されて分かりそうにない．
データベースの中身を触れるんだろうかと思ったけどさすがにできないように見える....
わざわざ guest 以外という書き方をしているのでパスワードの入力でバイパスするか，パスワード guest だけど username が guest でないログインみたいな方法がありそうと思って色々調べるけどわからず

```username === 'guest'``` とあるところを見て，```===``` って ```==``` より厳しい制約だった気がするので調べてみる．
6文字以下という制約があるのである程度絞られると思って，エンコードを試してみるがうまくいかない(Guest, guest, gues\74 ...).
型が違わないと ```===``` を乗り越えられない

ここで，以下の挙動を見つける（何かの writeup で見たけどメモしていなかった）

```shell
> ['guest'] == 'guest'
< true

> ['guest'] === 'guest'
< false
```

なので，```username[] = guest``` になればいい

- username.length -> 1
- user = USER_DB[['guest']] -> true
- username === 'guest' -> false

```username[]=guest&password=guest```

189 solved もあるのに死ぬほど時間かかってしまった...
こういう怪しい実装に対する嗅覚が無い

### Web: Too Faulty

username と password を入れるフォームがある．
二段階認証か，デバイス固有の識別による認証がある．
THM の手癖で admin/admin ってやったらログインできたが(!?)，二段階認証があってログインできない．

ソースコードが公開されていないので developer tool からログイン時のソースコードを見てみる．

```javascript
document
  .getElementById("loginForm")
  .addEventListener("submit", function (event) {
    event.preventDefault();

    const username = document.getElementById("username").value;
    const password = document.getElementById("password").value;
    const browser = bowser.getParser(window.navigator.userAgent);
    const browserObject = browser.getBrowser();
    const versionReg = browserObject.version.match(/^(\d+\.\d+)/);
    const version = versionReg ? versionReg[1] : "unknown";
    const deviceId = CryptoJS.HmacSHA1(
      `${browserObject.name} ${version}`,
      "2846547907"
    );
```

ここまででデバイスIDを生成する．
「device identification solution」っていう部分だと思う

```javascript
    fetch("/login", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "X-Device-Id": deviceId,
      },
      body: JSON.stringify({ username, password }),
    })
      .then((response) => {
        if (response.redirected) {
          window.location.href = response.url;
        } else if (response.ok) {
          response.json().then((data) => {
            if (data.redirect) {
              window.location.href = data.redirect;
            } else {
              window.location.href = "/";
            }
          });
        } else {
          throw new Error("Login failed");
        }
      })
      .catch((error) => {
        console.error("Error:", error);
      });
  });

function redirectToRegister() {
  window.location.href = "/register";
}
```

ここまでで ```deviceId``` を全探索するのか，と思ったがさすがに時間がかかると思うので writeup を見る

どうやら，ユーザ単位での認証ではなく，セッション単位でデバイスが認証されているらしい．[Too Faulty](https://nanimokangaeteinai.hateblo.jp/entry/2024/04/01/184626#Web-150-Too-Faulty-67-solves)
それさえ分かってしまうとあとは簡単で，自分のアカウントでログインした際の cookie を admin でログインするときに書き換えてしまえばいい

```ACSC{T0o_F4ulty_T0_B3_4dm1n}```

### Web: Buggy Bounty

id, URL, report を送信すると rewarded xx$ という風なメッセージが返ってくる．
flag の場所を知りたいので ```docker-compose.yml``` を見る

```docker
version: "3.8"

services:
  bugbounty:
    build:
      context: .
      dockerfile: ./bugbounty/Dockerfile
    image: bugbounty
    ports:
      - "80:80"
    depends_on:
      - reward
    restart: always

  reward:
    build:
      context: .
      dockerfile: ./reward/Dockerfile
    image: reward
    environment:
      FLAG: "ACSC{FAKE_FLAG}"
    restart: always
```

Flag は /reward にあるらしい．
とりあえず /reward/app.py を見てみる

```python
from flask import Flask
import os

app = Flask(__name__)


@app.route('/bounty', methods=['GET'])
def get_bounty():
    flag = os.environ.get('FLAG')
    if flag:
        return flag


if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=False)
```

```/boundy``` にアクセスするとフラグが返ってくるように思う．
ここにアクセスできないことが課題で，違うサービスからアクセスする必要がある．
index.html の Submit(terminal-submil) を押すと /report_bug に data が POST される．
/report_bug にアクセスすると，bot.js の visit が動いて，/triage にアクセスする．

```javascript
router.get("/triage", (req, res) => {
  try {
    if (!isAdmin(req)) {
      return res.status(401).send({
        err: "Permission denied",
      });
    }
    let bug_id = req.query.id;
    let bug_url = req.query.url;
    let bug_report = req.query.report;

    return res.render("triage.html", {
      id: bug_id,
      url: bug_url,
      report: bug_report,
    });
  } catch (e) {
    res.status(500).send({
      error: "Server Error",
    });
  }
});

router.post("/report_bug", async (req, res) => {
  try {
    const id = req.body.id;
    const url = req.body.url;
    const report = req.body.report;
    await visit(
      `http://127.0.0.1/triage?id=${id}&url=${url}&report=${report}`,
      authSecret
    );
  } catch (e) {
    console.log(e);
    return res.render("index.html", { err: "Server Error" });
  }
  const reward = Math.floor(Math.random() * (100 - 10 + 1)) + 10;
  return res.render("index.html", {
    message: "Rewarded " + reward + "$",
  });
});
```

/triage は ```triage.html``` のレンダリングとかをしていて，```isAdmin``` で Admin の確認(?) をしている．

```javascript
const isAdmin = (req, res) => {
  return req.ip === "127.0.0.1" && req.cookies["auth"] === authSecret;
};
```

```req.ip``` と ```req.cookies``` の確認が入る．
もう一つ謎のエンドポイントがある

```javascript
router.get("/check_valid_url", async (req, res) => {
  try {
    if (!isAdmin(req)) {
      return res.status(401).send({
        err: "Permission denied",
      });
    }

    const report_url = req.query.url;
    const customAgent = ssrfFilter(report_url);
    
    request(
      { url: report_url, agent: customAgent },
      function (error, response, body) {
        if (!error && response.statusCode == 200) {
          res.send(body);
        } else {
          console.error("Error:", error);
          res.status(500).send({ err: "Server error" });
        }
      }
    );
  } catch (e) {
    res.status(500).send({
      error: "Server Error",
    });
  }
});
```

/check_valod_url に url を渡すと代わりに url にアクセスしてくれる．
ただし，[ssrfFilter](https://github.com/y-mehta/ssrf-req-filter) があるので

### Web: DB Explorer

### Web: FastNote

### crypto: RSA Stream2

### crypto: strongest OAEP

### crypto: oblivion

### crypto: Jenga

### crypto: Strange Machine
