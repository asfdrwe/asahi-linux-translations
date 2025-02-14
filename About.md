[About Asahi Linux](https://asahilinux.org/about/)の2025年2月14日時点の非公式日本語訳です。

訳注: プロジェクトページへのリンクは対応する日本語訳へのリンクに置き換え

---
# Asahi Linuxについて
Asahi Linuxは、2020年のM1 Mac Mini、MacBook Air、MacBook Proから始まったApple Silicon MacにLinuxを移植することを目的としたプロジェクトであり、コミュニティです。

私たちの目標は、単にこれらのマシン上でLinuxを動作させるだけでなく、日常的なOSとして使用できるレベルまで磨き上げることです。Apple Siliconは全く文書化されていない
プラットフォームなので、これを実現するには膨大な作業が必要です。特に、Apple GPUのアーキテクチャをリバースエンジニアリングしオープンソースドライバを開発しています。

Asahi Linuxはフリーでオープンソースのソフトウェア開発者たちの活発なコミュニティによって開発されています。

## 名称
Asahiは日本語で『日の出』を意味し、リンゴの品種の名前でもあります。旭りんご（asahi ringo）は、Macの名前の由来となったリンゴの品種
『McIntosh Apple』として知られていています。

## ロゴマーク
![Asahi Linux logo](https://asahilinux.org/img/AsahiLinux_logomark.svg)

Asahi Linuxのロゴとウェブサイトは、[soundflora\*](https://soundflora.tokyo/)がデザインしました。
ロゴのアートワークは[こちら](https://github.com/AsahiLinux/artwork/tree/main/logos)でご覧いただけます。

![Kawaii Asahi Linux logo](https://asahilinux.org/img/AsahiLinux_kawaii_logo.png)
Kawaii Asahi Linuxのロゴは[さわらつき氏](https://x.com/sawaratsuki1004)がデザインしました。ロゴのアートワークは[こちら](https://github.com/SAWARATSUKI/Logos)でご覧いただけます。クリックしてKawaiiモードを有効・無効にしてください(訳注:日本語訳では変更できません）。

## よくあるご質問
### どのようなデバイスに対応するのですか？
すべてのApple Macが対象です。また、開発期間が許す限り将来の世代にも対応します。現在、M1およびM2世代のほとんどのマシンに対応しています。
最新情報は[機能対応ページ](https://github.com/asfdrwe/asahi-linux-translations/wiki/%E6%A9%9F%E8%83%BD%E5%AF%BE%E5%BF%9C)をご覧ください。
また、機能概要の要約を[こちら](https://github.com/asfdrwe/asahi-linux-translations/blob/main/fedora.md#%E6%A9%9F%E5%99%A8%E5%AF%BE%E5%BF%9C)でご覧いただけます。

### これはLinuxのディストリビューションですか?
Asahi LinuxはこれらのMacのサポートを開発するための全体的なプロジェクトです。作業の大部分はハードウェア対応、ドライバ、ツールであり、関連するプロジェクトの
上流に統合されます。私たちのフラッグシップ・ディストリビューションは[Fedora Asahi Remix](https://github.com/asfdrwe/asahi-linux-translations/blob/main/fedora.md)で、
これはAsahi LinuxとFedora Projectのコラボレーションであり、洗練されたエンドユーザー・ディストリビューションであると同時に、私たちの作業を取り入れたい他の
ディストリビューションのリファレンスとしての役割も果たしています。

他のディストリビューションもすでにこれらのプラットフォーム対応の実装に取り組んでおり、将来的にはより多くのオプションが公式に利用できるようになると期待しています。
現在進行中のディストリビューション統合プロジェクトのリストについては、[代替ディストリビューション](https://github.com/asfdrwe/asahi-linux-translations/wiki/SW%3A%E4%BB%A3%E6%9B%BF%E3%83%87%E3%82%A3%E3%82%B9%E3%83%88%E3%83%AA%E3%83%93%E3%83%A5%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3)のページをチェックしてください。

### Appleはこれを許可しているのでしょうか?脱獄は必要ないのですか?
さらに読む 「Apple Silicon入門」と「Apple Platform Security Crash Course」。
AppleはApple Silicon Mac上で署名されていない/カスタムカーネルを脱獄せずに起動することを許可しています! これはハックや手抜きではなく、Appleがこれらの
デバイスに組み込んだ実際の機能なのです。つまり、iOSデバイスとは異なり、AppleはMacで使用できるOSをロックダウンするつもりはないということです
（おそらく開発には協力しないでしょうが）。さらなる情報は『[Apple Siliconの紹介](https://github.com/asfdrwe/asahi-linux-translations/wiki/Apple-Silicon%E3%81%AE%E7%B4%B9%E4%BB%8B)』と『[Appleプラットフォームセキュリティクラッシュコース](https://github.com/asfdrwe/asahi-linux-translations/wiki/Apple%E3%83%97%E3%83%A9%E3%83%83%E3%83%88%E3%83%95%E3%82%A9%E3%83%BC%E3%83%A0%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E3%82%AF%E3%83%A9%E3%83%83%E3%82%B7%E3%83%A5%E3%82%B3%E3%83%BC%E3%82%B9)』を参照してください。

### これは合法ですか？
Linux対応の構築にmacOS からコードを取得しない限り、macOS の派生作品ではないので、配布やエンドユーザの使用は完全に合法です。
詳しくは[著作権およびリバースエンジニアリング方針](https://github.com/asfdrwe/asahi-linux-translations/blob/main/copyright.md)をご覧ください。

### どのようにリリースされるのですか？
すべての開発は私たちの[GitHub](https://github.com/AsahiLinux)で行われます。すべての貢献はそれぞれの（Linuxカーネルから始まる)上流プロジェクトに統合する
ことを意図して書かれていて、現実的な範囲で早期に統合される予定です。コードは上流ライセンス (例: GPL) と寛容なライセンス (例: MIT) の二重ライセンスとし、
可能な限り他の OS でも再利用できるようにします。

### これでApple Silicon Macは完全にオープンなプラットフォームになるのでしょうか？
いいえ。Appleは、例えば、ブートプロセスやSecure Enclave Processor上で動作するファームウェアを依然として制御しています。しかし、現代のデバイスで
『完全にオープン』なものはありません。完全にオープンなソフトウェアとハードウェアを持つ有用なコンピュータは
現在存在しません（一部の企業がそのように売り込もうとするのは当然ですが）。最終的に変わるのはクローズドパーツとオープンパーツの間にどこで線を引くかです。
Apple Silicon Mac における線引きは、SEP ファームウェアが閉じたまま、代替カーネルイメージが起動するときです。これは、標準的な PC における線引きと
非常に似ており、UEFI ファームウェアは OS ローダを起動するが、ME/PSP ファームウェアは閉じたままです。実際、メインストリームのx86 プラットフォームは、
プロプライエタリな UEFI ファームウェアが SMM 割り込みを介していつでも OS からメイン CPU を奪うことが許されているため、間違いなくより侵入的です。
Apple Silicon Mac ではそうなっていません。これは単なる哲学的な問題ではなくパフォーマンスや安定性に大きく影響します。
さらなる情報は『[Apple Silicon MacでのオープンOSエコシステム](https://github.com/asfdrwe/asahi-linux-translations/wiki/Apple-Silicon-Mac%E3%81%A7%E3%81%AE%E3%82%AA%E3%83%BC%E3%83%97%E3%83%B3OS%E3%82%A8%E3%82%B3%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0)』を参照してください。

### Asahi Linuxは誰が作っているのですか？
Asahi Linux はコミュニティであり、誰でも貢献することができます。
もし興味があれば[貢献ページ](https://github.com/asfdrwe/asahi-linux-translations/blob/main/contribute.md)をご覧ください！
プロジェクトのインフラと財務は、私たちの理事会が監督しています。
詳しくは[ガバナンスページ](https://github.com/asfdrwe/asahi-linux-translations/blob/main/governance.md)をご覧ください。

現在の主な貢献者は以下の通りです。
- [Alyssa Rosenzweig氏](https://rosenzweig.io/) Asahi GPU代表。Alyssa氏は、Arm Mali GPUをリバースエンジニアリングし、フリーのPanfrostドライバを構築したことで知られるLinuxグラフィックスハッカーです。現在、上流のMesa3D開発者として、AsahiのOpenGLとVulkanドライバのメンテナンスを行っています。
- [Asahi Lina氏](https://github.com/asahilina) GPUカーネル魔術師。Lina氏は、M1 GPUカーネルインターフェイスのリバースエンジニアリングのためにチームに参加し、気がついたら世界初のRust Linux GPUカーネルドライバを書いていました。Asahi DRMカーネルドライバに取り組んでいないときは、オープンソースのVTuberツールやインフラをハックすることもあります。
- [Dougall Johnson氏(dougallj)](https://github.com/dougallj) 命令セットアーキテクチャの天才。Dougall氏はApple GPUの命令セットの多くをリバースエンジニアリングし、マイクロアーキテクチャの詳細を推測するためにApple M1のCPUコアのタイミングを分析しています。
- [Eileen Yoon氏 (eiln)](https://github.com/eiln) マルチメディアとアクセラレータを担当する信号処理の第一人者。Eileen氏は現在、ハードウェアのビデオコーデック・アクセラレーションに取り組んでおり、すでにニューラルエンジンとイメージ・シグナル・プロセッサー（カメラ）を完成させています。
- [James Calligeros氏 (chadmed)](https://github.com/chadmed) エネルギーを考慮したスケジューリング(EAS)を実装し、スピーカーのために派手なDSPをハンドチューニングし、派手なDSPが派手な電力を消費しないようにPipeWireに利用クランプを追加しました。また、asahi-gentooのメンテナンスも行っています。
- [Janne Grunau氏](https://github.com/jannau)  M1シリーズのタッチパッド／キーボードサポートを実装し、最近HDMI出力サポートを追加したディスプレイコントローラ（DCP）ドライバをメンテナンスしています。また、DeviceTreeやサブミッションを含む、数え切れないほどの 他の項目にも関与しています。
- [Mark Kettenis](https://github.com/kettenis) OpenBSDの開発者。Mark氏 は、PCIe および NVMe (ANS) に必要なブローアップを含む、Apple M1 コアペリフェラル用の m1n1 および U-Boot ドライバを書きました。また、Mark は、Linux への移植と並行して、Apple M1 用の OpenBSD ドライバも記述しています。
- [Sven Peter](https://github.com/svenpeter42) USB、PCIe、イーサネット、Wi-Fi に必要な Apple の DART (Device Address Resolution Table) の上流Linux対応に精力的に取り組んできました。また、m1n1にUSBガジェット対応を追加し、現在はDisplayPortとThunderboltのサポートに取り組んでいます。

過去の主な貢献者は以下の通りです。

- [Hector Martin氏(marcan)](https://github.com/marcan) Asahiプロジェクトの創設者。marcan氏は熟練のリバースエンジニア兼デベロッパーで、15年以上の間、Linuxを移植したり、文書化
されていないデバイスやクローズドデバイスで非公式ソフトウェアを実行した経験を持っています。Asahi Linuxはmarcan氏にとって最も野心的なプロジェクトでした。
これまでのプロジェクトには、PS4上の独自のハードウェアに Linux を移植し、OpenGL と Vulkan (radeon/amdgpu ドライバー) を使用して完全な 3D アクセラレーションを可能にした 
[PS4 Linux](https://github.com/fail0verflow/ps4-linux)、GameOS モード用の PS3 Linux ブートローダーと関連カーネルパッチで PS3 Slim で Linux を動作させる 
[AsbestOS](https://github.com/marcan/asbestos)、[The Homebrew Channel](https://wiibrew.org/wiki/Homebrew_Channel) と 
[BootMii](https://wiibrew.org/wiki/BootMii) 開発チームの一員として [Wii Homebrew エコシステム](https://wiibrew.org/)に多くの貢献をし、
多くのハードウェアを文書化してopen homebrew SDKツールに寄与しました。
- [Martin Povišer (povik)](https://github.com/povik/) 私たちのオーディオカーネルドライバの取り組みをリードしてくれました。Martin氏は、Apple固有のSoCオーディオドライバと、Apple独自のコーデックおよびコーデックバリアント用のドライバを書きました。
