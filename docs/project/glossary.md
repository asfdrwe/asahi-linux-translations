---
title: 用語集
---

2025/3/31時点の[glossary](https://github.com/AsahiLinux/docs/blob/main/docs/project/glossary.md)の翻訳

---
知っておくと役立つ用語

ハードウェアやリバースエンジニアリングやアップルのエコシステムに特有の用語で、関連性のあるものを追加します。誰もが知っていると思われるもの（CPU、GPU、HDD、SSD、OSX、USB、HDMI、RAMなど）やこのプロジェクトには関係なさそうなものは加えないでください。対象者は『ランダムなLinux開発者』と考えてください。

副分野(GPUなど)に特化した用語を大量に集めたい場合は自由に別のページを作成してください。

### 1
* **1TR**: One True RecoveryOS。電源ボタン長押しで起動するRecoveryOSのことをこう呼ぶ。これは、物理的な存在を主張し
Appleが完全に信頼するリカバリ環境を実行してカスタムOSをインストールできるなど特別な権限が与えられることを意味する。
rootアクセスが可能だがAppleの署名入りのソフトウェアのみ実行可能。FileVaultが有効になっている場合は最初に認証が必要

### A
* **AGX**: Apple の GPU シリーズの内部名称
* **AIC**: Apple Interrupt Controller。標準のGICではAppleにとって標準的すぎるためAppleがカスタムしたARM割り込みコントローラ
* **AMX**: Apple Matrix eXtensions(Apple行列拡張)。 ISAに部分的に統合された行列コプロセッサ
* **ANE**: Apple Neural Engine (Appleニューラルエンジン)。 FP16 積和装置
* **ANS**: NVME/ストレージコプロセッサ？
* **AOP**: Always On Processor。 macOSで『Hey Siri』などを有効にするAppleのSoC コプロセッサ/DSP
* **AP**: Application Processor。OSの大部分を実行するメインCPU。SEPとは対照的
* **APFS**。Apple File System（アップルファイルシステム）。ZFSやbtrfsのように、Appleが開発したコンテナやボリュームを重視した「モダン」なファイルシステム
* **APFSコンテナ**: ディスク上の物理的なパーティションで、それ自体に複数のファイルシステム（ボリューム）を格納することができ、すべてのファイルシステムが動的に領域を共有
* **APFSスナップショット**: APFSボリュームの読み取り専用のコピーオンライトのスナップショット
* **APFSボリューム**: APFSコンテナ内の論理ファイルシステムでディレクトリにマウント可能
* **APR**: APR ProRes。 ProResビデオエンコーディング＋デコーディング制御
* **APSC**: Automatic Power State Controller。自動電源状態制御装置
* **ASC**: * **ASC**: コプロセッサの総称？例：gfx-asc。おそらくApple Silicon Silicon Coprocessor
* **AVD**: Apple Video Decoder(Appleビデオデコーダ)
* **AVE**: Apple Video Encoder(Appleビデオエンコーダ)

### B
* **BootROM**:  M1などのチップに内蔵された読み取り専用のメモリで、起動時に最初に実行されるコード。SecureROMを参照

### C
* **Chicken Bits**:  別名『kill bits』とも呼ばれ、特定の機能を無効にしたり有効にしたりするための設定ビット

### D
* **DART**: Device Address Resolution Table。Apple のカスタム IOMMU
* **DCP**: Display Control Processor (おそらく)。ティアリングのない新しいフレームの表示、マウスカーソルなどのハードウェアスプライト、解像度の切り替え、複数の出力の設定などに対応
* **DFR**: Dynamic Function Row。Touch BarのAppleの内部名称
* **DFU**: Device Firmware Update。USB経由でデバイスのファームウェアを更新することができるUSBモード。Apple のデバイスは SecureROM でこの機能に対応しており、文鎮化されたデバイスを復元可能にする
* **DPE**: Digital Power Estimator。デジタル電力推定器
* **DVFM**: Dynamic Voltage and Frequency Management。動的電圧周波数管理

### E
* **EEPROM**：Electrically Erasable Programmable Read Only Memory。 再書き込み可能なメモリの一種で、一般的には最大でも数キロバイトのサイズ。NORフラッシュよりも堅牢。設定やブートコードの最初の部分に使用

### F
* **Fallback Recovery OS**: 電源ボタンをダブルクリックしたまま起動することでアクセスできるリカバリーOSの2つ目のコピー。1TRとは異なり、セキュリティ状態(設定)を変更することは不可能。リカバリ OS 1TR とは、ユーティリティーの『Start Security Utility』オプションがないことで区別可能
* **fuOS**：カスタムOS、『fully untrusted OS(完全に信頼されていないOS)』という意味と推測

### G
* **GPT**: GUID Partition Table。EFI/UEFI 用に作成されたパーティションテーブルフォーマットで、現在ほとんどの最新システムで使用される
* **GXF**: 恐らくGuarded Execution Function。ページテーブルや同様に重要な構造体をXNU自身から保護するために、低オーバーヘッドのハイパーバイザを作成するために使用される横方向の例外レベル。[Sven's writ-up](https://blog.svenpeter.dev/posts/m1_sprr_gxf/)や[SPRR と GXF](../hw/cpu/sprr-gxf.md)などを参照

### H
* **HFS+**: Hierarchical Filesystem+ (階層型ファイルシステム) 。Apple の従来のファイルシステムで外部ストレージに使用。M1 Macの内部ストレージでは未使用

### I
* **I²C**: Inter-Integrated Circuit。基板上のチップ間を低速で通信するための2線式の規格
* **iBEC**: iBoot Epoch Change。DFUブートフローでロードされる2段目のiBootの代替品
* **iBoot**: Apple のブートローダ。OSのPrebootパーティションにある特定のセカンドステージローダ（しばしばiBoot2と呼ばれる）、
またはLLB、iBSS、iBEC、SecureROM（これらはすべて異なる機能を持つiBootのビルド）のいずれかを指す。iBoot1 は LLB の新しい名称
OS の Preboot パーティションにある特定のセカンドステージローダーまたはLLB、iBSS、iBECを指す
* **iBoot**: Apple のブートローダ。iBoot1やiBoot2、iBSS、iBECを指したり、SecureROMを指す場合あり(すべてiBootの異なるビルドで、機能も異なる）
* **iBoot1**: NORに位置する第1段のiBootで、SecureROMによってロード。初期化の最初にOSに依存しないファームウェアのロードを行った後、OSプレブートパーティションにある第2ステージiBoot（iBoot2）をチェーンロード。LLBはiBoot1の古い名称
* **iBoot2**: OSのPrebootパーティションにある第2ステージiBoot。インストールされるOSごとに固有のバージョンでありで、OSが動作するために必要なランタイムファームウェアのバンドルと一緒にパッケージ化
* **iBSS**: iBoot Single Stage。ファーストステージのiBoot(iBoot1/LLB)の代替品で、NORが破損している場合にDFUブートフローでロード
* **IOKit**: I/O KitはAppleのXNU(AppleのOSカーネル)用のデバイスドライバフレームワーク
* **IOMMU**: I/O Memory Management Unit。アップル社のDARTの総称
* **IPI**: Inter-processor interupt。あるプロセッサが別のプロセッサに割り込みする際の割り込み
* **iSC**: iBoot System Container。システム全体のブートデータを格納するディスクパーティション(通常内蔵SSDの最初のパーティション)([既製品パーティションレイアウト](../platform/stock-partition-layout.md)を参照）
* **ISP**： Image Signal Processor。Mシリーズのラップトップのウェブカメラ。センサーからストロボやコプロセッサまでカメラユニット全体を表す

### J
* **JTAG**: Joint Test Action Group。実際には同グループが発表した、チップやCPUをハードウェアレベルでデバッグするための4/5線式のデバッグインターフェース

### K
* **kASLR**: kernel Address Space Loacation Randomization。Linux カーネルの機能で起動時にカーネルコードをメモリ上のどこに配置するかをランダム化するもの。ブートフラグに 『nokaslr』 を指定すると無効
* **kcOS**：カスタムカーネルキャッシュを持つOS
* **Kernel cache**:カーネルとその拡張機能のバンドルでオプションで暗号化も可能
* **kmutil**: macOSのカーネル拡張(kexts)を管理するカーネルユーティリティ。m1n1などの代替カーネルの起動に使用

### L
* **LLB**: Low Level Bootloader。iOSプラットフォーム由来のiBoot1の旧名称

### M
* **Mini**: 内部調査用のカスタムブートローダ。SSDからの起動に対応する場合と非対応の場合あり。このプロジェクトではM1N1と呼ばれる派生を使用
* **Mux**: Multiplexer。USB、UART、SWDの各モードを切り替えるピンのセットなど複数のものを1つの接続で接続できるデバイス

### N
* **NAND**:Not-AND。論理ゲートの一種だが通常はフラッシュメモリの一種を指す。SDカードやSSDなど最近の大容量フラッシュベースのストレージに使われているが、生のチップもあり
* **NOR**：Not-OR。論理ゲートの一種だが、通常はフラッシュメモリーの一種を指す。低容量のアプリケーション（最大でも数メガバイトまで）にのみ使用。NANDよりも堅牢。最近は8ピンのベアチップが主流
* **NVRAM**: Non-Volatile RAM。この名称は時代遅れ。Macのブート設定用に保存されたキーと値のパラメータのリストを意味する。UEFIの変数に類似

### P
* **PMGR**: Power Manager(電源管理)
* **PMP**: Power Management Processor(電源管理プロセッサ)

### R
* **RecoveryOS**: リカバリ環境。OSインストールとペアになるリカバリイメージ（APFSサブボリューム内に配置）またはディスク上の最後のAPFSコンテナに
インストールされたグローバルリカバリイメージ。macOS 11.x はデフォルトでグローバルイメージを使用し、macOS 12.0 以降はペアリングされた 
recoveryOS を使用
* **RestoreOS**: 復元環境。Apple ConfiguratorによるDFUモードでの『revive』の際にデバイスに読み込まれる。[詳細](https://www.theiphonewiki.com/wiki/Restore_Ramdisk)
* **ROM**: Read-Only Memory の頭文字。永久または半永久的なデータを格納したコンピュータのメモリチップ
* **RTKit**: Appleが独自に開発したリアルタイム・オペレーティング・システム。ほとんどのアクセラレータ（AGX, ANE, AOP, DCP, AVE, PMP）は内部のプロセッサでRTKitを実行。『RTKSTACKRTKSTACK』という文字列がRTKitを含むファームウェアの特徴
* **RTOS**: Real-time Operating System

### S
* **SBU**: Sideband Use。Type Cコネクタの2つのピンはランダムなものに使用することができType C標準では未定義
* **SecureROM**: M1のBootROM。NORからiBoot1を読み込んで制御を渡したりDFUモードにフォールバックしたりする役割を担当
* **SEP**: Secure Enclave Processor。M1に内蔵されたHSM/TPM/その他のデバイス。Touch IDやほとんどの暗号、およびブートポリシーを決定。Linuxには無害だが望めばその機能を使うことができる。APと対照的
* **SFR**: システムファームウェアとリカバリ、システムにインストールされたすべてのOSによって共有されるファームウェアとリカバリイメージのコレクション。NOR内のコンポーネント（iBoot1など）、iBootシステムコンテナ、システムリカバリパーティション、外部フラッシュメモリやその他の雑多な場所を含む。SFRは常にバージョンが進み、後戻りすることはなし（フルワイプを除く）
* **SIP**: System Integrity Protection。『rootless』とも呼ばれ、macOSのカーネルはrootであってもいくつかのことを不可能にする
* **SMC**: System Management Controller: 温度センサー、電圧/電力計、バッテリー状態、ファン状態、LCDバックライトと蓋スイッチなどへのアクセスを制御
するハードウェア。[SMC](../hw/soc/smc.md)を参照
* **SOP**: Start Of Packet。USB-PDでパケットの種類を区別するために使用。SOPは通常の通信用、SOP'とSOP "はケーブルに内蔵されたチップとの通信用、SOP'DEBUGとSOP "DEBUGはApple VDMのようなベンダー固有のカスタム用
* **SPI**: Serial Peripheral Interface。ボード上のチップ間を低速で通信するための4線式規格
* **SPMI**: MIPI Alliance の System Power Management Interface: 2線式双方向インターフェース、マルチマスター (最大 4)、 マルチスレーブ 
(最大 16)、 32KHz ～ 26MHz。[System Power Management Interface](https://en.wikipedia.org/wiki/System_Power_Management_Interface)を参照
* **SPRR**: おそらくShadow Permission Remap Registers。通常のページパーミッション属性(AP,PXN,UXN)を別のテーブルへのインデックスに変更。この新しいテーブルが実際のページパーミッションを決定。また書き込み可能なページと実行可能なページを同時に無効化。[Sven's write-up](https://blog.svenpeter.dev/posts/m1_sprr_gxf/)や[SPRR と GXF](../hw/cpu/sprr-gxf.md)などを参照
* **SWD**: Serial Wire Debug 。ARMコアのデバッグに使用される2ピンのインターフェイスで、JTAGと同様に少ないピンで動作。Appleのデバイスでは使用されているが、市販のデバイスでは、セキュリティ上の制約から（メインのCPU/SoCには）アクセス不可

### T

* **TBT**: Thunderbolt Technology

### U
* **UART**: Universal Asynchronous Receiver Transmitter。シリアルポートの背後のハードウェア
* **USB-PD**:USB Power Delivery。USB Type Cでのサイドバンド通信の規格（古い規格は我々の健全性のため触れない）。ケーブルの種類やコネクタの向きの検出、電源電圧の設定、非USBモードへの切り替えなどに使用
* **USC**: Unified Shader Core。すべてのシェーダタイプ（バーテックス、フラグメント、コンピュート）に対応するシェーダコア。AGXはユニファイドアーキテクチャであるため、単にシェーダーコアのことを指す

### V
* **VBUS**: 電源を供給するUSB端子。デフォルトは5Vだが、USB-PDでは20Vにもなることが可能
* **VDM**: Vendor Defined Message(ベンダー定義のメッセージ)。 USB Alternate Mode (プロプライエタリではない) と USB-PD 上のベンダープロプライエタリのコマンドの両方に使用。AppleはType Cポートの特別なモードを設定するためにこれらを使用
* **VHE**: Virtual Host Extensions。OS/VM/ユーザースペース間の切り替えをより効率的に行うための追加レジスター。[ARM VHE explanation](https://developer.arm.com/documentation/102142/0100/Virtualization-Host-Extensions)を参照

### X
* **XNU**: macOS、iOS、iPadOS、watchOS、tvOSなどに搭載されているAppleのOSカーネル。『XNU』は『X is not Unix』の略語
