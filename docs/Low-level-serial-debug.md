2025/3/1時点の[Low-level-serial-debug](https://github.com/AsahiLinux/docs/blob/main/docs/Low-level-serial-debug.md)の翻訳

---
開発者向けクイックスタートから引用

## シリアルコンソールの取得

シリアルコンソールは迅速な開発サイクルとローレベルの問題をデバッグするために不可欠です。

M1 MacではType-Cポート（DFUに使用されるものと同じ）の1つがデバッグ用のシリアルポートを公開しています。MacBookの場合は、背面左ポートです。
また、Mac Miniでは、電源プラグに最も近いポートになります。

また、迅速なテストサイクルの実現のために、USB-PD VDMコマンドを使ってターゲットマシンをハードリブートすることもできます(電源ボタンを押しっぱなしにしなくてすみます)。

USB-PD VDMコマンドの詳細と何ができるかについては[HW-USB-PD](HW-USB-PD.md)を参照してください。

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
(`TX -- 220Ω -+- 470Ω -- GND` `+`点で1.8VのTXを1.22Vに下げる)。

デフォルトの『vdmtool』コードはSBU1/SBU2ピンにシリアルを入れます。デバイス側のコネクタ（ケーブルなし）では、TX（Macからの出力）はアクティブなCCラインを持つコネクタ側のSBU1ピンになり（1つだけ接続する必要あり）、RX（Macへの入力）はCCラインの反対側になります。

これはかなり原始的なもので、適切な解決策を得るための一時的なものです。

### 柔軟でないUSB-PDデバッグ・インターフェース（別名Central Scrutinizer）

上記のDIYアプローチに代わるものとして、Central Scrutinizerプロジェクトがあります。全く同じように始めたもので、ブレッドボードの代わりに
カスタムPCBを使うだけです。追加機能をサポートするように進化しましたが、コア機能は全く同じです：

![Central Scrutinizer](https://github.com/AsahiLinux/docs/blob/main/docs/assets/central-scrutinizer.jpg)
![Central Scrutinizer (side)](https://github.com/AsahiLinux/docs/blob/main/docs/assets/central-scrutinizer-2.jpg)

主な機能は以下の通りです：
- マイクロコントローラーとしてRaspberryPi Picoを使用（はい、完全にオーバーキルだが、Arduinoより安い！）
- シリアルライン用レベルシフター
- m1n1に供給するためのUSB2.0パススルー
- スタック可能（1台のPicoで**2**個のCentral Scrutinizerボードを駆動可能）
- USB-Cの方向検出（v2+）
- パススルー機能を使用できない代償として、シリアル用のUSB2.0ケーブルを使用可能（v3+）

KiCadプロジェクトは[こちら](https://git.kernel.org/pub/scm/linux/kernel/git/maz/cs-hw.git)から、Pico用の対応ファームウェアは[こちら](https://git.kernel.org/pub/scm/linux/kernel/git/maz/cs-sw.git)から入手できます。このプロジェクトのハードウェア側は、JLCPCBであらかじめ製造用に
設定されているため、ほんの数クリックで製造することができます。それとは別に、Tindieでいくつかのビルド済みボードを見つけることもできますが、
自分でビルドするのが最初の選択になります。

なお、これが信頼できるものであるという保証は**一切ありません**。私（と他何人か）にとっては動作しますが、最終的には**自己責任**となります。

このプロジェクトについてもっと情報が欲しい方は、遠慮なく[maz](mailto:maz@kernel.org)までご連絡ください。

### 柔軟な USB-PD デバッグインターフェース (プロジェクト名未定)

~今後数週間内にM1シリアルポートに接続するためのオープンハードウェアインターフェースを設計します(Apple機器の他のデバック用ピンセットやある種のAndroidフォンなど他の機器のUARTにも対応)。今後の情報にご期待ください。初期のプロトタイプを入手したい既存のカーネル開発者は[marcan](mailto:marcan@marcan.st)までご連絡ください。~

注：これは、m1n1/hypervisorがUSBに対応したことにより、無期限に先送りされ、ほとんど時代遅れになっています。
