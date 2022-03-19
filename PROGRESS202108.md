[Progress Report: August 2021](https://asahilinux.org/2021/08/progress-report-august-2021/)の非公式日本語訳です。

訳注: 
- 原文で本家文書wikiへのリンクは対応する日本語訳文書wikiへのリンクに置き換え
- Githubのmarkdownで使えないiframeでのtwitterへの埋め込みをリンクに置き換え

---
# 進捗報告:2021年8月
- [前回](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS20210102.md)
- [次回](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS202109.md)

前回の更新から随分と時間が経ってしまいました！正直なところ、第1回進歩報告はハードルが高すぎて、それに見合うだけのマンスリーレポートを
作成するのは難しいと思いました。そのため、今後は月1回のペースを維持しつつ、より短い内容のレポートを提供していきたいと思います。

とはいえ、この数ヶ月間に様々なことがあったので、今回はより大きなアップデートをお届けします！

## コアをLinux 5.13上流へ

[第1回進捗報告](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS20210102.md)でお伝えしたコアbring-up作業は
6月27日にLinux 5.13の一部として上流に入りリリースされました。
この初期段階ではエンドユーザーにとってあまり役立つものではありませんが、上流のカーネルコミュニティに受け入れられる方法で基盤の構築と
いくつかの難しい問題を解決する方法を提示した何ヶ月もの成果を意味します。また、カーネル・コミュニティのメンバーが私たちのプロジェクトに
興味を持ってくれました。これは重要なことです。カーネルのベテランたちと良好な関係を築くことは、
開発を進めていく上で、上流で物事を維持するために作業を確実していくために非常に重要なことだからです。

## m1n1ハイパーバイザによるハードウェアのリバースエンジニアリング

M1には、文書化されていない特注のハードウェアが多数存在するため、リバースエンジニアリングの大きな課題となっています。
ハードウェアのリバースエンジニアリングには、Appleの割り込みコントローラのリバースエンジニアリングに用いられたような
ブラインドプロービングという手法がありますが、これは複雑なハードウェアにはあまり有効ではありません。

ハードウェアを制御する方法を正しく理解するためには、唯一存在する文書であるmacOS自体を見なければなりません。
技術的にはmacOSドライバを逆アセンブルしてリバースエンジニアリングすることも可能ですが、これには法的な問題があり、
私たちのプロジェクトの著作権が脅かされる可能性があります。また、コードの多くはmacOSドライバフレームワークに固有のものであり、
ハードウェアに関する有用な情報を得ることができず効率が悪くなります。

代わりに、[Nouveau](https://nouveau.freedesktop.org/MmioTrace.html)のようなプロジェクトが過去に使用してきた
より安全なアプローチは公式ドライバが実際のシステム上で実行するハードウェアアクセスのログを記録することです。実際にコードを
見たりしません。NouveauはNvidiaの公式Linuxドライバによるアクセスを傍受するLinuxドライバを使って実現しました。
もちろん、AppleのM1ドライバはLinuxではなくmacOS用です。私たちはmacOSカーネルのオープンソースコアにカスタムパッチを
当てて同じアプローチで実装することもできましたが、それよりもさらに一歩進んで、本物のM1ハードウェアを透過的に提示するVMの中で
macOSの全体を変更せずに実行できるハイパーバイザーを構築することにしました。

これは仮想化されたハードウェアを備えたホストOSの上でゲストOSを実行するために設計された一般的な仮想マシンとは大きく異なります。
私たちのハイパーバイザーは、
[m1n1](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS20210102.md#%E3%83%8F%E3%83%BC%E3%83%89%E3%82%A6%E3%82%A7%E3%82%A2%E3%81%A7%E9%81%8A%E3%81%B6)
ブートローダーとハードウェア実験ツールをベースに構築されており、完全に特注の実装です。
ハイパーバイザーはゲストOSの邪魔をしないように設計されており、可能な限りベアメタルに近い環境でゲストOSを動作させる一方で、
ハードウェアへのアクセスを透過的に傍受して記録します。このようにして、macOSは通常通り完全に高速化されたデスクトップを利用する
本物のM1ハードウェアを『見る』ことができ、通常通りに操作することができます。

#### [埋め込みツイート](https://twitter.com/AsahiLinux/status/1397963184959418370)
(このツイートはハイパーバイザー上で動作するmacOSのSafariから投稿)

ハイパーバイザーはm1n1上に構築されているため、別のホストマシン上で動作するPythonコードと一緒に動作します。事実上、
PythonホストはM1とそのゲストOSをリモートで『操る』ことができます。ハイパーバイザー自体も部分的にPythonで書かれています！
これにより非常に高速なテストサイクルが可能となり、ゲストの実行中にハイパーバイザー自体の一部を再起動せずにライブで
更新することもできます。

ハイパーバイザーには標準的なデバッグツール（実行停止、シングルステップ、バックトレースの取得など）も含まれています。
m1n1自体やLinuxをハイパーパイザー上で実行できるので、リバースエンジニアリングに役立つだけでなく、ローレベルの
デバッグツールとしても利用できます。そう、m1n1上でm1n1を実行できるのです！

ハイパーバイザーの内部構造に興味のあるならば、私が
[3時間のコードレビューストリーム](https://www.youtube.com/watch?reload=9&v=igYgGH6PnOw)を行っています。
実装のほとんどだけでなく、ARMv8-A仮想化の一般的なトピックやM1固有の詳細や奇妙な点なども扱っています。

ハイパーバイザーの上に、柔軟なハードウェアI/O追跡フレームワークを構築し、特定のハードウェアがどのように動作するかを理解する
トレーサーをシームレスに読み込みアップロードできるようにしました。例えば、GPIO（汎用I/O）ハードウェアの
[トレーサー](https://github.com/AsahiLinux/m1n1/blob/c2c6da3df25c0605894244b4ea9387e882321efc/proxyclient/m1n1/trace/gpio.py)は、
macOSが各GPIOピンの状態や構成をいつ切り替えたか教えてくれます。これにより、生のレジスタの読み書きからより高度な機能まで
ハードウェアに関する理解を深めることができます。これは次に取り組んだハードウェアであるDCPにとって非常に重要なことでした。

## DCPのリバースエンジニアリング

Asahi Linuxの最大の課題はM1のGPUを動作させることです。一般に『GPU』と呼ばれるものは実際にはまったく異なる2つのハードウェアからなります。
メモリ上でフレームのレンダリングを担当するGPUプロパーと、レンダリングされたフレームをメモリからディスプレイに送信するディスプレイコントローラです。

Alyssa氏がGPUのユーザースペースコンポーネント（ドローコールからシェーダーまで）の
[リバースエンジニアリング](https://rosenzweig.io/blog/asahi-gpu-part-4.html)に尽力してきましたが、
メモリ管理やGPUへのコマンド送信を処理するハードウェアの最下位レベルはまだ見ていません。ただ、GPUを使って何かをレンダリングする前には、
それを画面に表示する方法が必要です！これまではファームウェアが提供するフレームバッファを使っていました。これは画面に表示する
ピクセルを書き込むための単なるメモリ領域で、これでは実際のデスクトップには対応できません。新しいフレームをティアリングせずに表示したり、
マウスカーソルなどのハードウェアスプライトをサポートしたり、解像度を切り替えて複数の出力を設定するなどの機能が必要です。
これがディスプレイコントローラーの仕事です。

多くのモバイルSoCでは、ディスプレイコントローラーはシンプルなレジスタを持つハードウェアに過ぎません。これはM1でも同様ですが、アップルは
これに一工夫を加えることにしました。ディスプレイエンジンにコプロセッサ（DCPと呼ばれる）を追加し、独自のファームウェア
（システムブートローダで初期化される）を実行し、ディスプレイドライバのほとんどをコプロセッサに移したのです。しかし、
自然なドライバーの境界ではなく...macOSのC++ドライバーの半分を取り出してDCPに移し、DCPリモートプロシージャコールの
インターフェースを作って、残りの半分が他のCPU上でC++オブジェクトのメソッドを呼び出すようにしたのです！
物事を複雑にしすぎているような...

これをリバースエンジニアリングするのは非常に難しい課題ですが、ハイパーバイザーのおかげでレイヤーごとに仕組みを理解していくことができます。
最下層では、DCPはアップルが『ASC』と呼ぶコプロセッサのインスタンスです（M1はたくさんのASCを搭載！）。ASCプロセッサは独自の
ファームウェアを実行し、メールボックスインターフェースを介してメインCPUと通信します。メールボックスインターフェースとは、
それぞれの側から64ビットのメッセージを相手に送ることができるシンプルなメッセージキューのことで、『endpoint』というタグが付けられています。

このシンプルなインターフェース上で、Appleは特注のRTOSであるRTKitを搭載したすべてのASCプロセッサに対して共有のendpointセットを
使用しています。このインターフェースは、syslogメッセージやクラッシュダンプをASCからメインCPUに送信したり、endpointを初期化する（サービス）などの
機能を提供します。そこで、これらのメッセージを理解し、ハイパーバイザーのコンソールに直接syslogメッセージを出力するなどの機能を持つ
[トレーサー](https://github.com/AsahiLinux/m1n1/blob/c2c6da3df25c0605894244b4ea9387e882321efc/proxyclient/m1n1/trace/asc.py)
を作りました。

さらに、DCPには複数のendpointが実装されており、そのうちの1つが『メイン』インターフェースとして機能しています。このインターフェース自体が
双方向の遠隔メソッドコールをサポートしています。DCPは呼び出しを受けた後メインCPUに同期コールバックを何度も発行し、メインCPUは
さらに同期DCPコールを発行することができます。つまり、実行コールスタックがCPUとDCPの境界を越えて拡張されるのです！
このインターフェースは非同期再入可能性(asynchronous reentrancy)にも対応しており、複数の『チャンネル』を持っているので、例えば、
DCPは別の動作中であってもいつでもメインCPUに非同期メッセージを送信することができます。

メソッドコール自体は共有メモリ上のバッファを介して引数を送信しデータを受信します。これらのバッファは、整数のような単純な型、入力や出力
または両方向にデータを渡すことができるポインタ、より複雑な固定構造、さらにはJSONのようなより高レベルのデータ構造のための2つの異なる
シリアライズフォーマット（IOKitの問題！）をエンコードします。私たちの
[dcpトレーサー](https://github.com/AsahiLinux/m1n1/blob/c2c6da3df25c0605894244b4ea9387e882321efc/proxyclient/hv/trace_dcp.py)
は、これらのバッファを取得してトレースファイルにダンプすることができ、プロトコルの理解を深めるためにオフラインで分析できるようになっています。

次に、このRPCプロトコルとマーシャリングシステムの
[Python実装](https://github.com/AsahiLinux/m1n1/blob/c2c6da3df25c0605894244b4ea9387e882321efc/proxyclient/m1n1/fw/dcp/ipc.py)
を構築しました。この実装は3つの目的があります。ハイパーバイザーからのDCPログを解析してmacOSの動作を理解することと、
[prototype DCPドライバ](https://github.com/AsahiLinux/m1n1/blob/c2c6da3df25c0605894244b4ea9387e882321efc/proxyclient/m1n1/fw/dcp/manager.py)
を完全にPythonで構築することと、そして将来的にはLinuxカーネルのDCPドライバ用のマーシャリングコードを自動生成するために使用することです。

ハイパーバイザーのDCPトレースをこのデコーダに通すことで、macOSとDCPの間でやり取りされるすべてのメソッドコールのトレースを得ることができます:

```
>C[0x0] A401 IOMobileFramebufferAP::start_signal()
  <d[0x0] D598 IOMobileFramebufferAP::find_swap_function_gated()
  >d[0x0] D598 IOMobileFramebufferAP::find_swap_function_gated()
  <d[0x0] D107 UnifiedPipeline2::create_provider_service()
  >d[0x0] D107 UnifiedPipeline2::create_provider_service() = True
  [...]
  <d[0x0] D000 UPPipeAP_H13P::did_boot_signal()
  >d[0x0] D000 UPPipeAP_H13P::did_boot_signal() = True
  <d[0x0] D001 UPPipeAP_H13P::did_power_on_signal()
  >d[0x0] D001 UPPipeAP_H13P::did_power_on_signal() = True
  <d[0x0] D116 UnifiedPipeline2::start_hardware_boot()
    >C[0x40] A357 UnifiedPipeline2::set_create_DFB()
    <C[0x40] A357 UnifiedPipeline2::set_create_DFB()
    >C[0x40] A443 IOMobileFramebufferAP::do_create_default_frame_buffer()
    <C[0x40] A443 IOMobileFramebufferAP::do_create_default_frame_buffer() = 0
    >C[0x40] A103 UPPipe2::test_control(cmd=0, arg=2863267840)
    <C[0x40] A103 UPPipe2::test_control(cmd=0, arg=2863267840) = 4638564681600
    >C[0x40] A029 UPPipeAP_H13P::setup_video_limits()
      <d[0x40] D107 UnifiedPipeline2::create_provider_service()
      >d[0x40] D107 UnifiedPipeline2::create_provider_service() = True
      <d[0x40] D401 ServiceRelay::sr_get_uint_prop(obj='PROV', key='minimum-frequency', value=0)
      >d[0x40] D401 ServiceRelay::sr_get_uint_prop(obj='PROV', key='minimum-frequency', value=0) = False
      <d[0x40] D107 UnifiedPipeline2::create_provider_service()
      >d[0x40] D107 UnifiedPipeline2::create_provider_service() = True
      <d[0x40] D408 ServiceRelay::sr_getClockFrequency(obj='PROV', arg=0)
      >d[0x40] D408 ServiceRelay::sr_getClockFrequency(obj='PROV', arg=0) = 533333328
      <d[0x40] D300 PropRelay::pr_publish(prop_id=38, value=470741)
      >d[0x40] D300 PropRelay::pr_publish(prop_id=38, value=470741)
      <d[0x40] D563 IOMobileFramebufferAP::setProperty_int(key='MaxVideoSrcDownscalingWidth', value=27582)
      >d[0x40] D563 IOMobileFramebufferAP::setProperty_int(key='MaxVideoSrcDownscalingWidth', value=27582) = True
      <d[0x40] D563 IOMobileFramebufferAP::setProperty_int(key='VideoClock', value=74250000)
      >d[0x40] D563 IOMobileFramebufferAP::setProperty_int(key='VideoClock', value=74250000) = True
      <d[0x40] D563 IOMobileFramebufferAP::setProperty_int(key='PixelClock', value=533333328)
      >d[0x40] D563 IOMobileFramebufferAP::setProperty_int(key='PixelClock', value=533333328) = True
    <C[0x40] A029 UPPipeAP_H13P::setup_video_limits()
    >C[0x40] A463 IOMobileFramebufferAP::flush_supportsPower(arg0=True)
    <C[0x40] A463 IOMobileFramebufferAP::flush_supportsPower(arg0=True)
    >C[0x40] A036 UPPipeAP_H13P::apt_supported()
    <C[0x40] A036 UPPipeAP_H13P::apt_supported() = False
    >C[0x40] A000 UPPipeAP_H13P::late_init_signal()
      <d[0x40] D003 UPPipeAP_H13P::rt_bandwidth_setup_ap(config=<out>)
      [...]
    <C[0x40] A000 UPPipeAP_H13P::late_init_signal() = True
    >C[0x40] A460 IOMobileFramebufferAP::setDisplayRefreshProperties()
      <d[0x40] D561 IOMobileFramebufferAP::setProperty_dict(key='IOMFBDisplayRefresh', value=...)
        value = {'displayMinRefreshInterval': 71582788,
                 'displayMinRefreshIntervalMachTime': 399984,
                 'displayMaxRefreshInterval': 178956970,
                 'displayMaxRefreshIntervalMachTime': 999984,
                 'displayRefreshStep': 0,
                 'displayRefreshStepMachTime': 0}
      >d[0x40] D561 IOMobileFramebufferAP::setProperty_dict(key='IOMFBDisplayRefresh', value=...) = True
    <C[0x40] A460 IOMobileFramebufferAP::setDisplayRefreshProperties() = True
  >d[0x0] D116 UnifiedPipeline2::start_hardware_boot() = True
  <d[0x0] D122 UnifiedPipeline2::setDCPAVPropStart(length=5821)
[... thousands more lines of initialization and kilobytes of serialized data ...]
```

そしてすべてがどのように組み合わされているかという知識を得ることで、ディスプレイバッファスワップを最終的にディスプレイに
送信するための十分なインターフェースを実装することが可能になります。これにより、ダブルバッファリングによるティアフリーなグラフィックス、
ハードウェアによるマウスポインタのアクセラレーション、フレームバッファの拡大縮小と合成などを実装することができます！

![image](https://asahilinux.org/img/blog/2021/08/asahi_dcp_coming_soon.png)
試作したDCPドライバーを使った最初のフレーム（スクリーンショットはHDMIキャプチャーで撮影したもの）

さらなる工夫について、DCPのインターフェースは安定しておらずmacOSのバージョンごとに変更されています！
これでようやくAsahi Linuxプロジェクトの疑問に答えが出ました。私たちは特定のバージョンのファームウェアのみに対応します。
『ペア』のファームウェアのみ対応しリリースごとに変更する余裕のあるmacOSとは異なり、Linuxではファームウェアを
アップグレードせずにカーネルをアップグレードできるようにするためには最初にサポートされたものから遡ってすべてのバージョンの
ファームウェアに対応する必要があります。AppleがリリースするすべてのDCPファームウェアバージョンをサポートしようとするのは
メンテナンス上悪夢なので、代わりに、Linuxが対応すると祝福(bless)されている特定の『ゴールデン』ファームウェアバージョンを
選択します。心配しないでください。macOSをアップグレードできないのではありません。このファームウェアはシステムごとではなく
OSごとに提供されているので、Linuxは対になるmacOSのインストールとは異なるファームウェアバンドルを使用することができます。

初期のカーネルDCP対応にはmacOS 12 『Monterey』（現在パブリックベータ版）でリリースされたファームウェアが必要になると予想しています。
タイミングによっては、12.0またはそれより新しいポイントリリースが必要になるかもしれません。バグフィックスや新しいハードウェアへの対応など、
必要に応じて新しいバージョンのファームウェアを追加していきます。

この複雑なDCPインターフェースを使用することの利点はDCPが実際に膨大な量のコードを実行しているということです - DCPのファームウェアは
7MB以上もあります！DisplayPortリンクトレーニング、リアルタイムのメモリ帯域幅計算、Mac miniのDisplayPort-HDMIコンバーターの処理、
有効なビデオモードの列挙とモード切り替えの実行など、複雑なアルゴリズムを実装しています。これらをすべてリバースエンジニアリングして
生のハードウェアドライバとして実装するのは大変な作業なので、最終的にはこのハイレベルインターフェイスを使用して、汚れ仕事はすべてDCPに任せる方が
時間の節約になるでしょう。特に、新しいApple Silicon チップは共有のDCPコードベースを使用し、どのmacOSバージョンでも同じ
インターフェイスを使用している可能性が高いので、比較的小さいDCPファームウェアABIのアップデートだけで新しいチップを
『ただで(for free)』サポートすることができます。

ちなみに独自のファームウェアを書くことはできません。ファームウェアはOSに制御を移す前にiBootによってロードされ、Appleによって署名されているからです。
しかし、生のディスプレイ・コントローラー・レジスタにアクセスできるので、DCPファームウェア自体では非対応の機能やトリックを実装する余地が
あります。また、ファームウェアはIOMMUの背後で安全に動作しているので、Appleを信頼しないですむことを意味します。
（悪名高いIntel Management Engineとは異なり）ファームウェアがシステムを危険にさらしたり、バックドアを仕掛けたりすることはできません。

## インストーラーの作成

m1n1やLinuxパッチを自分で試してみたいという冒険心のある方は、
[開発者向けのクイックスタートガイド](https://github.com/asfdrwe/asahi-linux-translations/wiki/%E9%96%8B%E7%99%BA%E8%80%85%E5%90%91%E3%81%91%E3%82%AF%E3%82%A4%E3%83%83%E3%82%AF%E3%82%B9%E3%82%BF%E3%83%BC%E3%83%88)
をご覧になったことがあると思います。そこには膨大で退屈な手動インストールのプロセスが書かれています。

Apple SiliconマシンでOSを起動するためには本物のmacOSをインストールしたように『見えなければなりません』。つまり、APFSコンテナの中に
複数のボリュームがあり、その中に特定のディレクトリ構造やファイルが含まれていなければならないのです。これまでは、macOSを実際に別の
パーティションに2回目のインストールを行い、そのカーネルをm1n1に置き換えるのが最もシンプルな方法でした。これは言うまでもなく
インストールに時間がかかるため非常に面倒な作業です。また、アップグレード可能なmacOSのインストールに必要な約70GBのディスクスペースが
無駄になります。また、特定のバージョンのmacOSをインストールすることも難しくなります。これは、特定のファームウェアバンドルの使用を
求めるようになったときに問題となるでしょう。これでは初期開発以上のものには明らかになりません。

必要なのは、Appleのツールが起動可能なOSとして認識するために必要なブートローダコンポーネントとメタデータを含む『macOS』のインストールを
ゼロから作成する方法です。実際のmacOSのrootファイルシステムを含む必要はありませんが、以下のコンポーネントは必要です:

- バージョン情報とメタデータのファイル
- iBoot2とその構成（DeviceTreeを含む)
- OSに対になるファームウェア（DCP、AGXなど）
- macOSのリカバリー環境（約1GBのmacOSシステムイメージ）
- カスタムカーネルとしてインストールされmacOS XNUカーネルをオーバーライドするm1n1
- リカバリ操作のためのユーザー認証に関連するファイル

これを行うために、macOSのインストールプロセスのこの部分を再現し、ユーザーにm1n1のインストールプロセスを案内できるプロトタイプの
インストーラーを作成しました。このインストーラは、Apple Silicon上でmacOSのリリースごとに公開されているAppleの
[公式復元イメージ](https://mrmacintosh.com/apple-silicon-m1-full-macos-restore-ipsw-firmware-files-database/)を
使用しています。13GB以上のOSフルイメージをダウンロードするのではなく、必要なコンポーネント（最大のものは1GBのリカバリーイメージ）のみを
オンデマンドでストリーミングします。

インストーラを起動するために、ユーザーはmacOSまたはリカバリーモードからシェルスクリプト（curl | shスタイル）を実行しプロンプトに従ってください:

```
# curl https://..... | sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   564  100   564    0     0  70500      0 --:--:-- --:--:-- --:--:-- 70500

Bootstrapping installer:
  Downloading...
  Extracting...
  Initializing...


Welcome to the Asahi Linux installer!

This installer is in a pre-alpha state, and will only do basic
bootloader set-up for you. It is only intended for developers
who wish to help with Linux bring-up at this point.

Please make sure you are familiar with our documentation at:
  https://alx.sh/w

Press enter to continue.


Collecting system information...
System information:
  Product name: Mac mini (M1, 2020)
  SoC: Apple M1
  Device class: j274ap
  Product type: Macmini9,1
  Board ID: 0x22
  Chip ID: 0x8103
  System firmware: iBoot-7429.30.8.0.4
  Boot UUID: DCBCA6BD-BFF1-4F8F-AE1A-6E937D2D4BDC
  Boot VGID: DCBCA6BD-BFF1-4F8F-AE1A-6E937D2D4BDC
  Default boot VGID: DCBCA6BD-BFF1-4F8F-AE1A-6E937D2D4BDC
  Boot mode: macOS
  OS version: 11.5.2
  Login user: marcan

Collecting partition information...
  System disk: disk0

Collecting OS information...

Partitions in system disk (disk0):
  1: APFS [Macintosh HD] (112.91 GB, 6 volumes)
    OS: [B*] macOS v11.5.2 [DCBCA6BD-BFF1-4F8F-AE1A-6E937D2D4BDC]
  2: APFS [macOS 12] (70.00 GB, 6 volumes)
    OS: [  ] macOS v12.0 [985C5BA1-4C81-4095-B972-99F7E8AB4CE5]
  3: (free space: 65.19 GB)

  [B ] = Booted OS, [R ] = Booted recovery, [? ] = Unknown
  [ *] = Default boot volume

Choose what to do:
  f: Install Asahi Linux into free space
  q: Quit without doing anything
Action (q): f

Using OS 'Macintosh HD' (disk0s2) for machine authentication.

Choose a free area to install into:
  3: (free space: 59.69 GB)
Target area: 3

Enter a name for your OS (Linux): Asahi Linux 

Choose the macOS version to use for boot firmware:
(If unsure, just press enter)
  1: 11.4
  2: 11.5.2
Version (2): 

Using macOS 11.5.2

Downloading OS package info...
.

Creating new stub macOS named Asahi Linux

Installing stub macOS into disk0s6 (Asahi Linux)
Preparing target volumes...
Checking volumes...
Beginning stub OS install...
++
Setting up System volume...
Setting up Data volume...
Setting up Preboot volume...
++++++++++++++++++
Setting up Recovery volume...
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Stub OS installation complete.

When the Startup Disk screen appears, choose 'Asahi Linux'.
You will have to authenticate yourself.

Press enter to continue.


The system will now shut down.
To complete the installation, perform the following steps:

1. Press and hold down the power button to power on the system.
   * It is important that the system be fully powered off before this step,
     and that you press and hold down the button once, not multiple times.
     This is required to put the machine into the right mode.
2. Release it once 'Entering startup options' is displayed.
3. Choose Options.
4. Click on the Utilities menu and select Terminal.
5. Type the following command and follow the prompts:

/Volumes/'Asahi Linux'/step2.sh

Press enter to shut down the system.
```
これは最終的に私たちの短いドメイン https://alx.sh で利用できるようになりますが、まだそこには配備されていません。
これはm1n1だけをインストールするインストーラーの最初の試作品で、私たちのクエストに参加したいと思っている開発者にとっては
非常に便利なものです。最終的には、よりユーザーフレンドリーなバージョンとして、Linux用にドライブを分割したり、スペースを
確保するためにmacOSのサイズを変更したり、好みのディストリビューションをインストールする方法も案内する予定です。将来的には
グラフィカルなmacOSアプリになるかもしれませんね！

![image](https://asahilinux.org/img/blog/2021/08/asahi_bootpicker.png)
内蔵のブートピッカーを使って、2つのバージョンのmacOSとAsahi Linuxをトリプルブートすることが可能

## カーネルドライバーの追加

Sven氏がM1のIOMMU(I/O Memory Management Unit、I/Oメモリ管理ユニット)である
[DART](https://github.com/asfdrwe/asahi-linux-translations/wiki/%E7%94%A8%E8%AA%9E%E9%9B%86#d)の
Linuxドライバをひたすら開発しています。
このドライバは、PCIe、USB、DCPなど、あらゆる種類のハードウェアを動作させるために必要です。
このドライバは上流に承認されLinux 5.15に搭載されることになりました！

このドライバが入ったことで、最小限の追加パッチとドライバでUSBとPCIeを動作させることができるようになりました。他にも様々な依存関係
（雑多なもののためのGPIO、適切なUSB-PD対応のためのI²C、ラップトップでのタッチパッド/キーボードサポートのためのSPI、
NVMe対応のパッチ）があり、それらは人々が取り組む様々なツリーに散らばっております。次に、私たちはこれらのシンプルな
ドライバーを仕上げ、開発を継続し新しい開発者に安定した基盤を提供するために使用できるクリーンで実用的なリファレンスツリーを
まとめることに注力します。現在の状態でAsahi Linuxを（非アクセラレータの）GUIを備えた開発マシンとして使用することができますが、
まだ粗削りです。(技術的には単純な)ドライバをどう実装するか、お役所的な部分もありますので上流に入るのにはもう少し時間がかかりますが、
それほど時間はかからないでしょう。

これができたら...次はGPUカーネルドライバに取り組みましょう! これからが楽しみです :-)

#### marcan - 2021-08-14
