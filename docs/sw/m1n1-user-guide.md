---
title: m1n1 ユーザガイド
---

2025/3/9時点の[m1n1-user-guide](https://github.com/AsahiLinux/docs/blob/main/docs/sw/m1n1-user-guide.md)の翻訳

---
m1n1は、Apple（XNU）ブートエコシステムとLinuxブートエコシステムの橋渡しをするために、Asahi Linuxプロジェクトが開発したブートローダです。

GitHub: [AsahiLinux/m1n1](https://github.com/AsahiLinux/m1n1)

## 何をするのですか

* ハードウェアの初期化
* きれいなロゴを表示
* ペイロードをロード
  * DeviceTree(FDT)、プラットフォームに応じて自動選択
  * Initramfsイメージ(CPIO圧縮アーカイブ)
  * Linux ARM64ブートフォーマットのカーネルイメージ（オプションで圧縮可）
  * Configuration statement
* FAT32パーティションから自身の別バージョンをチェーンロード可能

プロキシモードでは、Linux カーネルのテストサイクルを 7 秒に短縮したり、ハードウェアのライブプローブや実験を行ったり、
USB 上の仮想 UART を提供しながらmacOS や Linux を実行してハードウェアアクセスをリアルタイムでトレースできる
ハイパーバイザーなど、膨大な開発用ツールセットを使用することができます。これについては[m1n1:開発者ガイド](m1n1-dev-guide.md)を
参照してください。このガイドでは、簡単なプロキシの使用例のみを説明します。

# ビルド
`aarch64-linux-gnu-gcc` クロスコンパイラツールチェーン (またはARM64で動かす場合はネイティブのもの) が必要です。
また、ブートロゴには (ImageMagickの) `convert` が必要です。

```
git clone --recursive https://github.com/AsahiLinux/m1n1.git
cd m1n1
make
```

build/m1n1.{bin,macho}に出力されます。

arm64のネイティブマシンでビルドする場合は、`make ARCH=`を使用します。

ARM64 macOS上でのビルドは clang と LLVM で対応しています。 必要な依存関係をインストールするために
Homebrew を使う必要があります。


```shell
$ brew install llvm imagemagick
```

その後`make`と打ってください。

### コンテナセットアップを使用したビルド
PodmanやDockerなどのコンテナランタイムがインストールされているなら、すべてのビルドの依存関係を含むcompose
セットアップを利用することができます。

```shell
$ git clone --recursive https://github.com/AsahiLinux/m1n1.git
$ cd m1n1
$ podman-compose build m1n1
$ podman-compose run m1n1 make
$ # or
$ docker-compose run m1n1 make
```

### ビルドオプション

* `make RELEASE=1`でm1n1リリース動作が有効に。これはデフォルトでコンソールを隠し、
早期プロキシモード([プロキシモード](#プロキシモード)を参照)を有効にするための避難口を提供

* `make CHAINLOADING=1` でチェーンローディング対応を有効に。これには aarch64 をサポートする Rust nightly toolchainが
必要。次の手順で入手可: `Rustup toolchain install nightly && rustup target install aarch64-unknown-none-softfloat`

Asahi Linuxインストーラでパッケージされたm1n1ステージ1のリリースビルドはこの両方のオプションが設定されています。
ディストリビューションでパッケージされたm1n1ステージ2のリリースビルドは、単に `RELEASE=1` にすべきです
（さらにチェーンロードする必要がないため）。したがって、ビルドにRustは必要ありません。

## インストール

### ステージ1 (fuOSとして)

m1n1（ペイロードを選択可能）は、1TR（macOS 12.1 OS/stub以降）から以下の手順でインストールできます。

```
kmutil configure-boot -c m1n1-stage1.bin --raw --entry-point 2048 --lowest-virtual-address 0 -v <OSボリュームのパス>
```

古いバージョンでは（推奨しませんが）、代わりに`macho`が必要です。

```
kmutil configure-boot -c m1n1-stage1.macho -v <OSボリュームへのパス>
```

通常、Asahi Linuxのインストーラがこの作業を行ってくれますので、ほとんどのユーザーはこの作業を手動で改めて行う必要はありません。

各OSにはそれぞれリカバリ用のOSがあります。このコマンドを実行するには、Asahi Linuxのリカバリに移動しなければなりません。そうでないと
`not paired`のようなエラーが出ます。

正しいリカバリOSでない場合は、以下の方法で正しいリカバリOSに移動することができます:

```shell
/Volumes/Asahi\ Linux/Finish\ Installation.app/Contents/Resources/step2.sh
```

### ステージ2 (ESP内)

ステージ2のm1n1は通常はEFIシステムパーティションに格納され、典型的にはペイロードとしてU-Bootが格納されます。
ESP が `/boot/efi` にマウントされていると仮定すると、通常は以下のようになります。

```
cat build/m1n1.bin /path/to/dtbs/*.dtb /path/to/uboot/u-boot-nodtb.bin > /boot/efi/m1n1/boot.bin
```

m1n1にdtb(訳注:DeviceTree)とubootを追加する方法の詳細については、以下の`ペイロード`の`ステージ2の設定`を参照してください。

## ペイロード

m1n1では以下のペイロードをサポートしています。

* DeviceTree（FDT）、オプションで gzip/xz 圧縮
* Initramfsイメージ（gzip/xzで圧縮されている*必要*があり)
* arm64 Linux スタイルのカーネルイメージ、gzip/xz 圧縮されている*必要*あり
* 『var=value\n』形式の設定変数

ペイロードは最初の m1n1 バイナリの後に単純に連結されます。

### ステージ1の設定

m1n1ステージ1は、Asahi Linux Installerで、次のように変数を付加して設定されます。

```
cp build/m1n1.bin m1n1-stage1.bin
echo 'chosen.asahi,efi-system-partition=EFI-PARTITION-PARTUUID' >> m1n1-stage1.bin
echo 'chainload=EFI-PARTITION-PARTUUID;m1n1/boot.bin' >> m1n1-stage1.bin
```

**落とし穴:** UUIDが小文字であることを確認してください。そうしないと、DeviceTreeを使用する箇所の一部（特にU-Boot）がシステム部分を見つけられなくなります。

### ステージ2の設定

m1n1ステージ2は通常ペイロードを直接ブートし、加えてステージ1から`chosen.asahi,efi-system-partition`の設定を自動的に受け取ります。
U-Bootに同梱されているデバイスツリーを使って次のようにします。

```
cat build/m1n1.bin \
    ../uboot/arch/arm/dts/"t[86]*.dtb \
    <(gzip -c ../uboot/u-boot-nodtb.bin) \
    >m1n1-uboot.bin
```

U-Boot は追加前に圧縮されることに注意してください。圧縮されていないカーネルはサイズが正確に判断できないため、変数が失われる問題が
発生する可能性があります。

### Linuxを直接起動するための設定

m1n1はLinuxカーネルとinitramfsをステージ1または2で直接起動することができます（ただし、おそらくステージ1ではこれを実行したく
ないでしょう）。ブート引数を直接指定することができます。カーネルに同梱されているデバイスツリーを使って次のようにします。

```
cat build/m1n1.bin \
    <(echo 'chosen.bootargs=earlycon debug root=/dev/nvme0n1p6 rootwait rw') \
    ${KERNEL}/arch/arm64/boot/dts/apple/*.dtb \
    ${INITRAMFS}/initramfs.cpio.gz \
    ${KERNEL}/arch/arm64/boot/Image.gz \
    >m1n1-linux.bin
```

ここでも圧縮されたカーネルイメージを使用していることに注意してください。また、複数の initramfs イメージを連結する場合、
*まず最初に伸長*し、それから連結して再度圧縮する必要があります ([バグ](https://github.com/AsahiLinux/m1n1/issues/157))。

### プロキシモードの設定

ペイロードが与えられない場合、またはペイロードのブートに失敗した場合、m1n1 はプロキシモードにフォールバックします。

## プロキシモード

プロキシモードはデバッグ用のUSBデバイスインターフェイス（すべてのThunderboltポートで利用可能）を提供します。
使用するには、ターゲットデバイスとホストデバイスをUSBケーブルで接続します（例：USB-C to USB-Aケーブル、
C側がm1n1側になるように）。恐ろしく細かい詳細は[m1n1:開発者ガイド](m1n1-dev-guide.md)をご覧ください(訳注:2022/3/17時点では
全く詳細が書かれていないです)。これらは、あなたができることの簡単な例に過ぎません。

プロキシモードでは、Linuxホストには2つのUSB TTY ACMデバイスが表示され、通常は/dev/ttyACM0 & /dev/ttyACM1になります
(macOSでは/dev/cu.usbmodemP_01 と /dev/cu.usbmodemP_03)。
最初のものが適切なプロキシインターフェースで、
2番目のものはハイパーバイザーの仮想 UART 機能で使用するために予約されています。環境変数 `M1N1DEVICE` に
正しいデバイスへのパスを設定する必要があります。

### Linux カーネルの起動

```
export M1N1DEVICE=/dev/ttyACM0
proxyclient/tools/linux.py <path/to/Image.gz> <path/to/foo.dtb> <path/to/initramfs.cpio.gz> -b "boot arguments here"
```

### 別の m1n1 をチェーンロード


```
export M1N1DEVICE=/dev/ttyACM0
proxyclient/tools/chainload.py -r build/m1n1.bin
```

### 仮想 UART を使って m1n1 ハイパーバイザーのゲストとして Linux カーネルを実行

まず、シリアルターミナルでセカンダリポート(例: `/dev/ttyACM1`)を開きます。

```
picocom --omap crlf --imap lfcrlf -b 500000 /dev/ttyACM1
```

そして、カーネルを起動します。

```
proxyclient/tools/chainload.py -r build/m1n1.bin
cat build/m1n1.bin \
    <(echo 'chosen.bootargs=earlycon debug rw') \
    ../linux/arch/arm64/boot/dts/apple/*.dtb \
    <initramfs path>/initramfs-fw.cpio.gz \
    ../linux/arch/arm64/boot/Image.gz \
    > /tmp/m1n1-linux.macho
python proxytools/tools/run_guest.py -r /tmp/m1n1-linux.macho
```

run_guest.py には `macho` バージョンが使用されていることに注意してください (binにはまだ対応していません)。
また、最初に m1n1 をチェーンロードすることに注意してください。ハイパーバイザーの ABI は非常に不安定なので、
これは *必須* です。

生の `m1n1.bin` を使用し、`r` を `run_guest.py` に渡していることに注意してください。XNU以外のバイナリのMach-Oサポートは非推奨となりました。
Linux ベースのペイロードを Mach-O バージョンの m1n1 でビルドしないでください。

もしくは`run_guest_kernel.sh` スクリプトを使用することで、この作業の煩雑さを大幅に軽減することができます。

```sh
$ ./proxyclient/tools/run_guest_kernel.sh [kernel build root] [boot args] [initramfs (optional)]
```

### macOSカーネルをm1n1 ハイパーバイザーとして実行

[m1n1 Hypervisor](m1n1-hypervisor.md)を参照

### ステージ1リリースビルドにおけるバックドア・プロキシ・モード

m1n1の標準リリースビルドをfuOSとしてインストールしている場合（つまり、Asahi Linuxのインストーラを実行した場合に得られるもの）、
1TRに入り、macOSターミナルから以下の操作を行うことで、詳細メッセージとバックドアプロキシモードを有効にすることができます。

```
csrutil disable
```

マルチブートの場合適切なブートボリュームを選択するように促され、自身を認証するよう求められます。

次のようにして詳細メッセージを有効にできます。

```
nvram boot-args=-v
```

最初に有効にすると、この機能は次のようにしてオフにすることができます。

```
nvram boot-args=
```

こうすることで、m1n1は詳細ロギングをオンにし、5秒待ってからペイロードをブートするようになります。
もし、その間にUSBプロキシ接続を受けたら、プロキシモードになります。これは、動作する自動起動のLinuxインストールをしたいが、
何か問題が発生したときにプロキシモードでカーネルを起動する能力を保持したい場合や単に高速開発のために非常に便利です。

この状態は、インストールされたOSが `boot-args` プロパティを変更できるため、安全性が低いことに注意してください。
`csrutil enable` でブートポリシーをリセットしてください(プロンプトが出たときに完全なセキュリティを有効にすることを選択しないでください。m1n1 がアンインストールされます)。

プロキシモードに移行するには、ホストは USB ACM デバイスを*開く*必要があります (2 つのうちどちらか一方でいいです)。
これは、例えば、セカンダリインターフェース上でシリアルターミナルをループさせることで可能です
 (これはハイパーバイザーのために必要かもしれません): _注意: macOSで使う場合_ `/dev/cu.usbmodemP_01`

```
while true; do
    while [ ! -e /dev/ttyACM1 ]; do sleep 1; done
    picocom --omap crlf --imap lfcrlf -b 500000 /dev/ttyACM1
    sleep 1
done
```

picocom が接続すると、`M1N1DEVICE=/dev/ttyACM0` でプロキシスクリプトを呼び出すことができるようになります。

注意: プロキシバックドアは、冗長モード (`boot-args=-v`) と無効な SIP (`csrutil disable`) の両方が必要です。
冗長モードを使うだけでは m1n1 のデバッグ出力は得られますが (Apple の boot-arg フィルタリングポリシーに依存。
変更の可能性あり)、最近の m1n1 ビルドではセキュリティのためにプロキシバックドアは有効になりません。
