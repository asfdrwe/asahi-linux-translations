[July 2022 Release & Progress Report](https://asahilinux.org/2022/07/july-2022-release/)の非公式日本語訳です。

訳注: 
- まだDeepLの欠陥を貼ってmarkdownの記述を追加しただけ
- Githubのmarkdownで使えないiframeでのtwitterへの埋め込みをリンクに置き換え

---
# 進捗報告:2022年7月

- [前回](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS202203.md)

またまた遅ればせながら、進捗報告へようこそ! いつものように、予想以上に忙しく、そしてビッグニュースもあります。Mac Studio、Bluetooth、M2をサポートした新しいAsahi Linuxのアップデートをリリースしました!

Asahi Linuxを初めてお使いになる方は、インストール方法と一般的な情報を前回のリリース告知でご確認ください。

## Mac Studioがファミリーの一員に

Mac Studioが発表されたとき、私たちは新しいM1 UltraをAsahi Linuxで動作させることに取り掛かりました。これは難しいことではありませんでしたが、1つのSoCに複数のダイがあるというアイデアを扱うために、ブートローダとデバイスツリーにいくつかの変更を加える必要がありました。これは結局、他の変更と一緒になってしまったので、リリースまで予想より少し長く待つことになってしまいましたが、ついに登場しました!

M1 MaxモデルのフロントUSBポートと全モデルのType Aポート（これらはM1 iMacの4ポート版にも必要な特別なファームウェアアップロードサポートでブロックされています-リストにあります）を除いて、ほとんどのハードウェアは期待通り（Mac Miniと同等）に動くと思っていただいて結構です。

## ワイルドなBluetooth登場

Bluetoothは、Appleがどうやら他のベンダーは使っていない新しい特注のPCIeインターフェースに切り替えて以来、しばらく後回しになっていた。しかし、Rがそのリバースエンジニアリングに挑戦したことで、すべてが変わりました! ありがたいことに、ホストとコントローラのインタフェースはほぼ標準化されているので、Bluetooth自体は非常にシンプルです。AppleはPCIe上で動作するものを作りましたが、上位レイヤーは他のBluetoothコントローラーと同じです。Rがユーザ空間の概念実証ドライバをまとめた後、Svenがその作業を引き継ぎ、適切なカーネルドライバを書き始めた。数日前の時点で、Bluetooth が動き出しました!

私たちは、思い切ってこのドライバをアルファユーザに直接出荷することにしました。そして、最新のカーネルとサポートパッケージで利用できるようになりました。ありがたいことに、PCIe トランスポートは新しいものですが、その上で動作する HCI インターフェースは標準的なものなので、ドライバのコアの初期化およびデータ転送部分が動作し始めたら、ほとんどの Bluetooth 機能も動作するようになりました。ドライバはそのような細かいことを気にする必要はなく、ただデバイスとの間でデータをシャッフルするだけです。WiFiとBluetoothの共存がまだ適切に設定されていないため、2.4GHzのWiFiネットワークに接続している場合、Bluetoothのパフォーマンスが低下します。Bluetoothを使用したい場合は、現時点ではWiFiをオフにするか、5GHzネットワークを使用することをお勧めします。

ドライバに加えて、Bluetooth のサポートは、私たちが目指してきた「シームレスアップグレード」のパラダイムを試す最初の大きな試練となります。Asahi Linux はまだ実験的なプロジェクトですが、私たちは、新しいものを動作させるために、ユーザに困難な手作業のステップを強いることはしたくないと考えていました。Bluetoothを有効にするには、スタックのさまざまな部分に小さな変更を加える必要がありました。

- Apple Device Tree から Linux Device Tree に Bluetooth アドレスとキャリブレーション情報を転送するための m1n1 の変更。
- 新しいデバイスを追加し、特定のマシンごとにハードウェアボードの種類を指定するためのデバイスツリーの変更
- Bluetooth ファームウェアを抽出し、Linux が見つけることができる正しい場所に配置するためのインストーラの変更
- 全く新しいLinuxドライバ
- Plasma デスクトップユーザー向けの新しいデフォルトパッケージの追加、および bluetooth.service を有効にする必要があります。

私たちは、このようなことが最初からわかっていたので、リファレンスディストロを設計して、そのすべてを自動的に行うことをサポートしました。

- m1n1は2つのステージに分かれており、第2ステージ（ロジックの大部分を持っています）は通常のパッケージとしてアップグレードすることができます。
- デバイスツリーはLinuxカーネルにバンドルされ、カーネルがアップグレードされるときにm1n1もアップグレードされるので、常に同期が保たれます。
- 既存のインストーラは、いくつかのファームウェアが欠けていることがわかっていたので、Linux から再処理できるように、すべての生のファームウェア（処理前）の「フォールバック」アーカイブを作成するようにしました。私たちは、インストーラのファームウェア処理コンポーネントを含む asahi-fwextract パッケージを追加し、インストール時にフォールバックアーカイブを入力として使用して、ファームウェアバンドルを自動的にアップグレードするようにしました。
- デフォルトのインストールに空の meta パッケージを同梱し、既存のユーザのために、新しいパッケージを依存関係として追加できるようにしました。これを利用して、asahi-fwextract を全員にインストールし、デスクトップユーザが良好な Bluetooth 体験を得るために必要な BlueZ/Bluedevil/etc の雑多なパッケージもインストールしました。また、アップグレード時にbluetooth.serviceを有効にします。

この結果、既存のリファレンスディストロのユーザは、パッケージをアップグレードして再起動するだけで、それ以上の設定や変更なしにBluetoothが動作するようになります!

## M2がやってきた!

アサヒリナックスが始まって以来、最もよく聞かれる質問のひとつが "M2 はどうなるんだ？" です。確かに、M1 は最初の一歩に過ぎず、Apple は新しいチップやマシンのリリースをすぐに止めるつもりはないようです。このことは、Asahi Linuxプロジェクトにどのような影響を与えるのでしょうか？私たちは長い間、新しいチップへの移植は最初のときほどの挑戦にはならないだろうし、多くのドライバは修正なしで動くだろうと考えてきました...そして M2 はこの理論の最初の真のテストとなります。どうだったでしょうか？

[たった12時間の立ち上げマラソン](https://www.youtube.com/watch?v=SidIJkC5YN0) の後、Linux が M2 で起動し、USB、NVMe、バッテリ統計/制御、CPUfreq、WiFi、その他が使えるようになりました! さらに数日の作業で、キーボードとトラックパッドが動作するようになり、既存のシステムと同等の機能を実現することができました。さらにいくつかの統合作業を経て、Asahi Linux のインストーラで M2 マシンを実験的にサポートすることを発表できることを誇りに思います!

M2マシンで試す場合は、以下の注意事項に留意してください。

- これは M1 のサポートよりもさらに実験的なものなので、バグがあることが予想されます。M2にインストールするためには、Asahi Linuxのインストーラでエキスパートモードを有効にする必要があります。
- キーボードは U-Boot/GRUB では動きません。このドライバはまだ書かれていませんし、U-BootとLinuxの間のハンドオフをどのように行うか、まだわかっていません。ブートローダのシェルをつつく必要がある場合は、外付けのUSBキーボードを使うことができます。
- M2 MacBook Pro 13 "のみテストしています。M2 MacBook Air のサポートは完全に未検証ですが、私たちはまだ誰も持っていません。もし持っていても、よほど冒険したい気分のときだけ試してください（そして、うまくいかなくても私たちを責めないでください）。
- Linux 用のファームウェア/スタブは、Apple がこれらのマシンのためだけにリリースした「特別リリース」の macOS 12.4 バージョンがベースになっています。私たちはまだこのバージョンの長期サポートを約束していませんので、GPU や外部ディスプレイ出力などの将来の機能を動作させるには、macOS とインストーラを経由してブートコンポーネントを (おそらく 13.0 に) アップグレードしなければならないかもしれません (つまり、pacman だけでは「シームレスアップグレード」できないが、完全な再インストールも必要ない)。今後どのように進めるか決定し、必要であれば時期が来たらインストーラに必要なアップグレードモードを追加します。結局、12.4をサポートする可能性もありますが、約束はできません。

## トラックパッドのトリック

トラックパッドで何が起こったのか、面白い話なので見てみましょう。Apple MacBookは歴史的にキーボード/トラックパッドの部品に面白いデザインを持っています。ほとんどのx86ラップトップでは、これらは（おそらく仮想）PS/2か、時にはUSBで内部に取り付けられていますが、Appleは最初USBで、その後SPIに移行しています。SPIはUSBよりもシンプルなプロトコルなので、ユーザーからの入力に常に反応する必要があるこのような低帯域幅の周辺機器では電力を節約することができるのです。最新の（T2以前の）Intel MacBookでさえすでにSPIを使用しており、このデザインはM1にも受け継がれています（T2マシンはここでは特殊なので割愛します）。

Appleはまた、実際のキーボード/トラックパッドにかなり異なるアーキテクチャを使用している。多くのx86マシンでは、キーボードはマザーボード上の専用チップであるEmbedded Controllerで駆動され、他の多くのシステム管理機能を制御している。トラックパッドは通常、独立したものである。キーボードはx86マシンと同じくパッシブスイッチマトリックスですが、マザーボードに直接接続するのではなく、代わりにトラックパッドに接続します。トラックパッド自体も、AppleのForce Touch技術によって、かなり変わったものになっています。他のほとんどのラップトップはSynapticsの専用トラックパッドチップを使用していますが、Appleは代わりにBroadcomのBCM5976（iPhoneに搭載されているのと同じ、ARMコアが埋め込まれたタッチスクリーンコントローラ）を採用し、STM32 Cortex-M3マイクロコントローラと組み合わせています。STM32（RTKitファームウェアを実行）は、デザインのキーボードとForce Touch部分の処理を担当し、すべてのデータを結合してAppleのトラックパッドをうまく動作させるアルゴリズムの一部を実装するBCM5976のフロントエンドとしての役割も果たします。これらのチップを合わせると、かなりの量のファームウェアの賢さを備えていることになります。

それからM2が登場して、SPIインターフェイスがなくなりました。いや、なくなったわけではないんですが...OSからはもう見えないんです。その代わり、M2にはもう一つの小さな秘密、MTPがあります。そう、AppleはトラックパッドコントローラーをメインSoCに移したのです!

そう、正確には違うんです。BCM5976とSTM32はまだそこにありますが、どちらのファームウェアも劇的に縮小されました。これらのチップは、もはや派手なマルチタッチ アルゴリズムやロジックを担当するのではなく、生のキーボード/マルチタッチ/Force Touch データを M2 に取り込むための単なるインターフェース チップに追いやられています。M2 には専用のコプロセッサー（おそらく別の ARM64 Chinook インスタンスで、通常通り RTKit を実行）があり、これが処理の大部分を担当しています。

なぜAppleはこのようなことをするのでしょうか。いくつかの理由があります。まず、ファームウェアをM2に統合することで、アップデートの柔軟性が高まります。このファームウェアは、OSごとのバンドルに含まれるため、古いバージョンのmacOSとの後方互換性を損なうことなく、新機能の導入や変更を行うことができるようになりました。また、ファームウェアのアップグレード手順は、macOSのアップグレードと自動的に統合され、より強固になります。そしてもちろん、より小型のSTM32を使用してコストを削減することも可能です。

MTPで、AppleはメインOSとの通信を興味深い方法で処理した。ほとんどのコプロセッサは主要なメールボックス/RTKitインターフェースと共有メモリを使用していますが、AppleはSPIインターフェースの「バイトイン、バイトアウト」モデルを維持したかったようで、その代わりにDockChannelを使用しました... Dockchannelとは一体何なのか？これはAppleが独自にデザインしたもので、基本的にはシンプルなFIFOです。M1チップはすでにデバッグDockChannelを実装しており、Macの特定のType Cポートに接続するAppleの内部デバッガでアクセスできる仮想シリアルポートとして使用できます（私たちは少し調べましたが、このプロトコルがUSB側でどう動作するかはまだ分かっていません）。MTP は同じ FIFO モジュールをメイン OS との間の単純なバイトチャンネルとして再利用し、その上で新しい HID トランスポートプロトコルが実行されます。

良いニュースは、DockChannel自体が非常にシンプルなので、それを持ち出すのに時間がかからなかったことです。悪いニュースは、Apple が新しい HID トランスポートを複雑にしすぎたため、新しいドライバが 1000 行以上のコードになり、ファームウェアのアップロードや GPIO プロキシ、複数の複雑なネストしたデータ構造といったものを扱わなければならなくなったことです! また、MTPをリセットして新しい起動状態に戻す方法がまだわかっていないので、U-BootドライバとLinux間のハンドオフや、Linuxでのデバイスの取り外し/再プロブをまだサポートできていません。

macOS はそれを asn.1 img4 イメージに包まれた XML plist (Python XML plist パーサーがサポートしない機能を含む) として保存し、ドライバは初期化中に MTP に渡す前にそれをバイナリシリアライズ構造 (さらに別のシリアライズ形式..) に変換し、さらに bInterfaceNumber フィールドに正しいインターフェース番号をパッチする必要がある、というわけなのだ。言うまでもなく、私たちはLinuxカーネルにXMLパーサを入れているわけではないので、代わりにAsahi Linuxのインストーラはバイナリ形式に変換するモジュールを持つようになりました。bInterfaceNumberのオフセットを含む小さなヘッダがあるので、LinuxはMTPに渡す前にそれを変更することができます。ふぅ〜。

## Venturaの冒険

macOS 13.0 Venturaのベータ版がリリースされた後、Asahi Linuxのインストールが壊れるという報告を受けました。私たちは、ほとんどのファームウェアはOSに関連しているので、macOSのアップグレードはAsahiを壊さないはずだと長い間主張してきました（だからmacOSのアップグレードはAsahiには影響しないのです）。では、何が起こったのでしょうか？

ほとんどのファームウェアはOSに関連していますが、システムファームウェアの中には、すべてのOSに共有されている小さなサブセットがあります。これには、NVMeファームウェア（明白な理由）、SMCファームウェア、およびいくつかのThunderbolt関連のものが含まれます。しかし、古いバージョンのmacOSとの互換性を維持するために、Appleは基本的に古いファームウェアとの後方互換性を壊さないことを約束しているので、私たちは安全なはずです。

そして、バグがなければ...安全だったでしょう! 実際、m1n1のNVMeドライバは、アップデートされたファームウェアに何の問題もなく、箱から出してすぐに動きました。しかし、U-Bootの方は、それほど幸運ではありませんでした。そのバージョンはClever ShortcutTMを使用していて、新しいファームウェアが私たちの小さなトリックに満足しないことが判明しました。一方、Linux SMC ドライバには、古いファームウェアでは気づかれなかったが、新しい SMC ファームウェアをクラッシュさせるという、明らかな一行バグがありました。

linux-asahi、u-boot、m1n1 パッケージを更新したので、もし新しいベータを試したいなら、まず pacman upgrade をしてください! もし既にベータにアップグレードしてしまっていて、Asahi システムが起動できなくなった場合は、こちらのステップに従って復旧してください。

これらの問題は私たちのコードのバグや見落としが原因であり、Apple が互換性を壊しているわけではないことを心に留めておいてください。一般的には、AsahiがインストールされているMacをアップデートするのは常に安全ですが、バグに噛まれないようにしてください :-)

## DCPをお願いします

Mac Mini（そして現在のMac Studio）は、長い間、起動プロセス中に、HDMI出力と特定のモニターとの信頼性の問題を抱えていました。Appleは以前、OSが立ち上がる前に実際にディスプレイを初期化していましたが、（デスクトップでは）macOS 12.0から完全にそれをやめました。しかし、Asahi Linuxにはまだ適切なディスプレイコントローラドライバが同梱されておらず、また、m1n1、U-Boot、grubのブート画面を表示できる必要があるので、ブート時のフレームバッファをセットアップするためにm1n1でディスプレイを初期化する必要があるのです。これはうまくいきますが、問題は、ブートプロセス中にアクティブに動作するディスプレイドライバがない場合、DCP (the Display Controller Processor) が新しいモードセットコマンドを待つ間にそれをシャットダウンするので、ホットプラグ事象はディスプレイを失わせる原因になるということです。残念ながら、一部のモニタでは、スタンバイ状態から起動すると、数秒後に事実上入力が切断され、再び接続されるという奇妙に壊れた動作をするようです。m1n1はディスプレイを初期化して移動し、モニタは偽のホットプラグ・イベントを発生させて、DCPは信号をシャットダウンします。

そして Asahi Linux はまだカーネルに適切な DCP ドライバを同梱していないので、これは完全に壊れたディスプレイを意味します (これはすぐに修正される予定ですが、我々はまだブートローダのグラフィックスを動かしたいのです...)。しかし、これは不完全な解決策で、すべての人に起動を遅らせたり、インストール時にモニタに回避策が必要かどうかユーザに判断させたりするのは好ましくありませんでした。

私は、この問題に対する何らかの解決策、おそらくホットプラグイベントに反応しないようにするためのDCPコマンドを見つけようとあちこち探しましたが、見つかりませんでした。そこで思いついた。DCPをシャットダウンできないか......」。DCPのリバース・エンジニアリングを始めた頃、クラッシュした後に適切にリセットできないことは分かっていましたし、シャットダウン・モードがどのように機能するのかも明らかではありませんでした。DCPは起動したままでなければならず、完全に停止させるとすべてが壊れてしまうような気がしたのです。しかし、その後、RTKit の電源モードがどのように機能するか、コプロセッサを正しくシャットダウンする方法、およびフルパワーダウン（NVMe などで使用）後に復帰させる方法についての理解が深まりました。そこで、新しい、既知の、完全なシャットダウン手順を試してみたところ...うまくいきました! DCP が停止しているので、モニターのホットプラグ・イベントに反応する人は誰もいません。HDMI 信号は、モニターが何をしているのかに関係なく、ただその場に留まっています。DCPのシャットダウンは制御された方法で行われるので、Linuxは本物のドライバが利用可能になれば何の問題もなくそれを起動させることができます。

m1n1が画面モードを設定してからDCPをシャットダウンするまでの間にモニターがホットプラグされた場合）まだ小さな競合がありますが、それは小さな窓です。実際には、私たちが知る限り、この新しいアプローチによって、影響を受けるすべての人の表示上の問題が解消されました。このアップデートにより、Linux の使用中にモニターのプラグを抜いたり電源を落としたりしても、次に電源を入れたときに信号が途絶えることがなくなりました。システムをアップデートして、新しいm1n1を手に入れましょう!

DCPについて話している間、Appleデバイスは最近、DCPを使用してシステムを侵害する高度なマルウェアのターゲットとなりました。私たちがアサヒLinuxのDCPドライバを開発したとき（まだ出荷されていません）、私たちはすでにDCPを潜在的に危険で敵対的であるとみなして開発し、さらに、DCPを悪用できるような方法でユーザー空間に直接公開することもしていません。なお、Asahi Linux では、macOS/iOS と同じ脆弱性のある DCP ファームウェアを使用していますが、本脆弱性は影響しませんのでご安心ください。

## DARTの流用

M1 ProとM1 Max/Ultraは、新しいDART（IOMMU）ハードウェアのバリエーションも導入し、Thunderboltポートやメディアコーデック関連のハードウェアの一部に使用しているそうです。Rはすでにこれをリバースエンジニアリングしていましたが、まだ適切なドライバがありませんでした。M2 がこの IOMMU バリアントをずっと使っていることがわかったので、M2 を立ち上げる過程で m1n1 と Linux のサポートを追加しました。これだけでは、M1 Pro/Max で新しいものが動くようにはなりませんが、Thunderbolt とメディアハードウェアの要件は満たしていますので、これでまた一つボックスが増えました。

同じテーマで、Jannau は、Pro/Max の上流にある他の DART バリアントのサポートを追加するパッチセットを提出しました。私たちは、Asahi フォークでこれらのチップのサポートをしてきましたが、アップストリームのメンテナが、ARM IOMMU ページテーブルコードに Apple 固有のページテーブルの癖を加えることに完全に満足していないため、アップストリームがブロックされていました。これを別のファイルに移動することで、ブロックが解除され、最終的に M1 Pro/Max のサポートをアップストリームで得られることを期待しています（少なくとも、アップストリーム Linux が既に M1 で起動できるように、このチップで起動するには十分です；もちろん、フル機能のためにアップストリームすべき他のドライバーがまだ残っています）。

## U-Boot の更新

U-Boot のサポートは、順調に進んでいます。U-Boot 2022.07 がリリースされ、M1 モデル (M1 Pro/Max/Ultra を含む) を (ほぼ) 完全にサポートし、M2 の基本サポートも (Janne Grunau のおかげで) 行われました。最も重要なのは、Mac mini のタイプ A ポートの後ろにある PCIe USB3 コントローラのサポートです。

## Linux よりも

OpenBSD 7.1が2022年4月21日にリリースされ、M1およびM1 Pro/Max/Ultraに対応しました。これには、NVMe、USB（USB 2.0速度はタイプCポートのみ）、WiFi、Ethernet（M1 miniとiMac）、キーボードとタッチパッド（ノートパソコン）のサポートが含まれます。X11は初期のフレームバッファで動作し、オーディオのサポートは現在作業中です。OpenBSD は、まだ新しい M2 モデルをサポートしていませんが、これは 11 月にリリースが予定されている OpenBSD 7.2 のためのレーダーなのです。

OpenBSD は、U-Boot をペイロードとする m1n1 のみをインストールするオプションを提供する Asahi インストーラに依存しています。これにより、OpenBSD/arm64 がサポートする他のハードウェアと同じように、 標準的な UEFI ブート環境が得られます。この時点で、あなたは、単に minirootXX.img や installXX.img を USB ドライブに書き込んで、それを Mac に接続し、 OpenBSD のインストーラを起動することができるようになるのです。

OpenBSD のインストーラは、魔法の Apple パーティションについて知っているので、 インストーラでデフォルトの (W) hole ディスクオプションを選択しても、 システムパーティションは生き残り、あなたの macOS のインストールは安全です。OpenBSD のインストーラは、WiFi ファームウェアも自動的にピックアップします (EFI システムパーティションから。Asahi のインストーラはこれを準備します)。これは、インストール中に WiFi が利用できることを意味します。

## もう1つ...

みなさんが何を考えているかわかりますが...GPUはどうなるのでしょうか？

[埋め込みツイート](https://twitter.com/LinaAsahi/status/1537828477352615936)

いい知らせです。数ヶ月前、朝日リナが私たちのチームに加わり、M1 GPUハードウェア・インターフェースのリバースエンジニアリングとそのためのドライバ作成に挑戦してくれました。この短期間に、彼女は、既存のMesaの作業の上に構築して、実際のグラフィックス・アプリケーションやベンチマークを実行するのに十分なプロトタイプドライバをすでに構築しています。概念実証では、USB 接続経由で m1n1 を使用し、ドライバをリモートで実行するため、USB 帯域幅がボトルネックとなりますが、彼女は、GPU が GLMark2 フォン シェード バニー シーンを解像度 1080p で 1000FPS 以上で適切にレンダリングすることも実証しています。この完全なオープンソーススタックは、dEQP-GLES2テストスイートの94%に合格しています。悪くないですね。

近いうちに、LinaによるGPUドライバの全容を紹介するゲスト投稿を掲載する予定です。ご期待ください。もし興味があれば、YouTubeで彼女の仕事をフォローすることができます。

#### marcan 2022-07-18