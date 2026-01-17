---
title: "Badusb"
date: 2024-04-14T13:53:47+09:00
draft: false
---

BadUSB をファームウェア書き換えでやってみる.

## はじめに

- 本ポストの実験を実際に試して何か被害があっても筆者は責任をもてません
- 本ポストは BadUSB の促進を目的にしていません
- 本ポストの内容を自分の環境以外で行うことは罪に問われる可能性があります
- 思い出しながら書いてるので抜けがある場合があります

## モチベーション

- 昔に参加したセキュリティキャンプで BadUSB を作る講座があった
- EasyUSB を使っていたのである意味やりやすかった
- 実際に使われている USB でもできないか実験したい

## 環境

- [Psychson](https://github.com/brandonlw/Psychson)
- Toshiba TC58TEG7T2JTA00
  - Phison 2251-03 (2303)
- 以下ブログを参考にしている
  - [Psychsonを使用したBadUSBの作成方法](https://kikuzou.hateblo.jp/entry/2014/11/21/143115)

## 手順

- SDCC をインストールする
  - 最新版をインストールする場合は Psychson のビルドに失敗するので，コードを書きかえる必要がある
- Psychson を展開する
  - DriveCom, EmbededPayload, Injector を Visual Studio でビルド(Build Solution)する
- バーナーイメージをダウンロード
  - Phison Electronics - USBDev.ru からバーナーイメージをダウンロードする
  - 今回は BN03V104M.BIN を使用
- ファームウェアをダンプ
  - ```tools\DriveCom.exe /drive=E /action=SetBootMode```
  - ```tools\DriveCom.exe /drive=E /action=SendExecutable /burner=BN03V104M.BIN```
  - ```tools\DriveCom.exe /drive=E /action=DumpFirmware /burner=BN03V104M.BIN /firmware=TC58TEG7T2JTA00.bin```
- カスタムファームウェアをビルド
  - ```build.bat```
  - SDCC の最新版をダウンロードした場合は以下を修正する
  - 割り込みハンドラ(```__interrupt XXX```) を ```__interrupt(XXX)``` に直す．つまり割り込み番号とレジスタバンク番号を括弧で囲む
  - 引数無しの関数で ```void``` が省略されているので追加する
- ペイロード作成
  - [USB-Rubber-Ducky](https://github.com/midnitesnake/usb-rubber-ducky) を使う
  - ```java -jar encoder.jar -i keys.txt -o inject.bin```
  - ```tools\EmbedPayload.exe inject.bin hid.bin```
  - key.txt の内容は以下

``` txt
DELAY 3000
GUI r
DELAY 500
STRING notepad
DELAY 500
ENTER
DELAY 750
STRING Hello world
ENTER
```

- 書き込み
  - ```tools\DriveCom.exe /drive=E /action=SetBootMode```
  - ```tools\DriveCom.exe /drive=E /action=SendExecutable /burner=BN03V104M.BIN```
  - ```tools\DriveCom.exe /drive=E /action=SendFirmware /burner=BN03V104M.BIN /firmware=hid.bin```

- その他
  - ファームウェアの書き込みに失敗した場合は手動でブートモードに直す必要がある
    - コントローラの 2-3 ピンをショートしながら USB メモリを差す
    - ディレクトリからは見えないがドライブレターが割り当てられ，コマンドが通るようになる
