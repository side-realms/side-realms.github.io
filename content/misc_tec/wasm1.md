---
title: "WebAssembly1：入門する"
date: 2023-10-07T02:33:12+09:00
draft: false
---

そろそろ wasm をやらないと死ぬぜ！

# wasm とは何か

ブラウザで JavaScript 以外の言語を実行するための使用・環境.
C とか Rust をコンパイルして Wasm として動かすことができる．
でもブラウザ以外でも動く．へぇ\
バイナリ形式なので，CPU が直接実行でき，早い．

## 歴史

Unix はハードウェアを抽象化した．
そのうち，CPU とメモリは C 言語が抽象化した．
これにより，プログラマは Unix で動作するプログラムを C 言語で書けばよく，プログラマと CPU と HW とそれぞれ開発者のすみわけが可能になった．

一方で，同じ HW であっても OS が異なればプログラムが動作しなくなる．
そこで，JVM が開発された．
JVM は Java アプリケーションを動かすことができるシステムで，OS を抽象化してくれる．
その JVM の規格にそったプログラムを開発するために Java アプリケーションが作られた．
ということで，OS を抽象化するための JVM を抽象化するための Java 言語が存在している，ということになる

一方で，OS を抽象化するためにオリジナルの VM を作ってわざわざその規格に倣うために Java を開発しようというのは嬉しくない．
そこで，VM を抽象化するための規格として WebAssembly がでてきた．
しかし，結局 HW や OS の影響は受けるので，「どこでも同じように動作する」のは難しい．
そこで，Wasm は OS を抽象化することをあきらめ，OS 非依存のみに対応することを決めた．

一方で，Wasm において OS とのやりとりを許可するような規格を作ろうという動きもあった．
これが WesAssembly System Interface という．

## wasmtime

- WebAssembly 用の軽量でコンパクトなランタイムで，Rust で実装されている．
- 実際に簡単な Hello World を作ってみる

### tutorial

```sh
rustup target add wasm32-wasip2
cargo new hello-wasm
cd hello-wasm
```

```rust
fn main() {
    println!("Hello, Wasm!");
}
```

```sh
cargo build --target wasm32-wasip2 --release
wasmtime target/wasm32-wasip2/release/hello-wasm.wasm
```

ちなみに，wasm32-wasip1 と wasm32-wasip2 は，Rust コンパイラが用意している WASI 対応の Wasm 32-bit 向けターゲット．
（Rust はコンパイル時にターゲットを指定することができる．このとき，どの CPU でどの OS API 向けにバイナリを吐くかが決まる． wasm32-wasip1 とは，webasssembly ISA アーキテクチャで，WASI Preview1 向けということになる．）

## Wasmtime サンドボックス検証

WASI は，先ほども言ったように wasm とネイティブコードの間で標準化されたインタフェース．
Wasm で動くから WASI も移植性があるとは限らないが，ある程度の移植性は保証される．
WASI のおかげでブラウザ外の wasm の実行も可能にし，サーバレスやエッジコンピューティングにも使われるようになった（らしい）．

wasm モジュールは，適切な権限が与えられない限りファイルシステムやネットワークにアクセスできない．
このため，悪意ある wasm モジュールが，許可なしに OS リソースを改変したりできない．

WASI は POSIX に似たシステムコールを定義しているが， wasm のユースケースに特化している．

```Rust
┌──(kali㉿kali)-[~/hello-wasm]
└─$ cat src/main.rs
use std::{
    env,
    fs::File,
    io::{self, Read, Write},
    process::exit,
};

fn main() -> io::Result<()> {
    let path = env::args().nth(1).unwrap_or_else(|| {
        eprintln!("usage: read_sample <file>");
        exit(1);
    });

    let mut file = File::open(&path).map_err(|e| {
        eprintln!("Error Opening Input: {e}");
        e
    })?;
    let stdout = io::stdout();
    let mut handle = stdout.lock(); 
    io::copy(&mut file, &mut handle)?;
    Ok(())
}
```

これは，ファイルを与えるとそれを開いてくれる．
普通に実行するとこんな感じ

```sh
┌──(kali㉿kali)-[~/hello-wasm]
└─$ ./target/release/hello-wasm sample_text 
hello hello
```

でも wasm で実行するとこうなる

```sh
┌──(kali㉿kali)-[~/hello-wasm]
└─$ wasmtime target/wasm32-wasip2/release/hello-wasm.wasm sample_text        
Error Opening Input: failed to find a pre-opened file descriptor through which "sample_text" could be opened
Error: Custom { kind: Uncategorized, error: "failed to find a pre-opened file descriptor through which \"sample_text\" could be opened" }
```

で，ファイルを明示的に与えてあげるとこうなる

```sh
┌──(kali㉿kali)-[~/hello-wasm]
└─$ wasmtime --dir=. target/wasm32-wasip2/release/hello-wasm.wasm sample_text
hello hello
```

同じ Rust のソースでも，ネイティブファイルとして実行すると，普通の OS 権限で実行できてしまう．
一方で，wasm32-wasi バイナリを wasmtime で実行すると，WASI の capability が機能するので，--dir で明示的に許可しない限りファイルを開くことができない．

これにより，プログラマは capability を意識した特別な API を書く必要がなくなる．

アプリが呼ぶ open() は，libc と同じだが，wasi-libc は musl を流用しているので，アプリ側からは違いが分からない．

```C
int open(const char *path, int oflag, ...) {
    // WASI libc's `openat` ignores the mode argument, so call a special
    // entrypoint which avoids the varargs calling convention.
    return __wasilibc_open_nomode(path, oflag);
}
```

内部では open() -> __wasilibc_open_nomode() -> find_relpath() となり，パスが preopen 済のディレクトリ下にあるかどうかを判定する．

```C
int __wasilibc_open_nomode(const char *path, int oflag) {
    char *relative_path;
    int dirfd = find_relpath(path, &relative_path);

    // If we can't find a preopen for it, fail as if we can't find the path.
    if (dirfd == -1) {
        errno = ENOENT;
        return -1;
    }

    return __wasilibc_nocwd_openat_nomode(dirfd, relative_path, oflag);
}
```