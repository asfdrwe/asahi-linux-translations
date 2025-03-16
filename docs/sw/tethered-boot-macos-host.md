---
title: テザーブート: macOS ホストマシン
---

2025/3/9時点の[tethered-boot-macos-host](https://github.com/AsahiLinux/docs/blob/main/docs/sw/tethered-boot-macos-host.md)の翻訳

訳注:原文のスペルミスやリンクミスは正しいものに修正

---
# macOSホストでのテザーブートセットアップ

この案内では、macOS ホストでのテザーブートで前もって必要なセットアップについて、より詳しく説明します。

## macOS ホスト

ホストの要件:

* 最新版の MacOS が動作する Apple のコンピュータ
  * ホスト上にソフトウェアのインストールやコンパイルに十分なディスクスペース
  * ホストに空き USB ポート
  * USB-A/USB-C または USB-C/USB-C ケーブル
  * [前提条件となるソフトウェアのインストール](#前提条件となるソフトウェアのインストール)

テスト環境:

* macOs Big Sur 11.7(20G817)を実行するiMac 27インチ後期2015

## シリアルポートを設定

macOs の m1n1 UART デバイス名は、Linux ホストのもの (Linux では `/dev/m1n1` と `/dev/m1n1-sec`) とは違います。一般的には以下のようなデバイス名です:

* `/dev/cu.usbmodemP_01` は m1n1 propy クライアントが使用するプライマリ UART デバイス
* `/dev/cu.usbmodemP_03` はテザリングされたマシンが起動した直後に、m1n1ハイパーバイザーに接続するためのセカンダリUARTデバイス

デバイス名は機器や設定によって変わる可能性があるので注意してください。不明な場合は[実際のデバイス名の検索](#実際のデバイス名の検索)で確認してください。

## m1n1 バックドアを起動

シャットダウン(電源オフ状態)して、対象機器に USB ケーブルを接続し、ホスト上で以下のコマンドを実行します:

```shell
~/asahi/m1n1/proxyclient/tools/picocom-sec.sh
```

ここで、テザリングされた機器を起動します。`picocom-sec.sh` スクリプトはデバイスを待ち、デバイスが現れると接続します。
そしてm1n1ハイパーバイザーのバックドアをトリガーし、ターミナルに出力が表示されるでしょう。


```console
picocom v3.1

port is        : /dev/cu.usbmodemP_03
flowcontrol    : none
baudrate is    : 500000
:
:
```

これで、m1n1ハイパーバイザーはm1n1プロキシクライアントツールからコマンドを受け付けることができるようになりました:

```shell
$ python3 ~/asahi/m1n1/proxyclient/tools/shell.py
:
:
TTY> Waiting for proxy connection... . Connected!
Fetching ADT (0x00058000 bytes)...
m1n1 base: 0x802848000
Have fun!
>>>
```

### 実際のデバイス名の検索

`pyserial` をインストールし、以下のコマンドを実行します (ターゲットマシンがホストに接続されていることを確認):

```shell
while : ; do pyserial-ports ; sleep 1 ; done
```

テザリングされた機器を(電源オフの状態から)コールドブートして、新しいデバイスが現れるのを待ちます。デバイス名を
メモしてください(番号の小さい方がメインのm1n1 UARTデバイス、もう一方がm1n1のはず)。

## 前提条件となるソフトウェアのインストール

### Homebrewをインストール

macOS ホストにソフトウェアをインストールする好ましい方法は`homebrew` パッケージマネージャを使用することで、単純な shell コマンドを
実行することです。

ターミナルウィンドウを開き (`[Cmd]`+`[Space]` キーを押し、`iterm` と入力して `[Enter]` を押す)、以下のコマンドを入力してください
 (不明な場合は [Homebrewウェブサイト](https://brew.sh) を参照してください)。

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Python >= 3.9 をインストール

macOS は Python 3.8 をバンドルしていますが、ホスト上で動作する m1n1 スクリプト部分と、テザリングされたマシン上の m1n1 ハイパーバイザーを 
操作するために、Python 3.9 以降が必要です。

```shell
brew install python3
```

目的の python 実行ファイルにアクセスできることと、必要最小限のバージョンを取得できることを確認します。

```shell
$ type python3
python3 is /usr/local/bin/python3
$ python3 --version
Python 3.10.8
```

### LLVMをインストール

m1n1とその依存関係、およびカーネルのビルドにはCコンパイラが必要です。LLVMは(私が思う？)Asahiの推奨コンパイラです:

```shell
brew install llvm
```

### pyserialをインストール

pyserial は m1n1 に必要です。m1n1 の起動時に macOS から公開されるシリアルポートデバイスの名前を特定するのに役立ちます。

```shell
pip3 install pyserial construct serial.tool
```

pyserial-ports` がインストールされていることを確認します:

```shell
$ type pyserial-ports
pyserial-ports is /usr/local/bin/pyserial-ports
```

### picocomをインストール

m1n1プロキシとの通信を確立するには、シリアルポート通信ソフトウェアが必要です。シリアルターミナルとして使用するために、
homebrewで利用可能な `picocom` のインストールをおすすめします:

```shell
brew install picocom
```

### img4tool をインストール (オプション)

Homebrew Tap を使うか、手動でインストールしてください。

#### Homebrew tapを使用

Homebrewでのインストールを容易にするためにtapを作成しました。tapを追加してインストールするだけです。

```shell
brew tap aderuelle/homebrew-tap
brew install img4tool
```

#### 手動でインストール

macOS の純正カーネルを起動する予定なら、対象機器上にインストールされた macOS の kernlecache から実際のカーネルファイルを
抽出するためにこれらのツールが必要になります。macOS 用のプリコンパイルバージョンがない場合は、コンパイルする必要があります。

この手順では、ホームディレクトリに `asahi` フォルダを作成し、そこにすべてをクローンします。さらに、システムの残りの部分を
ぐちゃぐちゃにさせないために、すべてを `~/asahi/deps` フォルダにインストールします。

まず、`libgeneral`をクローンし、ビルドしてインストールします:

```shell
cd ~/asahi
git clone https://github.com/tihmstar/libgeneral.git
cd libgeneral
./autogen.sh
./configure --prefix=/Users/alexis/asahi/deps
make && make install
```

次に、`img4tool`をクローンし、ビルドしてインストールします:

```shell
cd ~/asahi
git clone https://github.com/tihmstar/img4tool.git
cd img4tool
./autogen.sh
PKG_CONFIG_PATH=~/asahi/deps/lib/pkgconfig ./configure --prefix=~/asahi/deps
make && make install
```

img4tool` のバイナリにアクセスできるように `PATH` 変数を更新します。

```shell
export PATH=~/asahi/deps/bin:$PATH
```


