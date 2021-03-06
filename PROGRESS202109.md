[Progress Report: September 2021](https://asahilinux.org/2021/10/progress-report-september-2021/)の非公式日本語訳です。

訳注: 
- Githubのmarkdownで使えないiframeでのtwitterへの埋め込みをリンクに置き換え

---
# 進捗報告:2021年9月
- [前回](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS202108.md)
- [次回](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS20211011.md)

今月はとても忙しい月でした。カーネル側で多くの動きがあり、ツールの改良やリバースエンジニアリングのセッションも行われました。
現時点でAsahi Linuxは基本的なLinuxデスクトップ（GPUアクセラレーションはなし）として使用可能です！これまで基盤が変動していましたが、
ドライバを安定化させました。これまでの経緯を見てみましょう。

## 多数のLinuxドライバ
今年の初めに最下層レベルのドライバがカーネルに統合されました。これらはbring-upとしては重要ですが、使えるシステムのためにはさらに多くの
ドライバが必要です。9月に入ってからこの分野で多くの動きがあり、多くの重要なドライバがレビュー段階になり、一部はLinux 5.16ですでに統合
されました。Asahi Linuxプロジェクトの目標はすべてをLinuxカーネルに取り込ませることなので、私たちのすべてのドライバは最終的に
Linux上流でのレビューに進んでいきます。

- **PCIe バインディング**（**統合済み**): Mark Kettenis氏はU-Boot と OpenBSD の M1 への移植に取り組んでおり、M1 の PCI Express 
ハードウェアのための DeviceTreeバインディングに貢献しました。これらのバインディングは事実上複数のオープンな OS がどのようにハードウェアの
記述方法について合意するための基準で、これによりOSは同じブートローダから起動可能になります。

- **PCIe ドライバ** (`pcie-apple`、**統合済み**): Marc Zyngier氏はPCIe ドライバとそれに付随するすべての細かな依存関係や変更点を仕上げました。
このドライバの仕事は、物理的なPCIeポートをセットアップし、M1のAIC割り込みコントローラへのMSI割り込みマッピングを処理し、DART IOMMUドライバと
協力してデバイスを正しいIOMMU groupに入れることです。これによりMac MiniのUSB-AとEthernetポートが動作するようになりました。
これらはLinuxにはすでにドライバが用意されている標準的なPCIeチップです。

- **USB-C PD ドライバ** (`tps6598x`、**統合済み**): M1マシンはTexas Instruments社のUSB-Cコントローラチップを使用しており、USB-PDの
ネゴシエーションや代替モードなどを処理します。Linux にはすでに TPS6598X 用の基本的なドライバがありますが、M1 マシンのチップは 
Appleバリアント(CD3217/CD3218) で、わずかな違いがあります。[Sven Peter](https://twitter.com/svenpeter42)氏は、既存のドライバをこの特別バージョンに
対応させるために尽力しています。これにより、充電とUSB2ホットプラグが動作するようになりました。ただし、後者については
追加のパッチが必要です。M1マシンのUSB-CコントローラとM1のUSBハードウェアの間でハンドシェイクが発生し、両者間の
[eUSB2](https://e2e.ti.com/blogs_/b/analogwire/posts/understanding-embedded-usb2-eusb2)リピータを
設定する必要があるためです。このUSBの規格を知らなかった人は多いはずです。

- **Pinctrlドライバー**（`apple-gpio-pinctrl`、*レビュー中*）: [Joey Gouly](https://twitter.com/captainjey)氏は GPIO・pinctrl 
ドライバの整理と書き直しに忙しかったです。このドライバはM1の汎用I/Oピンを処理し、外部ペリフェラルのリセットラインの制御などに使用されます。
これはPCIeを適切に動作させるために必要なものです。Joey氏と私（Marcan氏)は、マルチメーターとオシロスコープを片手に、GPIOコントローラのすべてのビットが
何をしているのかを正確に調べることに時間を費やしました。ドライバは順調に仕上がっており、次のパッチセット・バージョンでは統合できるように
なっているでしょう。

- **I²Cドライバー**（`i2c-pasemi`、*レビュー中*）: M1はI²Cハードウェアを借用しています...PA SemiのPWRficient PA6T-1682M以外の
なにものでもない、AmigaOne X1000で使用されているものです！これらのチップは明らかにPA Semiの遺産だと判明しました。Linuxにはすでに
このハードウェア用のドライバがありますが、PowerPCチップではPCIデバイスであるのに対して、M1ではプラットフォームデバイスとなっています。
Sven氏は、既存のドライバをPCI部分から切り離し、プラットフォームデバイスのサポートを追加するパッチシリーズを提出しました。現在、
AmigaOneのハードウェアを持っている人たちによってテストされ何も壊れていないことを確認しており、その後統合できるようになるはずです。
このハードウェアは、オーディオアンプチップやUSB-Cポートコントローラなどとの通信に使用されます。

- **ASC メールボックスドライバー**（`apple-mailbox`、*レビュー中*）: アップルのSoCには補助的なタスクを処理するためのさまざまなサイドコアが多数搭載
されていて、これらのコアはメインCPUと通信する必要があります。これらは『ASC』と呼ばれ、すべて同じローレベルの『メールボックス』ハードウェアを
共有しています。Sven氏はこのドライバにも貢献しており、これは最小レベルの通信（96ビットのメッセージの送受信）を処理します。

- **IOMMU 4Kパッチ**（*レビュー中*）: M1が特異なのは、16Kと4Kページのどちらかを使用するOSに対応しているにもかかわらず、実際には16Kシステム用に
設計されていることです。M1のDART IOMMUハードウェアは16Kページしか対応していません。このチップが4Kに対応しているのは主にmacOS上で
Rosettaを動作させるためです。macOS自体は常に16Kページで動作しており、Rosettaアプリだけが4Kモードになっています。
Linuxでは、このようにページサイズを混在させることはできませんし、今後もできないでしょうから、難問が残ったままです。
16Kカーネルを実行していると、古いユーザースペース（主にAndroidやx86エミュレーション）との互換性が難しくなり、
またディストリビューションは通常16Kカーネルを出荷しません。一方で4Kカーネルを実行すると、DARTとの大きなミスマッチが発生します。
これは当初解決不可能な問題のように思えましたが、Sven氏がこの問題に取り組み、IOMMU のページサイズがカーネルのページサイズよりも
大きいハードウェアとうまく連携するようにLinux の IOMMU サポートレイヤーを対応させるパッチを作成しました。これは完全ではありません。
(この状況では根本的に対応できないことを行う)少数のコーナーケースドライバでは対応できないからです。しかし、これはうまく機能し、
4K カーネルを実現するために必要なすべてのことに対応します。

- **デバイス電源管理** (`apple-pmgr-pwrstate`、*レビュー中*):最近の電力効率の高いSoCの基本パーツは、電力を節約するために、
さまざまな範囲で自身の一部をオンまたはオフにする機能になります。多くのSoCでこの機能は、異なるブロックへの電力制御や
クロックのオン/オフなどを行う個別のハードウェアとして扱われ、複雑な操作シーケンスを必要とします。代わりにAppleのSoCでは、
ハードワークのほとんどを自動化するはるかにハイレベルのインターフェースを備えています。そのため、最初はハードウェアが実際より
少ない処理しかしていないのではないかと混乱してしまいました。結局、ハードウェアをより深く理解するために、様々なカーネル抽象化を
用いてこのドライバーを何度も書き直しました。私が書いた最新のバージョンでは、ハードウェアをLinux Generic Power Domainsと
して表現しています。これにより、Linuxのデバイスフレームワークとスムーズに統合することができ、ドライバで電源管理に対応
していないハードウェアでも動作するようになりました（この場合デバイスは常にオンになっています）。また、概念の証明として、
UARTドライバにも電源管理対応を追加しました。おまけに、ドライバはハードウェアブロックのリセットも処理します 
(リセットプロバイダとして)。これを利用して他のハードウェアのドライバを問題なく再ロードできるようにしています。

- **CPU 周波数のスケーリング** (`apple-cluster-clk`, *RFC 前の最終的なクリーンアップ*): 電源管理のテーマに引き続き、Linux は
CPU コア周波数のスケーリングのためのドライバを必要としています。起動時、4つの『Icestorm』高効率コアは最大のパフォーマンスを
発揮しますが、4つの『Firestorm』高性能コアは最低のパフォーマンス状態になります。デバイス電源管理と同様に、M1はこのための非常に
ハイレベルのインターフェースを持っていますが、ちょっとした工夫があります。CPUのパフォーマンスがより高い状態では、システムの
パフォーマンスを向上させるためにメモリコントローラの構成を調整することも望ましいのです。これを実現するために2つドライバを
書きました。1つはCPUクラスタのパフォーマンス状態を制御するためのもの、もう1つはメモリコントローラの設定を制御するためのものです。
現在のアプローチは、既存の`cpufreq-dt`ドライバを利用して重い処理を行っていますが、このアプローチに自信を持って確定するためには、
一通りのコメントを経る必要があります。また、この作業の一環として、CPUの周波数切り替え時のレイテンシーをベンチマークしました。
これをcpufreqフレームワークで提供することにより、cpufreqフレームワークが正しい判断をできるようになります。

- **RTKitレイヤー**（`apple-rtkit`、*開発中／機能する*）: ASCメールボックスドライバはローレベルの通信しか提供しませんが、
ほとんどすべてのASCコプロセッサはRTKit組み込みOSを実行しており、その上で同様のハイレベルの通信インターフェースを提供しています。
Sven氏は、下位のデバイスドライバがこの共通の処理コードを共有できるように、このライブラリモジュールを開発しました。
現在、NVMeの『ANS』ドライバーで使用されていますが、上流に向かうにはもう少し作業が必要です。

- **NVMe + SART** (`apple-ans-nvme`/`apple-sart`, *開発中／機能する*): M1に搭載されているNVMeハードウェアは非常に独特です。
複数の点で仕様を破っており、LinuxのNVMeサポートのコア部分にパッチを適用する必要があるほか、PCIeではなくプラットフォームデバイスと
して公開する必要があります。さらに、このデバイスはASCである『ANS』によって管理されており、NVMeが動作する前に立ち上げる必要があります。
また、最小のIOMMUのような付属の『SART』ドライバにも依存しています。Sven氏はこのためのすべての要素をまとめるのに多くの時間を費やし、
現在では下流のカーネルブランチでNVMeがうまく動作するようになりました。しかし、上流に向かうにはまだ大幅な改善が必要です。

- **DCP**（`apple-dcp`、*開発中/機能する*）: M1のディスプレイコントローラのハードウェアについては、
 [前回の進歩報告](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS202108.md)で
紹介したので、ここでは説明を割愛します。詳細を知りたい方は戻ってください。[Alyssa Rosenzweig氏](https://twitter.com/alyssarzg)が
Linux用のドライバを書く課題に取り組んでいて、すでにうまく機能しています。これにより、解像度の切り替え（4K HDMIモニターのサポートを含む）や、
適切なティアフリーのページめくりなどが動作します。これはDCPがRTKitのASCでもあるので、Sven氏のRTKitとメールボックスレイヤー上に
構築されています。

## AppleのSoCは他と違う

今回のLinuxドライバ開発では、組み込みARMシステムの世界では珍しいことをしてきました。一般的なSoCでは、ドライバーは基盤となるハードウェアを
熟知しており、その正確なレイアウトをハードコーディングしています（レジスタ、ピン、相互関係など）。、これはほとんどのSoCにとって事実上の必須条件です。
なぜならば、ハードウェアは世代ごとに大きく変化する傾向があり、ドライバーは常に新しいハードウェアに対応するための変更を必要とするからです。

しかし、AppleはSoCの世代を超えて
[ハードウェアインターフェースの互換性を保つこと](https://twitter.com/stuntpants/status/1442276493669724160)に重点を置いている
唯一の企業です。M1のUARTハードウェアは初代iPhoneにまで遡ります！つまり、M1で動作するだけでなく将来のチップでも
そのまま動作する可能性のあるドライバーを書いてみることができるという唯一の立場に私たちは置かれているのです。これは、ARM64の世界では非常に
エキサイティングな機会です。AppleがM1X/M2をリリースするまではわかりませんが、もし新しいチップでLinuxを起動できるだけの
ドライバーの前方互換性を実現できれば、古いディストリビューションのインストーラーを新しいハードウェアで起動できるようになります。
これはx86では当たり前のことですが、組み込みの世界では通常不可能です。それをこのマシンで変えていきたいと思っています。

そのためには今までとは違った考え方が必要です。ハードウェアの正確なレイアウトをドライバーにハードコーディングするのではなく、
DeviceTreeに基づいて情報を提供するのです。DeviceTreeとは、デバイスごとに変わる『パラメータ』のような部分で、動作を
根本的に変えることはありません。例えば、他のSoCでは、デバイス電源管理ドライバは単一のデバイスを制御しハードコードされた
リストとしてすべてのオンボードデバイスの電源管理を提供します。しかし、私たちのPMGRドライバーは、管理すべきデバイスごとに
実際にインスタンス化されて1つのレジスタを制御します。そして、DeviceTreeがこれらの電源領域間の依存関係を
動的に表現するために使われます。つまり、M1X/M2では電源管理レジスタの数や配置が異なっていても、それぞれのレジスタが同じように
動作する限り、既存のドライバが機能するということになります。GPIOドライバー（ピン数）、CPU周波数ドライバー（クラスターの数や種類、
対応する周波数）なども同様です。

このアプローチは、ほとんどの上流サブシステムのメンテナには馴染みがありませんが、時間の経過とともにそのメリットを
認識してもらえることを願っています。もしかしたら他のメーカーがこの方法を採用するきっかけになるかもしれません。

## デスクトップへの道

これらのドライバにより、M1 Macは実際にデスクトップLinuxマシンとして使用可能となっています！GPUアクセラレーションはまだありませんが、
M1のCPUは非常にパワフルなので、ソフトウェアでレンダリングされたデスクトップはハードウェアアクセラレーションを
***備えた***Rockchip ARM64マシンなどよりも実際に***速く***なっています。

まだまだ粗削りな部分や不足しているドライバがあるのは確かですが、ここまでくれば、開発をセルフホストで行うことができ、
開発者は自分のドッグフードを食べることができます。Alyssa氏はまさにそれを実践しており、自分のカーネルマージを実行する
M1 Macを日常的に使用しています。彼女の[Twitter](https://twitter.com/alyssarzg)をフォローして、セットアップの最新情報を入手してください。

[埋め込みツイート](https://twitter.com/alyssarzg/status/1443348289949212674)
(M1でAsahi Linuxを試すのが待ちきれない :-))

これらのLinuxドライバーの準備が整い次第、Asahi Linuxを最小限の手間で試すことができる公式インストーラーの提供を開始する予定です。
ただし、まだ多くの欠落部分（USB3、TB、カメラ、GPU、オーディオなど）や、そのままバンドルするには少々問題のあるパッチセット
（大幅な書き換えが必要なWiFi）があるので、このプロジェクトの目標である洗練された体験には程遠いことをご了解ください。
しかし、このプロジェクトにより最先端にいることを望む人々がこれらのマシン上でLinuxを実行することがどのようなものかを味わうことが
できると私たちは願っています。また、何人かには本番での使用がこれで十分な場合になっているもしれません。

## インストーラーの更新

ますます多くの方がこの機能を試してみたいと思っているので、アルファ版インストーラー（現在はスタブOSパーティションとしてm1n1を
インストールするだけ）をアップデートして、古いmacOSバージョンのサポート、異なるリカバリバージョンの処理、同じAPFSコンテナ内の
複数のmacOSインストールの対応を強化しました。もしあなたが開発者でこの機能を試してみたいと思われるなら、私たちのIRCチャンネルに
来ていただければすべてのセットアップをお手伝いします。インストーラがどのように動作するかを知りたい方は、
[前回の進捗報告](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS202108.md)をご覧ください。

安定したカーネル基盤ができあがったら、『公式』インストーラーの公開を開始します。そのバージョンでは、Linux のためのスペースを
確保するために macOS をリサイズしたり、m1n1 + U-Boot をインストールしたり、EFI パーティションをセットアップしたり、
オプションでビルド済みのディストリビューションとそのブートローダ（最初は Arch Linux ARM）をインストールしたりすることができます。
ところで、Mark Kettenis氏は数日前にアップストリーム U-Boot に M1 サポートを追加するシリーズの v2 を
[提出](https://lore.kernel.org/all/20211003183050.67925-1-kettenis@openbsd.org/)しました! 
今後のアップデートにご期待ください :-)

## さらなるリバースエンジニアリング

### ハイパーバイザー SMP

CPU周波数ハードウェアのリバースエンジニアリングの一環として、完全なSMPをサポートしたm1n1ハイパーバイザの下でmacOSを実行する必要が
あることに気づきました。ハイパーバイザーは8つのCPUコアすべてをゲストに公開し、CPU起動ハードウェアを仮想化することができるようになりました。
これは、SMP関連の機能をリバースエンジニアリングする上で重要なだけでなく、ベアメタル上とほぼ同じ速さで起動することを意味しています。

[埋め込みツイート](https://twitter.com/AsahiLinux/status/1438152384165728257)
(8倍のコアのハイパーバイザー)

カーネルの開発においてハイパーバイザーを利用しない理由はほとんどありません。ハイパーバイザーはUSB経由で仮想UARTを提供し、
インタラクティブなデバッグ機能も備えているので、シリアルデバッグケーブルを持っていない人でも作業がしやすくなっています。

### オーディオ
Martin Povišer氏は M1 のオーディオハードウェアのリバースエンジニアリングに時間を費やし、DMA とスピーカーアンプハードウェアのための
[PoC m1n1 ドライバ](https://github.com/AsahiLinux/m1n1/blob/main/proxyclient/experiments/speaker_amp.py)を作成し、
Mac Mini の内蔵スピーカーでオーディオを再生できるようにしました。彼は現在、このハードウェアのLinux ASoCドライバの開発に着手しています。
オーディオは私たちのリストの中でまだ誰も見ていない大きなTODOでしたので、誰かがこの挑戦に取り組むを見るのは非常にエキサイティングです！
Martin氏ありがとう！

驚いたことに、macOSのオーディオドライバーがM1の汎用クロック制御レジスタのいくつかを直接制御していることがわかりました。
このレジスタの仕組みやクロック周波数のマッピングには時間をかけていたのである程度知ってはいましたが、まさかドライバーが
必要になるとは思っていませんでした。おそらく必要になりそうなので、将来的には`apple-clocksel`が登場することになるでしょう。

# Type-C / USB3 / DisplayPort / Thunderbolt

Sven氏と私はSuperSpeed関連のType-Cハードウェアを動作させるために何が必要かを調査しはじめました。これは複数のドライバ
（DisplayPort多重化、Thunderbolt、Apple Type-C PHY、DWC3 host/gadgetなど）が複雑に絡み合う大きな研究領域で、
Linuxカーネルでは初めての試みで他のSoCではまだ行われていないものも含まれていますが、近い将来これに対するLinux対応を
開始したいと考えています。その間でもUSB2は正常に動作しMac MiniのType-A USB3ポートも問題なく動作します。

## 次回

次の大きな段階が何であるかは誰もが知っています。GPUです！まもなくGPUカーネルインターフェースに着手します。カーネルの未完成な部分が
いくつか解決されレビューが終わってからです。参考までに、Alyssa氏はすでにMesaのユーザースペース側（シェーダー、描画コマンドなど）で
多くの作業を行っており、macOSのカーネルドライバでうまく動作しています。その作業をLinuxに移植するにはどれくらいの時間がかかるのでしょうか？
ご期待下さい！

#### marcan 2021-10-06


