---
title: "Alpacahack1"
date: 2024-09-05T23:17:34+09:00
draft: true
---

すげ～プラットフォームができていた.
上から順に解いていく

## Simple Login

```python
from flask import Flask, request, redirect, render_template
import pymysql.cursors
import os


def db():
    return pymysql.connect(
        host=os.environ["MYSQL_HOST"],
        user=os.environ["MYSQL_USER"],
        password=os.environ["MYSQL_PASSWORD"],
        database=os.environ["MYSQL_DATABASE"],
        charset="utf8mb4",
        cursorclass=pymysql.cursors.DictCursor,
    )


app = Flask(__name__)


@app.get("/")
def index():
    if "username" not in request.cookies:
        return redirect("/login")
    return render_template("index.html", username=request.cookies["username"])


@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        username = request.form.get("username")
        password = request.form.get("password")

        if username is None or password is None:
            return "Missing required parameters", 400
        if len(username) > 64 or len(password) > 64:
            return "Too long parameters", 400
        if "'" in username or "'" in password:
            return "Do not try SQL injection 🤗", 400

        conn = None
        try:
            conn = db()
            with conn.cursor() as cursor:
                cursor.execute(
                    f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
                )
                user = cursor.fetchone()
        except Exception as e:
            return f"Error: {e}", 500
        finally:
            if conn is not None:
                conn.close()

        if user is None or "username" not in user:
            return "No user", 400

        response = redirect("/")
        response.set_cookie("username", user["username"])
        return response
    else:
        return render_template("login.html")
```

```"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"``` という SQL 文が適用されるので，SQLi ができる．
init.sql を見ると，```INSERT INTO flag (value) VALUES ('Alpaca{REDACTED}');``` という命令が見えるので，これを見ればいい．
```UNION``` とかがいいのかなというかんじ

ただし，シングルクオーテーションが使えないので，工夫する必要がある．
```{username}``` の後のシングルクオーテーションがエスケープできれば ```{username} ~ password =``` となるので ```UNION``` を続けられる．
エスケープは ``` \' ``` でできるっぽい．
そうするとあとは UNION 分を作ればいいだけなので， ```UNION SELECT value, value FROM flag #``` とかでいい

## echo

入力したサイズのバッファに文字列を入力するとそれを echo してくれるプログラム．

```C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define BUF_SIZE 0x100

/* Call this function! */
void win() {
  char *args[] = {"/bin/cat", "/flag.txt", NULL};
  execve(args[0], args, NULL);
  exit(1);
}

int get_size() {
  // Input size
  int size = 0;
  scanf("%d%*c", &size);

  // Validate size
  if ((size = abs(size)) > BUF_SIZE) {
    puts("[-] Invalid size");
    exit(1);
  }

  return size;
}

void get_data(char *buf, unsigned size) {
  unsigned i;
  char c;

  // Input data until newline
  for (i = 0; i < size; i++) {
    if (fread(&c, 1, 1, stdin) != 1) break;
    if (c == '\n') break;
    buf[i] = c;
  }
  buf[i] = '\0';
}

void echo() {
  int size;
  char buf[BUF_SIZE];

  // Input size
  printf("Size: ");
  size = get_size();

  // Input data
  printf("Data: ");
  get_data(buf, size);

  // Show data
  printf("Received: %s\n", buf);
}

int main() {
  setbuf(stdin, NULL);
  setbuf(stdout, NULL);
  echo();
  return 0;
}
```