2025/3/1時点の[Tethered-Boot-Setup-For-Developers](https://github.com/AsahiLinux/docs/blob/main/docs/Tethered-Boot-Setup-For-Developers.md)の翻訳

訳注: Asahi LinuxではM1 Macとは別にもう一台PCを用意してそのPCからM1 Macを制御することをテザー(tether)と呼び、M1 Macのみで行うことを 
テザーなし(untether)と呼んでいます。このページは開発者向けに別なPCを用意してM1 Macでの開発を行うための作業に関する解説です。

---
# はじめに
この案内ではApple Silicon MacとmacOSのデュアルブート環境でLinuxカーネルを起動するために必要なセットアップ手順を説明します。

この案内は<a href="https://github.com/AsahiLinux/linux">`asahi`</a> ブランチにあるパッチのテストを支援したい
カーネル開発者や上級ユーザーを特に対象としています。カーネルを構築することはこの案内の範囲外です。ここにいるのなら、
自分で AArch64 カーネルを構築できるはずです。ある程度まともな `.config` は [こちら](https://github.com/AsahiLinux/PKGBUILDs/blob/main/linux-asahi/config)で見ることができます。
m1n1 は gzip されたカーネルイメージ、対象マシンのDeviceTree、そして追加で gzip された _initramfs_ を必要とすることを
覚えておいてください。

## ハードウェア要件

* 少なくとも **macOS 12.3** がインストールされ、設定された Apple Silicon Mac
  * パスワードで保護された管理者アカウントを持っている必要あり。通常これはマシンを初めてセットアップする際に作成した最初のアカウント
  * GNU/Linux ディストリビューションが動作するあらゆるアーキテクチャのホストマシン(macOSにも対応するがよくテストされていない、[macOS上でのテザーブートセットアップ](Tethered-boot-setup-on-macOS.md)を参照)
  * GCCとClang/LLVM AArch64の両方のクロスツールチェーンに対応

デバッグ用 UART を介して SoC にローレベルでアクセスしたい場合は、実際の物理的なシリアルポートソリューションも必要です。
これについては、[[ローレベルシリアルデバッグ]]を参照してください。これは一般的なカーネル開発やリバースエンジニアリングには
必要ありませんが、ローレベルのハードウェア問題のデバッグを容易にするためには良いでしょう。

### 準備段階
m1n1` は Apple Siliconのplayground/hypervisor/bootloader で、Linux カーネルや U-Boot を起動するために必要なものです。
`m1n1` ツールを使うには、Python 3.9 以降と `pip` が機器にインストールされている必要があります。m1n1` と対話するために
必要な _Python_ モジュールを取得するには、次のコマンドを実行します:


```shell
pip3 install --user pyserial construct serial.tool
```

それから、m1n1 gitリポジトリをクローンしてビルドします。

```shell
git clone --recursive https://github.com/AsahiLinux/m1n1.git
cd m1n1
make
```

ネイティブの aarch64 マシンを使っている場合は、代わりに `make ARCH=` を使ってください。

```shell
curl -L https://mrcn.st/alxsh | sh
```

[udev rules](https://github.com/AsahiLinux/m1n1/blob/main/udev/80-m1n1.rules) を `/etc/udev/rules.d` にインストールし
、m1n1 のきれいなデバイス名を取得します。デバイスに root 以外でアクセスするために、ユーザーを特定のグループ (例: `uucp`) に
追加したり、デバイスのパーミッションを変更するために udev ルールを変更したりする必要があるかもしれません。

また、シリアル端末として使用するために `picocom` をインストールすることをお勧めします。これは、お使いのディストリビューション
パッケージリポジトリで利用できるはずです。

## Apple Silicon Macにm1n1をインストール
公開されているAsahi Linuxのインストーラーを使用してインストールすることができます。macOSのターミナルを開いて次のように実行します:

```shell
curl https://alx.sh | sh
```

プロンプトに従って望むインストールモードを選択します。Fedora Asahi Remix イメージから一つ
インストールするか、UEFI only オプションで m1n1+u-boot のみをインストールし、USB ドライブから UEFI 実行ファイルを起動します
 (インストール後は内部ストレージから起動します)。この場合、お好みの rootf/kernel のインストールは読者のための練習として残されています。

また、インストーラでエキスパートモードを有効にすると、テザリングのみのプロキシモードで m1n1 をインストールすることができます。
この場合、m1n1 は既に (無条件に) プロキシモードで起動するので、次のセクションをスキップできます。

## バックドアプロキシモードを有効化

m1n1は2つのステージで構成されています。ステージ 1 はインストールの 1TR ステップ (macOS リカバリへの最初のリブート後) で
インストールされ、リカバリを経由しないと変更できません。ステージ2はEFIシステムパーティションから `m1n1/boot.bin` でロードされ、
新機能やハードウェアサポートを追加するためにディストリビューションによって更新されることがあります。ステージ1のリリースビルドは
バックドアプロキシモードを持っており、オプションでテザーブートを可能にします。これは1TRから有効にする必要があり、セキュリティの
ためにマシン認証が必要です。


このモードを制御するために、Asahi Linuxボリュームに適用されるSIP（System Integrity Protection）無効化フラグを使用します。
macOSボリュームでは、これは通常特定のカーネルセキュリティ機能を無効にしますが、m1n1では、システムが冗長モードで起動している場合に、
バックドアプロキシモードを有効にすることを伝えるために使用します。この変更はOSごとに行われるため、Asahi/m1n1ボリュームに
この変更を加えても、macOSのインストールには影響しません。

このモードを有効にするために、まずm1n1 / Asahi Linuxがデフォルトのブートボリュームであることを確認し（新規インストール後は
このようになります）、それから完全にシャットダウンした状態から『起動オプションを読み込み中...』と表示されるまで電源ボタンを押し続けて機器を
起動させます。『オプション』を選択し、プロンプトが表示されたらmacOSマシンのオーナー情報を入力し、『ユーティリティ』メニューを
クリックし、ターミナルウィンドウを開いてください。

ここから、次のように実行:

```shell
csrutil disable && nvram boot-args=-v
```

プロンプトが表示されたらAsahi Linux のボリュームを選択し、プロンプトが表示されたら自分自身を認証します。これが完了したら、機器を
シャットダウンします。ペアリング・エラーが出る場合は、デフォルトの起動OSと変更するOSが一致しないことを意味します（デフォルトの起動OSの
セキュリティ設定は、どのrecoveryOSが起動するか制御するため、ダウングレードのみ可能です）。

Select your Asahi Linux volume when prompted, and authenticate yourself when prompted. Once this is done, shut down the machine. If you get a pairing error, that means the default boot OS does not match the OS you are making the change for (you can only downgrade the security settings for the default boot OS, as that controls which recoveryOS boots).

このモードでは、m1n1は起動時に5秒間待機します。USB接続が検出され、対応するTTYデバイス(どちらか)がホストマシンで開かれると、
通常のブートプロセスを中断し、プロキシモードに移行します。これにより、必要なときはテザリングモードで起動し、そうでないときは
スタンドアロンで起動させることができます。

## USB 接続を確立

ホストとターゲットをUSB Type Cケーブルで接続します（妥当なケーブルであれば何でもかまいません）。TBポートがないマシン（iMacやMac Studioの一部など）では、
ターゲットのThunderboltポートを使用していることを確認してください。また、C-A変換ケーブルを使用し、A側をホスト側にすることも可能です。
また、ターゲットの別のポートに充電ケーブルを接続することをお勧めします。

ホストマシンでターミナルウィンドウを開き、 `proxyclient/tools/picocom-sec.sh` を実行します。これはm1n1デバイスが接続されるまで待ち、
セカンダリUSBデバイスをシリアルターミナルとして開きます。これは二つの目的があります: プロキシモードに入るため (前のセクションを参照) と、
ハイパーバイザーの下でカーネルを実行するときの仮想シリアルコンソールになるためです。

このスクリプトが実行され、マシンが接続されたら、ターゲットデバイスを起動します。`proxyclient/tools/shell.py` を実行すると、対話型の Python シェルに
なります(^D で終了します)。

## カーネルを直接起動

Linuxカーネルを起動するには、以下のコマンドを使用します。

```shell
python3 proxyclient/tools/linux.py -b 'earlycon debug rootwait root=/dev/nvme0n1p5 <other args>' /path/to/Image.gz /path/to/t6000-j314s.dtb [optional initramfs].
```

この例では、`m1n1` に gzip されたカーネルと、14" MacBook Pro with M1 Pro SoC のDeviceTreeを渡して、rootファイルシステムとして 
/dev/nvme0n1p5 をマウントするように指定しています。`linux.py` に渡す引数の順番が重要であることに注意してください。まずカーネルを渡し、
次にDeviceTreeを渡し、オプションで initramfs を最後に渡さなければなりません。すべてがうまくいけば、フレームバッファにペンギンが表示されるはずです。

Asahi Linux (Arch Linux ARM) の rootfs を使っている場合、テザーブートにはモジュールを無効化したカーネルを推奨します（全てビルトインです）。
Arch Linux の initramfs は無視してもかまいませんが、代わりに `/lib/firmware/{brcm/apple}` を含む initramfs を用意して、
ビルトインモジュールが早期にファームウェアを見つけることができるようにする必要があります。それから、`root=/dev/nvme0n1p5` 
(rootfs のパーティション番号を代入してください。5 は vanilla インストールの典型的な番号です) を渡すだけで、システムを直接起動することができます。

## ハイパーバイザー下でカーネルを起動

m1n1 ハイパーバイザーの下で Linux カーネルを正しく起動するこのユーティリティスクリプトを使ってください:

```shell
tools/run_guest_kernel.sh /path/to/linux/build/dir 'earlycon rootwait root=/dev/nvme0n1p5' [optional initramfs]
```

このスクリプトはLinuxのビルドツリー（すなわち `.config` と `Makefile` があるディレクトリ）へのパスを想定していることに注意してください。
スクリプトはその中からカーネルとそれに対応するDeviceTreeを選び出します。

これは

* m1n1, 希望するカーネル, コマンドライン, initramfs (あれば), そしてカーネルツリーから利用可能なすべてのデバイスツリー (m1n1 は正しいものを自動選択) でゲストイメージを構築
* ターゲットマシーンがホストの m1n1 ツリー/ツールに一致するバージョンを実行していることを確認するために、最初にプレーンな m1n1 バイナリをチェーンロード (プロキシ ABI が安定していないため)
* ハイパーバイザーでゲストイメージを起動

m1n1はまずm1n1内部のゲストとしてロードされ(発端！)、内部ゲストはその後組み込みカーネルとinitramfsをロードします。ゲスト m1n1 のデバッグ出力とカーネルコンソールが、先ほど (`picocom-sec.sh` で) 起動したセカンダリターミナルに表示されるはずです。

このスクリプトで initramfs を使用する場合、*gzip*で圧縮されなければならないことに注意してください (そして単一の gzip イメージでなければなりません。
連結してからgzip、gzipしてから連結してはダメ...)。これは、m1n1が埋め込みペイロードを処理する方法の制限によるものです。

ハイパーバイザーコンソールで ^C を使用すると、ゲストに侵入することができます。スクリプトは自動的に `System.map` をロードするので、
`bt` コマンドを使ってシンボルを含むスタックトレースを取得することができます。ゲストの CPU を切り替えるには `cpu(1)` (など)、実行コンテキストを
表示するには `ctx` 、システムを強制的に再起動するには `reboot` 、実行を再開するには `cont` (または単に ^D) を試してみてください。


ソースラインの位置の取得や構造体の調査などもGDBやLLDBを用いたデバッグで可能です。ハイパーバイザーのコンソールで `gdbserver` コマンドを実行し、GDB/LLDB を `/tmp/.m1n1-unix` UNIX ドメインソケットに接続してください。gdbserver にはいくつかの注意点があります:

- GDBはカーネル内ポインタ認証対応なし。ポインタの問題を避けるために`CONFIG_ARM64_PTR_AUTH_KERNEL` を無効にするか LLDB を使用
- GDB/LLDB と干渉するハイパーバイザーのコンソールコマンドを実行しない。実行すると同期が取れなくなる。例えば、ハイパーバイザーコンソールと GDB/LLDB の両方から同時にブレークポイントを編集しないように
