[About Asahi Linux](https://asahilinux.org/about/)の2021年12月25日時点の非公式日本語訳です。
 
---
# Asahi Linuxについて
Asahi Linuxは、2020年のM1 Mac Mini、MacBook Air、MacBook Proから始まったApple Silicon MacにLinuxを移植することを目的としたプロジェクトでコミュニティです。

私たちの目標は、単にこれらのマシン上でLinuxを動作させるだけでなく、日常的なOSとして使用できるレベルまで磨き上げることです。Apple Siliconは全く文書化されていない
プラットフォームなので、これを実現するには膨大な作業が必要です。特に、Apple GPUのアーキテクチャをリバースエンジニアリングし、Apple GPU用オープンソースドライバを
開発しています。

Asahi Linuxはフリーでオープンソースのソフトウェア開発者たちの活発なコミュニティによって開発されています。

## 名称
Asahiは日本語で『日の出』を意味し、リンゴの品種の名前でもあります。旭りんご（asahi ringo）は、Macの名前の由来となったリンゴの品種
『McIntosh Apple』として知られていています。

訳者による追記:[青森りんごTS導入協議会の旭りんごの説明](https://www.ringodaigaku.com/main/hinshu/a/asahi.html)

## ロゴマーク
![Asahi Linux logo](https://asahilinux.org/img/AsahiLinux_logomark.svg)

Asahi Linuxのロゴとウェブサイトは、[soundflora\*](https://soundflora.tokyo/)がデザインしました。
ロゴのアートワークは[こちら](https://github.com/AsahiLinux/artwork/tree/main/logos)でご覧いただけます。

## よくあるご質問
### どのようなデバイスに対応するのですか？
すべてのApple M1 Macが対象です。また、開発期間が許す限り将来の世代にも対応します。最初のターゲットはM1 Mac Miniです。

### これはLinuxのディストリビューションですか?
Asahi LinuxはこれらのMacのサポートを開発するための全体的なプロジェクトです。最終的には[Arch Linux ARM](https://archlinuxarm.org/)のリミックスを、
エンドユーザーによるインストール用にパッケージ化し、同名のディストリビューションとしてリリースする予定です。作業の大部分はハードウェア対応やドライバやツールで、
関連するプロジェクトの上流に統合される予定です。このディストリビューションは、エンドユーザーが簡単にインストールできる便利なパッケージで、私たちが開発する
ソフトウェアの最先端バージョンにアクセスできるようになります。

最終的にコードが吸い上げられ他のディストリビューションに還元されることを期待しています。その前に、上級ユーザはいつでも自由に好きなディストリビューションを使い、
必要なパッチやソフトウェアを自分で追加することができます。

### Appleはこれを許可しているのでしょうか?脱獄は必要ないのですか?
Appleは、Apple Silicon Mac上で署名されていない/カスタムカーネルを脱獄せずに起動することを許可しています! これはハックや手抜きではなく、Appleがこれらの
デバイスに組み込んだ実際の機能なのです。つまり、iOSデバイスとは異なり、AppleはMacで使用できるOSをロックダウンするつもりはないということです
（おそらく開発には協力しないでしょうが）。

### これは合法ですか？
Linux対応の構築にmacOS からコードを取得しない限り、macOS の派生作品ではないので、配布やエンドユーザの使用は完全に合法です。
詳しくは[著作権およびリバースエンジニアリングポリシー](https://asahilinux.org/copyright/)をご覧ください。

### どのようにリリースされるのですか？
すべての開発は私たちの[GitHub](https://github.com/AsahiLinux)で行われます。すべての貢献はそれぞれの（Linuxカーネルから始まる)上流プロジェクトに統合する
ことを意図して書かれていて、現実的な範囲で早期に統合される予定です。コードは上流ライセンス (例: GPL) と寛容なライセンス (例: MIT) の二重ライセンスとし、
可能な限り他の OS でも再利用できるようにします。

### これでApple Silicon Macは完全にオープンなプラットフォームになるのでしょうか？
いいえ。Appleは、例えば、ブートプロセスやSecure Enclave Processor上で動作するファームウェアを依然として制御しています。しかし、現代のデバイスで
『完全にオープン』なものはありません。完全にオープンなソフトウェアとハードウェアを持つ有用なコンピュータは
現在存在しません（一部の企業がそのように売り込もうとするのは当然ですが）。最終的に変わるのはクローズドパーツとオープンパーツの間にどこで線を引くかです。
Apple Silicon Mac における線引きは、SEP ファームウェアが閉じたまま、代替カーネルイメージが起動するときです。これは、標準的な PC における線引きと
非常に似ており、UEFI ファームウェアは OS ローダを起動するが、ME/PSP ファームウェアは閉じたままです。実際、メインストリーム x86 プラットフォームは、
プロプライエタリな UEFI ファームウェアが SMM 割り込みを介していつでも OS からメイン CPU を奪うことが許されているため、間違いなくより侵入的です。
Apple Silicon Mac ではそうなっていません。これは単なる哲学的な問題ではなく、パフォーマンスや安定性に大きく影響します。

### Asahi Linuxは誰が作っているのですか？
Asahi Linux はコミュニティであり、誰でも貢献することができます。もし興味があれば[貢献ページ](https://asahilinux.org/contribute)をご覧ください！
主な貢献者は以下の通りです。

- Hector Martin 『marcan』、Asahiプロジェクトのリーダーです。marcan氏は熟練のリバースエンジニア兼デベロッパーで、15年以上Linuxを移植したり、文書化
されていないデバイスや閉じたデバイスで非公式ソフトウェアを実行した経験を持っています。今回のプロジェクトは、彼にとって最も野心的なプロジェクトであり、
[コミュニティの寄付やスポンサーシップ](https://asahilinux.org/support)を通じて資金を調達しています。彼のこれまでのプロジェクトには、
PS4上の独自のハードウェアに Linux を移植し、OpenGL と Vulkan (radeon/amdgpu ドライバー) を使用して完全な 3D アクセラレーションを可能にした 
[PS4 Linux](https://github.com/fail0verflow/ps4-linux)、GameOS モード用の PS3 Linux ブートローダーと関連カーネルパッチで PS3 Slim で Linux を動作させる 
[AsbestOS](https://github.com/marcan/asbestos)、[The Homebrew Channel](https://wiibrew.org/wiki/Homebrew_Channel) と 
[BootMii](https://wiibrew.org/wiki/BootMii) 開発チームの一員として [Wii Homebrew エコシステム](https://wiibrew.org/)に多くの貢献をし、
多くのハードウェアを文書化してopen homebrew SDKトール寄与するなどです。
- Alyssa Rosenzweig、Asahi GPUの主役。Alyssa氏は、Arm Mali GPUのリバースエンジニアリングを行い、フリーのPanfrostドライバを構築したことで
知られるLinuxグラフィックスハッカーです。彼女は、Mesa3D の上流開発者であり、Panfrost と Asahi ドライバの両方をメンテナンスしています。
- Dougall Johnson 『dougallj』、命令セットアーキテクチャに関する並はずれた人物です。Dougall氏 は Apple GPU の命令セットの多くをリバースエンジニアリングし、
Apple M1 の CPU コアのタイミングを解析してマイクロアーキテクチャの詳細について解析しています。
- Sven Peter、 Sven氏は、USB、PCIe、Ethernet、Wi-Fiに必要なAppleのDevice Address Resolution Table（DART）に対するLinux上流での対応に
休むことなく取り組んできました。彼はまたm1n1 に USB ガジェットへの対応を追加しました。
- Mark Kettenis、OpenBSD の開発者。Mark氏は、PCIe と NVMe (ANS) に必要なbringupを含む、Apple M1 コアペリフェラル用の m1n1 と U-Boot 
ドライバを書きました。また、Mark氏は、Linux の移植と並行して、Apple M1 用の OpenBSD のドライバも書いています。
