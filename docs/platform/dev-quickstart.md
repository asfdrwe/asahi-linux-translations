---
title: 開発者クイックスタートガイド
---

2025/6/30時点の[Developer Quickstart](https://github.com/AsahiLinux/docs/blob/main/docs/platform/developer-quickstart.md)の翻訳

訳注: 
- 訳者は以下の内容を検証していません。トラブルが生じた場合は自己責任でお願いします。

---
# この案内はほとんど時代遅れです

この案内は、m1n1ハイパーバイザーを使ってmacOSドライバーのリバースエンジニアリングを行うカーネル開発者だけが必要とする、
手動によるインストールプロセスを文書化したものです。

もしあなたがより一般的な開発者で、カーネルの作業を手伝いたい、あるいは単にプラットフォームで実験したいだけで、USB 経由の
テザーブート (別のマシンからカーネルをアップロードする) に満足しているなら、[こちら](../sw/tethered-boot.md)をご覧ください。

もしエンドユーザであったりスタンドアロンインストールをセットアップしたいのであれば、おやめください。
まだ準備ができていないのです。もうまもなくです。頑張れ！

# 開発者向けクイックスタート

M1 MacにLinuxを導入しようとしている私たちを手伝いたいと考えていますか？お読みください！この案内はM1上でのLinuxを開発するためのおすすめ
セットアップ方法について説明します。

この案内はAsahi Linuxをハックしたいと考えている開発者のためのものです。エンドユーザーがインストールする際の手順を示すものではありません。この案内はMac や M1 マシンについては何も知らないが経験豊富な Linux 開発者を対象としています。

この手順に従うことでコンピュータに損害を与える可能性は極めて低いですが、現時点では何も保証できません。ご自身の責任において作業を進めてください。

簡単な手順についてはこの[簡潔な手順](https://tg.st/u/asahi.txt)をご覧ください。物事は急速に進んでいるので、実際には詳細が変更されているでしょう。

## はじめに

### 目的

ここでは2種類のOSのデュアルブート M1コンピューターをセットアップします。無改変のmacOS（オプションでセキュリティを低セキュリティ/セキュリティ制限なしにすることも可能）と、Linux（または m1n1経由で実行したいもの）のブートラッパーとして機能する許容的セキュリティを備えた『スタブ』の macOSの2つです。また、ハードウェアシリアルコンソールを利用する方法についても学びます。

### 前提条件

* Apple M1 マシン 
    基本ライン対象として Mac Mini をおすすめします。
    * Mac Mini: Type C 電源が不要。オンボードの PCIe ハードウェア（Type A ポートには Ethernet と xHCI）を搭載、HDMIモニターが必要
    * MacBook Air: DWC3/Type-C経由のUSBのみ。スクリーン内蔵、SPIキーボード、充電にType-C電源が必要
    * MacBook Pro: Airと同様だが、Fキーの代わりにTouch Barが搭載されていてドライバが作成されるまではキーボードが使用不可

* マシンにmacOSがインストール済み（macOSパーティションが1つの純正構成から始めることを想定)
    * FileVault（暗号化）なし、事態を複雑にするので未テスト

* Linuxの開発マシン（アーキテクチャは何でもいい)
    * arm64でない場合：`aarch64-linux-gnu`のgccクロスコンパイラ/ツールチェイン（m1n1ビルド用にclangにも対応しているがテスト不十分）

強く推奨:

* シリアルポートのソリューション(後述)

### M1 Mac Bootの要約

ストレージ：内蔵SSDはGPTパーティションのディスク、（初期状態では）APFSコンテナ格納、各コンテナには複数のAPFSボリューム格納

ブートプロセス:

1. M1 シリコンのSecureROM(Boot ROM)が起動
    * DFUモード(USBリカバリー)に入るか、または
    * NOR(SPI)フラッシュからiBoot1を起動
2. iBoot1 (グローバル) は iSC Preboot内のブートポリシーを検索し、OSの(NVRAM で定義されている)APFS Prebootボリューム からアクティブなものを起動
    * 電源ボタンが押されている場合、代わりに 1TR を起動
    * 画面を初期化しAppleロゴを表示
3. iBoot2 (OSごとに存在 ≒ OSローダー)がOSを起動
    * OS の APFS プリブートパーティションから二次的CPUエンジンファームウエアをロード
    * 次にPreboot パーティションから img4 コンテナでラップしたmach-o カーネルをロード
    * ADT(Apple Device Tree)の調整と受渡しを担当
4. OS カーネル（XNU/Darwin）を起動
    * ルートファイルシステムのマウント等

詳しくは[Boot](open-os-interop.md)や[PC Boot プロセスとの差異](pc-boot-differences.md)をお読みください。

ミニ用語集:

* 1TR: One True Recovery: 信頼されたシステムレベルのリカバリ、すべてのブートユーザとの対話（ブートピッカー）も含む、独自のAPFSコンテナ（最終パーティション）に格納
* APFS: Apple's fancypants filesystem (Appleの気取ったファイルシステム)
    * APFS コンテナ: ≒ ZFS プール (通常は GPT パーティションで保持)
    * APFSボリューム: ≒ ZFSデータセット
    * APFSスナップショット:　≒ ZFSスナップショット
* APチケット：特定のマシン上で特定のmacOSバージョンを実行することを許可するAppleからの署名入りブロブ、『macOS版』 ≒ iBoot2とファームウェアバンドル、『一般的な』ものは低セキュリティモードで使用可能（つまりこの場合はphone homeは不要）(訳注: 原文: "i.e. no phone home requirement in that case"、[phoning home](https://en.wikipedia.org/wiki/Phoning_home))
* ブートポリシー：特定のマシンで起動可能なOSを宣言。SEPが署名と検証。iSCに格納
* iBoot1。iBoot1: 第一段階のブートローダ。起動するOSを探索
* iBoot2: iBoot2: 第ニ段階のブートローダ。オンダイの二次コアファームウェアを実行しXNUカーネル(またはのm1n1)を起動
* iSC: iBoot System Container。APFSコンテナにブート設定やデータを格納
* kernelcache: カーネル + kexts ブロブ(≒ linux + モジュールを保持するinitramfs)
* kext: カーネルモジュール
* NOR: iBoot1を含むボード上のSPI NORフラッシュで、製品カスタマイズデータやnvram
* SecureROM: M1シリコンダイ上のブートROM
* SEP: Secure Enclave Processor ≒ 他のプラットフォームではTPMまたはセキュリティモニタ(M1では別のコア)
* SFR: System Firmware and Recovery (iBoot1 + iSCパーティション + 1TR パーティション)、ダウングレードは保護
* スタートアップセキュリティモード: 
    * Full Security(完全なセキュリティ): 完全なセキュアブートで、ユーザーが許可した署名済みでアドホックなユーザースペースバイナリをmacOS上で実行する場合を除き、アップル以外のソフトウェアは実行不可能、APチケットを強制（macOSのダウングレードは不可）
    * Reduced Security(低セキュリティ): サードパーティのKextsや管理などを許可
    * Permissive Security(セキュリティ制限なし): 署名されていないカーネルを許可（私たちが望むもの)
* XNU: Darwinのベースで、macOSのカーネル

注意事項: 

* M1マシンはブートローダレベルのユーザインタラクションやキーボードショートカットなどに『一切非対応』
    * iBoot1が見ることができるのは電源ボタンを押しているかどうかだけ
    * 『ブートピッカー』は、1TRで動作するmacOSアプリケーション。実際にそうなっている
* M1マシンは外部ストレージからの起動は『不可能』
    * ブートピッカーで外部ディスクを選択するとそのプリブートパーティション全体がiSCにコピーしブート・ポリシーを生成。事実上Linuxの/boot全体を内部ストレージに置くのと同じ。少なくとも現在のファームウェアでは**これらのマシンでは外部ストレージからmacOSカーネルを起動することは不可能**

### 参考

* [用語集](../project/glossary.md)
* [Boot](SW-Boot.md)
* [PC Boot プロセスとの差異](pc-boot-differences.md)
* [NVRAM](../fw/nvram.md)
* [既製品パーティションレイアウト](stock-partition-layout.md)

### うまくいかない場合

M1 Mac は、SPI フラッシュに書き込まない限り、実際上文鎮化しません (Linux のDeviceTree内にNOR Flashを宣言することは強く非推奨。現時点ではこれに触れる理由がないので大惨事のリスクを大幅に高めるだけ) 。しかし、内部のSSDストレージにインストールされたソフトウェアに基づいて通常のブートや1TRの起動を行うようになっています。

Macが起動できない状態になった場合は、[appleによるDFU回復手順](https://support.apple.com/en-in/guide/apple-configurator-2/apdd5f3c75ad/mac)に従ってください。この作業を行うにはmacOSが動作している別のMacが必要になります（Intel製でも問題ありません）。このガイドの手順に普通に従っていれば起こらないはずですが、誤って内部ストレージに不正な書き込みをしてしまった場合に起こる可能性があります（ブロックデバイス全体を消去してしまったりGPTパーティションテーブルを書き換えてしまった場合等）。

最終的には[idevicerestore](https://github.com/libimobiledevice/idevicerestore)を使ってLinux マシンから同じ手順で実行できるようになるはずですが、まだ準備できていません。

DFUブートは完全に破損したNORフラッシュでも動作しますが、そこには重要な製品識別情報やキャリブレーションデータが保存されており、Appleの助けなしには復元できません。そのため、NORフラッシュには触れないようにしてください。

## セットアップ

### ステップ0: macOS のアップグレード

少なくともmacOS 11.2が必要です。古いバージョンでは動作しないのでご注意ください。

また、パスワード付きの管理者ユーザーを作成するなど、macOS の初期設定を行っておく必要があります。

### ステップ1: パーティション設定

開始時:

* disk0s1: iBoot システムコンテナ
* disk0s2: Macintosh HD (OS #1 APFS コンテナ)
* disk0s3: 1TR (One True Recovery OS)

こうしたい:

* disk0s1: iBootシステムコンテナ
* disk0s2: Macintosh HD (OS #1 macOS APFS コンテナ)
* disk0s3: Linux (OS #2 "偽macOS" APFSコンテナ)
* disk0s4: Linux /boot (FAT32)
* disk0s5: Linux / (ext4)
* disk0s6: 1TR (One True Recovery OS)

ストレージは我々の開発ツリーやブートローダではまだサポートされていませんが、アプリオリにこのパーティションレイアウトを対象としていることに注意してください。
disk0s3は『スタブ』macOSで、現時点では完全なmacOSをインストールするために使用されます（最終的には、無駄なスペースを避けるために、
ブートファイルのみを含む最小限のパーティションにするためのツールを提供する予定ですが、これはまだ準備できていません）。

500GBモデルをお持ちの場合、以下のコマンドを実行してください（これらのコマンドを実行している間マシンが数分間フリーズすることがありますのでご注意ください）:

```shell
# diskutil apfs resizeContainer disk0s2 200GB
# diskutil addPartition disk0s2 APFS Linux 80GB
# diskutil addPartition disk0s5 FAT32 LB 1GB
# diskutil addPartition disk0s4 FAT32 LR 0
```

スタブパーティションには最低でも80GBが必要です。これはmacOSのインストール/アップデートプロセスが非常に非効率だからです。将来的には、独自のセットアッププロセスができれば不要になるでしょう。

OSのAPFSパーティションの縮小はオンラインで行うことができます。

`diskutil addPartition`を起動するたびに作成する新しいパーティションの*前*のパーティションノードを渡す必要があります。
これは変更される場合があるので各インスタンスの前に `diskutil list` を実行して最後に作成されたパーティションのデバイス名を選びます。
最後に作成されたパーティションのデバイス名を選びます。上の例では、`Linux`はdisk0s5に、`LB`はdisk0s4になりました。
はdisk0s4となりました。自分で判断してください。これは再起動するとリセットされます。

スタブパーティションのボリューム名はブートピッカーでのOS名を決定するので『Linux』と名づけてください。

真のLinuxルートパーティションとブートパーティションのFSとGPT種類は後で変更できます。

### ステップ2：スタブパーティションにmacOSをインストール

マシンをシャットダウンします。電源ボタンを押したまま『Loading startup options...』と表示されるまで起動します。これで OS のリカバリー環境である 1TR に入ります。

ブートピッカーで「オプション」を選択します。認証情報の入力を求められることがあります。
これはメインのmacOSインストールのログイン認証情報です。

メインのリカバリーメニューで『Reinstall macOS Big Sur』を選択します。

画面の指示に従い、ターゲットボリュームとして『Linux』を選択します。

しばらく待ちます。これでマシンが自動的に新しいOSに再起動されます。

### ステップ3：初期設定

OS の初期設定を行います。管理者ユーザーとパスワードを作成します。

OSのバージョンを確認します。11.2でない場合は少なくとも11.2にアップデートしてください。

再度マシンをシャットダウンします。

### ステップ4: セキュリティのダウングレード

1TR を再度起動します（ブートピッカーの『Options』の項目）。

上部メニューバーの『Utilities』→『Terminal』を選択します。これによりルートシェルが表示されます。

スタブパーティションのボリュームグループIDを探します:

```shell
# diskutil apfs listVolumeGroups | grep -B3 Linux | grep Group
```

次にセキュリティをダウングレードします:

```shell
# bputil -nkcas
```

該当するOSのインストールIDを選択して画面の指示に従います。ステップ3で作成した認証情報を入力する必要があります。
さらに念のためSIPを無効にします。

こちらはOSの名前を聞いてきます:

```shell
# csrutil disable
```

注意: 『ローカルポリシーの作成に失敗しました』というような奇妙なパーミッションエラーが出る場合は、次の方法でシステムのセキュリティポリシーをリセットする必要があるかもしれません。

```shell
# csrutil clear
```
これでカスタムカーネルを使用する準備が整いました。おめでとうございます！

## シリアルコンソールの取得

シリアルコンソールは迅速な開発サイクルとローレベルの問題をデバッグするために不可欠です。

M1 MacではType-Cポート（DFUに使用されるものと同じ）の1つがデバッグ用のシリアルポートを公開しています。MacBookの場合は、背面左ポートです。
また、Mac Miniでは、電源プラグに最も近いポートになります。

また、迅速なテストサイクルの実現のために、USB-PD VDMコマンドを使ってターゲットマシンをハードリブートすることもできます(電源ボタンを押しっぱなしにしなくてすみます)。

USB-PD VDMコマンドの詳細と何ができるかについては[USB-PD](../hw/soc/usb-pd.md)を参照してください。

シリアルポートは1.2Vのロジックレベルを使用するUARTで、有効にするにはベンダー固有のUSB-PD VDMコマンドが必要です。したがって、互換性のあるケーブルを作ることは容易ではありません。次のような選択肢があります:

### M1マシンを使用

M1マシンを2台お持ちの場合はこの方法が最も簡単です。[macvdmtool](https://github.com/AsahiLinux/macvdmtool/)を起動し*両方*のマシンのDFUポートを使って標準のType C SuperSpeedケーブルで接続すればOKです。

**重要な注意事項：** ケーブルはUSB 3 / SuperSpeedタイプである必要があります。USB 2のみのケーブルは動作しませんし、**MacBook/MacBook Airに付属の充電ケーブルも動作しませんし**、その他の安価なケーブルや充電用ケーブルも動作しません。パッケージに 『SuperSpeed』や 『USB3.0』と書かれていなければほぼ確実に動作しません。細くて曲がっているものはほぼ間違いなくSuperSpeedケーブル*ではありません*。動作するケーブルには内部に最大**15**本のワイヤーが必要です。内部にそれだけのワイヤーがあると思えない場合は求めているケーブルではありません。もしあなたのケーブルがSuperSpeedに対応していると確信していて（つまりデバイスがそのケーブルを通してSuperSpeedとして列挙され）それでも動作しない場合は、そのケーブルは非準拠であり、メーカーはAmazonのレビューページで恥をかかされるに値します。なぜならばSBU1/2ピンに接続していないことを意味するからです。


```shell
$ xcode-select --install
$ git clone https://github.com/AsahiLinux/macvdmtool
$ cd macvdmtool
$ make
$ sudo ./macvdmtool reboot serial
```

このように表示されるはずです:

```
Mac type: J313AP
Looking for HPM devices...
Found: IOService:/AppleARMPE/arm-io@10F00000/AppleT810xIO/i2c0@35010000/AppleS5L8940XI2CController/hpmBusManager@6B/AppleHPMBusController/hpm0/AppleHPMARM
Connection: Sink
Status: APP 
Unlocking... OK
Entering DBMa mode... Status: DBMa
Rebooting target into normal mode... OK
Waiting for connection........ Connected
Putting target into serial mode... OK
Putting local end into serial mode... OK
Exiting DBMa mode... OK
```

対応コマンド一覧については、macvdmtoolのページを参照するか、引数なしで実行してみてください。シリアルポートのデバイスは 『/dev/cu.debug-console』 です。picocom を試すには次のようにします: 『sudo picocom -q --omap crlf --imap lfcrlf -b 115200 /dev/tty.debug-console』

注意: ホストマシンでシリアルデバッグ出力を有効にしている場合 (nvram の設定経由) は、シリアルポートデバイスはカーネルに使用されます。一般的なシリアルポートとして使用するにはこれをオフにする必要があります。

### DIY Arduino ベースの USB-PD インターフェイス

以下のパーツでDIYのUSB-PDインターフェイスを作ることができます: 

* Arduino またはそのクローン
* FUSB302チップ、単体またはブレイクアウトボードに搭載
* USB Type C ブレークアウトボード（最も柔軟性を高めるために、ターゲットに直接差し込むオス型を用意)
* 1.2V互換のUARTインターフェース

ほとんどの FUSB302 ブレークアウト・ボードは、必要な Type C ピンをうまく切り出せないので、別の完全なブレークアウト・ボードを使用することをお勧めします。

詳しい情報や配線リストは [vdmtool](https://github.com/AsahiLinux/vdmtool) リポジトリをご覧ください。今のところ文書は少し少ないです。ヘルプが必要な場合は、IRC (OFTC/#asahi) で私たちに問い合わせ可能です。

1.2V 互換の UART インターフェースは比較的まれです。1.8Vのものは通常入力（RX）で動作します。TXの電圧を下げるために抵抗分割器を使用することができます
(`TX -- 470Ω -+- 220Ω -- GND` `+`点で1.8VのTXを1.22Vに下げる)。

デフォルトの『vdmtool』コードはSBU1/SBU2ピンにシリアルを入れます。デバイス側のコネクタ（ケーブルなし）では、TX（Macからの出力）はアクティブなCCラインを持つコネクタ側のSBU1ピンになり（1つだけ接続する必要あり）、RX（Macへの入力）はCCラインの反対側になります。

これはかなり原始的なもので、適切な解決策を得るための一時的なものです。

### 柔軟な USB-PD デバッグインターフェース (プロジェクト名未定)

今後数週間内にM1シリアルポートに接続するためのオープンハードウェアインターフェースを設計します(Apple機器の他のデバック用ピンセットやある種のAndroidフォンなど他の機器のUARTにも対応)。今後の情報にご期待ください。初期のプロトタイプを入手したい既存のカーネル開発者は[marcan氏](mailto:marcan@marcan.st)までご連絡ください。

### 標準的なUSBケーブルを使ったUSBガジェットモード

m1n1 は、標準的な USB [CDC-ACM](https://en.wikipedia.org/wiki/USB_communications_device_class)デバイスを介して、 デバッグコンソールとプロキシインターフェースを公開することに対応しました。必要なのは標準的なUSBケーブル（ホストマシンに合わせて、C to CまたはA to C）だけです。![USB Type-C to Type A ケーブル](https://raw.githubusercontent.com/amworsley/asahi-wiki/main/images/USB-TypeC-A-cable.png)

このインターフェースはシリアルポートよりもはるかに高速で、m1n1をリモートで使用する際には好ましい方法です。しかし、低レベルのデバッグや開発のためには、これに加えてシリアルコンソールを使用することが推奨されます。

m1n1を起動すると、2つのCDC-ACMデバイスが表示されます（Linuxでは『/dev/ttyACM0』と『/dev/ttyACM1』、macOSでは『/dev/tty.usbmodemP_01』と『/dev/tty.usbmodemP_03』のように表示）。

```
[977820.091168] usb 2-2: USB disconnect, device number 95
[977820.832569] usb 2-2: new high-speed USB device number 96 using xhci_hcd
[977821.069287] usb 2-2: New USB device found, idVendor=1209, idProduct=316d, bcdDevice= 1.00
[977821.069296] usb 2-2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[977821.069300] usb 2-2: Product: m1n1 uartproxy da44067
[977821.069303] usb 2-2: Manufacturer: Asahi Linux
[977821.069305] usb 2-2: SerialNumber: P-0
[977821.070574] cdc_acm 2-2:1.0: ttyACM0: USB ACM device
[977821.070825] probe of 2-2:1.0 returned 1 after 323 usecs
[977821.071036] cdc_acm 2-2:1.2: ttyACM1: USB ACM device
[977821.071099] probe of 0000:00:02.0 returned 0 after 19 usecs
[977821.071262] probe of 2-2:1.2 returned 1 after 284 usecs
[977821.071375] probe of 2-2 returned 1 after 1838 usecs
[977821.071428] probe of 0000:00:02.0 returned 0 after 9 usecs
```

pythonのproxyclientコマンド 『proxyclient/tools/{shell,chainload,linux}.py』 を以下のように実行する際には、USB ACMのシリアルデバイスを指定しなければなりません（注意: pythonのバージョンとインストール要件については以下を参照）。
例： 『shell.py』 の場合： 
```shell
$ M1N1DEVICE=/dev/ttyACM0
$ export M1N1DEVICE
$ python3 proxyclient/tools/shell.py
``` 

    * 2番目のttyでターミナルプログラムを起動でき、linuxのコンソール出力がそこに行くようにできる

```
picocom /dev/ttyACM1
```

なお、この方法は Linux のearlyconとしては(まだ)使えませんし、USB ガジェットのサポートもまだ Linux のメインツリーにはありません。

* 詳細は [USBケーブルでLinuxを実行](../sw/linux-bringup.md#USBケーブルでLinuxを実行) を参照

## m1n1の使用

m1n1 は我々の最初のブートローダで、XNU カーネルのふりをして Apple 固有の初期化を行う役割を担っています。

現在、m1n1はシリアル 『プロキシ』サーバとして動作しており、ホストからPythonスクリプトで制御されます。この方法により、Python コンソールを使ってシリアルや USB 経由でカーネルをロードしたりハードウェアを対話的に探索することができます。

### Debian のビルドに必要なもの

```shell
$ sudo apt install -y gcc-arch64-linux-gnu libc6-dev-arm64-cross device-tree-compiler imagemagick
```

pythonのプロキシクライアントを実行するには、デフォルトよりも新しいバージョンのpython3（3.9）が必要です。

* 最新の安定版 (3.9.5) [2021/05/16] をダウンロードしてビルド[Building latest python on Debian 10](https://linuxize.com/post/how-to-install-python-3-9-on-debian-10/)
* 一般ユーザとして必要な各種パッケージをインストール

```shell
python3.9 -m pip install pyserial construct serial.tool
```

### ビルド

『aarch64-linux-gnu-gcc』のクロスコンパイラツールチェイン（またはARM64ならネイティブのもの）が必要です。また、ブートロゴ用に『dtc』 (devicetree コンパイラ) と 『convert』 (ImageMagick のもの) が必要です。

```shell
$ git clone --recursive https://github.com/AsahiLinux/m1n1.git
$ cd m1n1
$ make
```

出力は 『build/m1n1.macho』 になります。

訳者追記(2021/12/25): macOS 12.1以降は同時にビルドされるであろう『build/m1n1.bin』を使ってください。
https://github.com/AsahiLinux/m1n1 参照

### インストール

Mac を 1TR で起動し、ターミナルを立ち上げます。ネットワークを使ってファイルをコピーするには、Mac から SSH を使うかcurl を使ってください。

例えばビルドマシンでは:

```shell
$ cd m1n1/build
$ python3 -m http.server
```

ターゲットマシンでは:

```shell
# curl buildmachine.lan:8000/m1n1.macho > m1n1.macho
```

m1n1をカスタムkernelcacheとしてスタブパーティションにインストール:

```shell
# kmutil configure-boot -c m1n1.machao -C -v /Volumes/Linux
```

認証情報を入力します。全てがうまくいけば、再起動してm1n1が起動します！

訳者追記(2021/12/25): macOS 12.1以降は上記の代わりに次のコマンドとなるはずです。
```
# kmutil configure-boot -c m1n1.bin --raw --entry-point 2048 --lowest-virtual-address 0 -v /Volumes/Linux
```
https://github.com/AsahiLinux/m1n1 参照

### ペイロードの直接起動

m1n1 はバイナリに直接埋め込まれたペイロードの実行をサポートしています。これにより、シリアル通信を必要とせずに Linux (または後続のブートローダステージ) を起動することができます。現在、デバッグ出力はシリアルのみですが、フレームバッファデバッグコンソールへの対応はまもなくです。

m1n1とペイロードを結合するにはそのまま連結してください:

```shell
$ cat build/m1n1.macho Image.gz build/dtb/apple-j274.dtb initramfs.cpio.gz > m1n1-payload.macho
```
注意：現在M1N1は、Hwから読み込んだモデルとDeviceTree(dtbファイル)にあるモデルをチェックし、一致しない場合はペイロードを実行しないようになりました。
src/payload.cのpayload_runを参照してください。j274は、MacMini Hwモデルを表します。

累積ペイロードの最大サイズは現在64 MiBです。これ以上必要な場合は、『m1n1.ld』 の先頭で変更することができます。また、m1n1をチェーンロードすると、現在のところこのメモリが『無駄』になります（次のイテレーションはこの領域より上にロードされ、『領域の下』のメモリを使用できません）。

対応するペイロードのファイル形式:

* Linux ARM64 カーネルイメージ（または互換性のあるもの）、圧縮するか最後のペイロードにする必要あり
* Devicetree blob (FDT) 非圧縮のままでも圧縮しても可
* Initramfs の cpio イメージ　圧縮する必要あり

対応する圧縮形式:

* gzip
* xz

### ホストの設定

現時点ではm1n1をインタラクティブに使用したりデバッグ情報を取得したりするにはシリアルコンソールが必要です。[シリアルコンソールの取得](#シリアルコンソールの取得)の項を参照してください。

**注意(2021/02/06)**: 近日中にフレームバッファコンソールを追加し、シリアルなしでのデバッグを容易にする予定です。USB OTGのサポートも将来的に計画されており、標準的なUSB Cケーブルでm1n1と対話し、カーネルをより速くロードできるようになります。

m1n1 proxyclientを使用するには、Pythonパッケージの『pyserial』と『construct』が必要です。ディストリビューションがこれらを提供していない場合は、以下を使用することができます:

```shell
$ pip install --user pyserial construct
```

macOSをお使いの方は、Homebrewを使ってpython3.9をインストールしてください。OS版のPythonはreadlineのヒストリーファイルに問題があります(?): 

```shell
$ arch -x86_64 brew install python
$ pip3.9 install --user pyserial construct
```

macportsを使ってpythonとpipをインストールし、rootとしてpipを使って依存関係をインストールすることもできます。MacportsにはM1用のネイティブビルドが用意されています。

```shell
$ sudo su -
$ port install python39 py39-pip
$ pip install pyserial construct
```

### 使用方法

proxyclientスクリプトを使用するには、envarの『M1N1DEVICE』にシリアルポートのデバイスを設定する必要があります。他のM1 macをホストとして使用する場合は『export M1N1DEVICE=/dev/cu.debug-console』としてください。USBで接続するのがLinuxホストの場合は『export M1N1DEVICE=/dev/ttyACM0』のようにしてください。

可能なこと:

#### プレイグラウンドのシェル
『python shell.py』を実行すると、インタラクティブなデバッグシェルができます。


```python
>>> read32(0x800000000) # RAMの最初のワードを読む
0x484f6666
>>> memset32(fb, 0x000003ff, 1920 * 1080 * 4) # フレームバッファを青くする
>>> mrs(HCR_EL2) # 一部のレジスタのみ対応、utils.pyで自由に追加してください
0x30488000000
>>> d = readmem(base, 0x10) # m1n1 (mach-o header)の最初の16バイトを読みhexdump
>>> chexdump(d)
00000000  cf fa ed fe 0c 00 00 01  02 00 00 00 0c 00 00 00  |................|
# バッファを確保して、AICのレジスタを読み込んで、ダンプ
# バッファを使用することを推奨、なぜなら 『readmem』 はバイトごとに 
# 2 回読み込んで、データが変わったら文句を言うから (チェックサム)
>>> AIC = 0x23b100000
>>> data = malloc(0x8000) # this is python-side malloc, allocs vanish when the script exits
>>> memcpy32(data, AIC, 0x8000)
>>> chexdump32(d)
00000000  00000007 000a0380 00180018 00000000 e0477971 00000000 000000f0 00000000
00000020  00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
00000040  *
00003320  00000000 00000000 00000000 00000001 00000000 00000000 00000000 00000000
[...]
>>> u.msr(DAIF, u.mrs(DAIF) & ~0x3c0) # Enable IRQs             
>>> mon.add(AIC, 0x8000) # Monitor the AIC registers
[...]
>>> write32(AIC + 0x2008, 1) # Fire off an IPI to ourselves
TTY> Exception: IRQ          # These debug messages are from m1n1's IRQ handler
TTY>  type: 4 num: 1
# こうすると、ここでは表示できない色分けでレジスタモニタが自動的に
# 変更されたレジスタを^表示します。
000000023b100020 00000000 00000000 00000000 00000000 00000001 00000000 00000000 00000000 
000000023b102000 00000000 00000000 00000001 00000001 00000000 00000000 00000000 00000000 
000000023b102020 00000000 00000001 00000001 00000000 00000000 00000000 00000000 00000000 
000000023b105000 00000000 00000000 00000001 00000001 00000000 00000000 00000000 00000000 
[...]
# 単純な障害からの回復に対応
>>> read64(0)
TTY> Exception: SYNC
TTY> Running in EL2
TTY> MPIDR: 0x80000000
TTY> Registers: (@0x81368bde0)
TTY>   x0-x3: 0000000000000000 000000081360c004 0000000000000002 000000081360c004
TTY>   x4-x7: 000000081368bfa8 0000000000007a69 000000081360c004 0000000000000000
TTY>  x8-x11: 0000000000000000 0000000000000000 0000000000000000 00000000aaaaaaab
TTY> x12-x15: 000000000000002c 00000008135b5488 000000081368bb80 0000000000000000
TTY> x16-x19: 00000008135aaabc 0000000000000000 0000000000000000 000000081368bf88
TTY> x20-x23: 000000081368bfb0 0000000813608000 0000000000000000 0000000002aa55ff
TTY> x24-x27: 0000000003aa55ff 000000081360c000 0000000001aa55ff 000000081368bfb0
TTY> x28-x30: 0000000000000000 000000081368bee0 00000008135abb98
TTY> PC:       0x8135aaa9c (rel: 0xaa9c)
TTY> SPSEL:    0x1
TTY> SP:       0x81368bee0
TTY> SPSR_EL2: 0x80000009
TTY> FAR_EL2:  0x0
TTY> ESR_EL2:  0x96000018 (data abort (current))
TTY> L2C_ERR_STS: 0x11000ffc00000080
TTY> L2C_ERR_ADR: 0x300000000000000
TTY> L2C_ERR_INF: 0x1
TTY> SYS_E_LSU_ERR_STS: 0x0
TTY> SYS_E_FED_ERR_STS: 0x0
TTY> SYS_E_MMU_ERR_STS: 0x0
TTY> Recovering from exception (ELR=0x8135aaaa0)
0xacce5515abad1dea
```

注意：もし、『proxy.ProxyCommandError: Reply error: Bad Command』というエラーが出た場合、ほとんどの場合
M1上のm1n1をアップグレードする必要があると思われます。これを回避するには以下のように chainload.py を使って最新の m1n1.macho をアップロードするのが手っ取り早い方法です:

#### 別のm1n1のロード

また、『kmutil』を使わずに変更点をテストするのも便利です。

```shell
$ python chainload.py ../build/m1n1.macho 
Base at: 0x803c5c000
FB at: 0x9e0df8000
Loading 442368 bytes to 0x812ce0000
....................................................
Jumping to 0x812ce4800
TTY> m1n1
TTY> sc: Initializing
TTY> CPU init... CPU: M1 Icestorm
[...]
TTY> m1n1 vf14489f
TTY> Copyright (C) 2021 The Asahi Linux Contributors
TTY> Licensed under the MIT license
[...]
Proxy is alive again
```

#### Linux カーネルを起動

このためにここに来たんですよね？ :-)。詳しい手順は [Linux Bringup](../sw/linux-bringup.md) を参照してください。

```shell
$ python linux.py -b 'earlycon console=ttySAC0,1500000 console=tty0 debug' Image.gz apple-j274.dtb initramfs.cpio.gz
```

m1n1との通信にUSBを使用していますが、Linuxにシリアルコンソールを使用したい場合は『--tty』という引数を使用します。例えば、m1n1に『/dev/ttyACM0』、Linuxに『/dev/ttyUSB0』を使うには次のようにします:

```shell
$ export M1N1DEVICE=/dev/ttyACM0
$ python linux.py --tty /dev/ttyUSB0 -b 'earlycon console=ttySAC0,1500000 console=tty0 debug' Image.gz apple-j274.dtb initramfs.cpio.gz
```

<details>
 <summary>sample boot log</summary>

```
Base at: 0x815300000
FB at: 0x9e0df8000
Loading 2046483 bytes to 0x8214fc000..0x8216efa13...
..........................................................................................................................................................................................................................................................
Loading DTB to 0x8216efa40...
Kernel_base: 0x821800000
Loading 953623 initramfs bytes to 0x821700000...
.....................................................................................................................
TTY> Starting secondary CPUs...
TTY> Starting CPU 1 (0:1)... OK
TTY>   Stack base: 0x8153ac040
TTY>   MPIDR: 0x80000001
TTY>   CPU: M1 Icestorm
TTY>   Index: 1 (table: 0x8153c4080)
TTY> 
TTY> Starting CPU 2 (0:2)... OK
TTY>   Stack base: 0x8153b0040
TTY>   MPIDR: 0x80000002
TTY>   CPU: M1 Icestorm
TTY>   Index: 2 (table: 0x8153c40c0)
TTY> 
TTY> Starting CPU 3 (0:3)... OK
TTY>   Stack base: 0x8153b4040
TTY>   MPIDR: 0x80000003
TTY>   CPU: M1 Icestorm
TTY>   Index: 3 (table: 0x8153c4100)
TTY> 
TTY> Starting CPU 4 (1:0)... OK
TTY>   Stack base: 0x8153b8040
TTY>   MPIDR: 0x80010100
TTY>   CPU: M1 Firestorm
TTY>   Index: 4 (table: 0x8153c4140)
TTY> 
TTY> Starting CPU 5 (1:1)... OK
TTY>   Stack base: 0x8153bc040
TTY>   MPIDR: 0x80010101
TTY>   CPU: M1 Firestorm
TTY>   Index: 5 (table: 0x8153c4180)
TTY> 
TTY> Starting CPU 6 (1:2)... OK
TTY>   Stack base: 0x8153c0040
TTY>   MPIDR: 0x80010102
TTY>   CPU: M1 Firestorm
TTY>   Index: 6 (table: 0x8153c41c0)
TTY> 
TTY> Starting CPU 7 (1:3)... OK
TTY>   Stack base: 0x8153c4040
TTY>   MPIDR: 0x80010103
TTY>   CPU: M1 Firestorm
TTY>   Index: 7 (table: 0x8153c4200)
TTY> 
TTY> FDT: initrd at 0x821700000 size 0xe8d17
TTY> FDT: framebuffer@9e0df8000 base 0x9e0df8000 size 0x7e9000
TTY> FDT: DRAM at 0x800000000 size 0x200000000
TTY> FDT: Usable memory is 0x80134c000..0x9db5e0000 (0x1da294000)
TTY> FDT: CPU 0 MPIDR=0x0 release-addr=0x8153c4050
TTY> FDT: CPU 1 MPIDR=0x1 release-addr=0x8153c4090
TTY> FDT: CPU 2 MPIDR=0x2 release-addr=0x8153c40d0
TTY> FDT: CPU 3 MPIDR=0x3 release-addr=0x8153c4110
TTY> FDT: CPU 4 MPIDR=0x10100 release-addr=0x8153c4150
TTY> FDT: CPU 5 MPIDR=0x10101 release-addr=0x8153c4190
TTY> FDT: CPU 6 MPIDR=0x10102 release-addr=0x8153c41d0
TTY> FDT: CPU 7 MPIDR=0x10103 release-addr=0x8153c4210
TTY> FDT prepared at 0x8193f0000
Uncompressing...
5758984
Decompress OK...
Ready to boot
DAIF: 3c0
MMU: shutting down...
MMU: shutdown successful, clearing caches
Booting kernel at 0x821800000 with fdt at 0x8193f0000
[    0.000000] Booting Linux on physical CPU 0x0000000000 [0x611f0221]
[    0.000000] Linux version 5.11.0-rc6-00015-gdc1ce163506d (marcan@raider) (aarch64-linux-gnu-gcc (Gentoo 10.2.0-r5 p6) 10.2.0, GNU ld (Gentoo 2.35.1 p2) 2.35.1) #201 SMP PREEMPT Fri Feb 5 03:15:40 JST 2021
[    0.000000] Machine model: Apple Mac Mini M1 2020
[    0.000000] earlycon: s5l0 at MMIO32 0x0000000235200000 (options '1500000')
[    0.000000] printk: bootconsole [s5l0] enabled
[    0.000000] Zone ranges:
[    0.000000]   DMA      [mem 0x000000080134c000-0x00000009db5dffff]
[    0.000000]   DMA32    empty
[    0.000000]   Normal   empty
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x000000080134c000-0x00000009db5dffff]
[    0.000000] Zeroed struct page in unavailable ranges: 648 pages
[    0.000000] Initmem setup node 0 [mem 0x000000080134c000-0x00000009db5dffff]
[    0.000000] On node 0 totalpages: 485541
[    0.000000]   DMA zone: 1897 pages used for memmap
[    0.000000]   DMA zone: 0 pages reserved
[    0.000000]   DMA zone: 485541 pages, LIFO batch:15
[    0.000000] percpu: Embedded 6 pages/cpu s57888 r0 d40416 u98304
[    0.000000] pcpu-alloc: s57888 r0 d40416 u98304 alloc=6*16384
[    0.000000] pcpu-alloc: [0] 0 [0] 1 [0] 2 [0] 3 [0] 4 [0] 5 [0] 6 [0] 7 
[    0.000000] Detected VIPT I-cache on CPU0
[    0.000000] CPU features: detected: Virtualization Host Extensions
[    0.000000] CPU features: kernel page table isolation disabled by kernel configuration
[    0.000000] CPU features: detected: Spectre-v4
[    0.000000] CPU features: detected: Address authentication (IMP DEF algorithm)
[    0.000000] CPU features: detected: FIQs
[    0.000000] alternatives: patching kernel code
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 483644
[    0.000000] Kernel command line: earlycon debug
[    0.000000] Dentry cache hash table entries: 1048576 (order: 9, 8388608 bytes, linear)
[    0.000000] Inode-cache hash table entries: 524288 (order: 8, 4194304 bytes, linear)
[    0.000000] mem auto-init: stack:off, heap alloc:off, heap free:off
[    0.000000] Memory: 7715584K/7768656K available (3136K kernel code, 1144K rwdata, 640K rodata, 512K init, 455K bss, 53072K reserved, 0K cma-reserved)
[    0.000000] random: get_random_u64 called from __kmem_cache_create+0x30/0x4b4 with crng_init=0
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=8, Nodes=1
[    0.000000] rcu: Preemptible hierarchical RCU implementation.
[    0.000000] rcu:     RCU restricting CPUs from NR_CPUS=16 to nr_cpu_ids=8.
[    0.000000]  Trampoline variant of Tasks RCU enabled.
[    0.000000] rcu: RCU calculated value of scheduler-enlistment delay is 100 jiffies.
[    0.000000] rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=8
[    0.000000] NR_IRQS: 64, nr_irqs: 64, preallocated irqs: 0
[    0.000000] aic_of_ic_init: AIC: initialized with 896 IRQs, 2 FIQs, 2 IPIs, 32 vIPIs
[    0.000000] arch_timer: cp15 timer(s) running at 24.00MHz (phys).
[    0.000000] clocksource: arch_sys_counter: mask: 0xffffffffffffff max_cycles: 0x588fe9dc0, max_idle_ns: 440795202592 ns
[    0.000000] sched_clock: 56 bits at 24MHz, resolution 41ns, wraps every 4398046511097ns
[    0.000650] Console: colour dummy device 80x25
[    0.000958] printk: console [tty0] enabled
[    0.001273] printk: bootconsole [s5l0] disabled
BusyBox v1.30.1 (Debian 1:1.30.1-6+b1) built-in shell (ash)
Enter 'help' for a list of built-in commands.
/bin/sh: can't access tty; job control turned off
/ #
```

</details>

スクリプトは現在TTYパススルーの入力と出力をサポートしています。
