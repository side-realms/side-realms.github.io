---
title: "java が生成する乱数を予測する"
date: 2023-08-28T17:57:25+09:00
math: true
# weight: 1
# aliases: ["/first"]
# tags: ["first"]
# author: "Me"
# author: ["Me", "You"] # multiple authors
#showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: ""
canonicalURL: "https://canonical.url/to/page"
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
# ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: false
# ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
#editPost:
#    URL: "https://github.com/<path_to_repo>/content"
#    Text: "Suggest Changes" # edit text
#    appendFilePath: true # to append file path to Edit link
---

java が生成する17兆通りある乱数を予測してみる

### 1. 乱数生成をするスクリプトを書く

[ソース](https://github.com/side-realms/random/blob/main/HelloWorld.java)

### 2. java の乱数生成を調べる

「予測できるらしい」しかしらないのでちゃんと調べる

``` Java
Random random = new Random();
random.nextInt();
```

この nextInt() はソースコード内で next() を呼び出していて，next() のソースコードは以下のようになる

``` Java
protected synchronized int next(int bits)
    {
      seed = (seed * 0x5DEECE66DL + 0xBL) & ((1L << 48) - 1);
      return (int) (seed >>> (48 - bits));
    }
```

このとき乱数は seed で返されるが，それ以外の値は全て既知．
なので，ひとつ seed が分かればそれ以降の seed も知ることができる．
取り出す seed は必要な上位 bit 分を最後の演算で呼び出す．
例えば nextInt(32) で呼び出したとき，返ってくるのは 32 bit の乱数で，残りの 16 bit が未知ということになる．
これくらいなら簡単に[予測できる](https://github.com/side-realms/random/blob/main/brute.py)

### 3. 4 bit だけで予測する

予測すべき範囲が 2^44 bit になるので，さっきより甚大に増える．
そこで，もらえる乱数を増やして考える．
さっきの変数をそれぞれ a=0x5DEECE66DL, b=0xBL, m = ((1L << 48) - 1) とおく．

$$x_{i+1} = a x_i + b \mod m$$

以下では mod m を省略する．
係数で b が邪魔なので，与えられた複数の乱数 x を組み合わせると，例えば以下のようになる．

$$x_3 - x_2 = a(x_2 - x_1)$$

このとき，与えられた乱数を seed, それ以外の未知乱数(システム内部にある)を unknown_seed とすると，
以下のようになる．

$$x_2 - x_1 = (seed*2^{44} + unknown\\_seed)$$

m 倍すれば mod 演算の答えは 0 になる．

$$m(x_2 - x_1) = 0$$
$$(x_3-x_2) - a(x_2-x_1) = 0$$
$$(x_4-x_3) - a^2(x_2-x_1) = 0$$
$$(x_5-x_4) - a^3(x_2-x_1) = 0$$

で，これをさっきの式と展開すると，

$$m(x_2-x_1)=m(seed2^{44}+unknown\\_seed)=s~m$$

$$m~unknown\\_seed = s~m - m~seed2^{44}$$

となる．(s は係数で，x_2-x_1 以外は m が消えないのでいったん消さずにこのまま)
このとき，右辺が無視できるほど小さいとすると(ε)，

$$s~m - m~seed~2^{44} = ε$$
$$s~m = ε - m~seed~2^{44}$$
$$s = ε/m - m~seed~2^{44}/m$$

となるが，ε/m は無視できるほど小さいので，右辺は既知の数だけになる．
すると s が求まる．
先の右辺を小さくするために工夫をする必要がある．

s が求まれば，
$$x_2 = ax_1 + b$$
と合わせて $x_1, x_2$ が分かる．
$x_1 = (x_2-x_1-c)/(a-1) \mod M$
となるが，M と $(x_2-x_1-c)/(a-1)$ が互いに素でないので，逆数($a-1$) が求まらない．
そこで，$x_1(a-1) = (x_2-x_1-c) \mod 2$ みたいにすれば総当たりで得ることができる．

### 4. LLL

LLL を知らないのでいくつか例を交えつつ見てみる．

格子は，一次独立な n 本のベクトル $v_1, v_2, ..., v_n \in \mathbb{R}^m(1 \leq i \leq n)$
のすべての整数係数の線形結合で表される集合全体をいう．
単純な座標は(1,0), (0,1) からなる格子といえる．
このときのベクトルの集合 $ \\{v_1, v_2, ..., v_n\\}$ を格子の基底という．
$n$ を格子の次数という．

- SVP : 格子に含まれるベクトルのうち，もっとも短い($\neq 0$)ものを探す問題
- CVP : ベクトル空間に含まれるベクトル v が与えられたとき，これにもっとも近い格子 L 上の
点を見つける問題
- LLL : 同じ格子を張れるような別の短い・直交した基底を求める問題．

#### Markle-Hellman Knapsack 暗号

$n$ ビットの平文を暗号化するために，超増加列 $w$ とランダムな $r, q$ を用意する．
超増加列 $w$ は，$w_0 \leq w_1, w_0 + w_1 \leq w_2, ...$ となるような順列のことをいう．
また，$\beta = r~w \mod q$ とする．
$a$ を平文とする．
このとき，
$$ c = \sum a_i~\beta_i$$
で表される暗号文があり，$c, \beta$ から $a$ を求めることが難しい，というのが今回の問題．
このとき，$a_0\beta_0 + a_1\beta_1 + ... + -c = 0$ と式変形する．
このとき，未知の $a$ を $x$ とおき，LLL を使って，格子を保つ小さな $x$ を求めることができる．

\begin{array}{cc}
   (x_0 & x_1 & ... & 1) * (1 & 0 & ... &\beta_0)(0 & 1 &...& \beta_1)...
\end{array}

とすれば，計算結果は $(x_0~x_1~...~0)$ となるはず．
LLL は $(x_0 ~ x_1 ~ ... ~ 1)$ をなるべく小さくしたいので，LLL に投げれば求まる．
