[Progress Report: August 2021](https://asahilinux.org/2021/08/progress-report-august-2021/)の非公式日本語訳です。

---
# 進歩報告:2021年8月
- [前回](https://asahilinux.org/2021/03/progress-report-january-february-2021/)
- [次回](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS202109.md)

前回の更新から随分と時間が経ってしまいました。正直なところ、第1回進歩報告はハードルが高すぎて、それに見合うだけのマンスリーレポートを
作成するのは難しいと思いました。そのため、今後は月1回のペースを維持しつつ、より短い内容のレポートを提供していきたいと思います。

とはいえ、この数ヶ月間にはいろいろなことがありましたので、今回はより大きなアップデートをお届けします。

## コアの成果をLinux 5.13アップストリームへ

第1回進捗報告でお伝えしたコアの引き上げ作業は、6月27日にLinux 5.13の一部としてアップストリームリリースされました。
この初期段階では、エンドユーザーにとってはあまり便利ではありませんが、何ヶ月にもわたって基礎を固め、アップストリームの
カーネルコミュニティに受け入れられる方法で、特定の難しい問題を解決する方法を考えてきたことを意味します。また、カーネル・コミュニティの
メンバーが私たちのプロジェクトに興味を持ってくれました。これは重要なことです。カーネルのベテランたちと良好な関係を築くことは、
開発を進めていく上で、アップストリームを維持するために協力していく上で非常に重要なことなのです。

## m1n1ハイパーバイザによるハードウェアリバースエンジニアリング

M1には、文書化されていない特注のハードウェアが多数存在するため、リバースエンジニアリングの大きな課題となっています。
ハードウェアのリバースエンジニアリングには、Appleのインタラプトコントローラのリバースエンジニアリングに用いられたような
ブラインドプロービングという手法がありますが、これは複雑なハードウェアにはあまり有効ではありません。

ハードウェアの駆動方法を正しく理解するためには、唯一存在するドキュメントであるmacOS自体を見なければなりません。
技術的にはmacOSドライバを逆アセンブルしてリバースエンジニアリングすることも可能ですが、これには法的な問題があり、
私たちのプロジェクトの著作権が脅かされる可能性があります。また、コードの多くはmacOSドライバフレームワークに固有のものであり、
ハードウェアに関する有用な情報を得ることができないため、効率が悪くなります。

代わりに、Nouveau のようなプロジェクトが過去に使用してきたより安全なアプローチは、実際にコードを見ることなく、
公式ドライバが実際のシステム上で実行するハードウェア アクセスのログを記録することです。ヌーボーは、Linuxドライバを使って、
Nvidiaの公式Linuxドライバによるアクセスを傍受することでこれを実現しました。もちろん、AppleのM1ドライバーはLinuxではなく
macOS用です。私たちは、macOSカーネルのオープンソースコアにカスタムパッチを当てて同じアプローチを実装することもできましたが、
それよりもさらに一歩進んで、本物のM1ハードウェアを透過的に提示するVMの中で、macOSの全体を変更せずに実行できるハイパーバイザーを
構築することにしました。

これは、仮想化されたハードウェアを備えたホストOSの上でゲストOSを実行するために設計された一般的な仮想マシンとは大きく異なります。
当社のハイパーバイザーは、当社のm1n1ブートローダーとハードウェア実験ツールをベースに構築されており、完全にオーダーメイドの実装です。
ハイパーバイザーは、ゲストOSの邪魔をしないように設計されており、可能な限りベアメタルに近い環境でゲストOSを動作させる一方で、
ハードウェアへのアクセスを透過的に傍受して記録します。このようにして、macOSは本物のM1ハードウェアを「見る」ことができ、通常通りに
操作することができ、完全に高速化されたデスクトップを利用することができます。

[参照ページ](https://twitter.com/AsahiLinux/status/1397963184959418370)
このツイートは、ハイパーバイザー上で動作するmacOSのSafariから投稿されたものです。


ハイパーバイザーはm1n1上に構築されているため、別のホストマシン上で動作するPythonコードと一緒に動作します。事実上、
Pythonホストは、M1とそのゲストOSをリモートで「操る」ことができます。ハイパーバイザー自体も部分的にPythonで書かれています。
これにより、非常に高速なテストサイクルが可能となり、ゲストの実行中にハイパーバイザー自体の一部を再起動せずにライブで更新することもできます。

ハイパーバイザーには標準的なデバッグツール（実行停止、シングルステップ、バックトレースの取得など）も含まれています。これにより、
リバースエンジニアリングに役立つだけでなく、m1n1自体やLinuxもハイパーバイザー上で実行できるため、低レベルのデバッグツールとしても
利用できます。そう、m1n1上でm1n1を走らせることができるのです。

ハイパーバイザーの内部構造に興味のある方は、私が3時間のコードレビューストリームを行い、実装のほとんどをカバーし、ARMv8-A仮想化の
一般的なトピック、M1固有の詳細や奇妙な点などもカバーしました。

ハイパーバイザーの上に、柔軟なハードウェアI/Oトレーシングフレームワークを構築し、特定のハードウェアがどのように動作するかを理解する
トレーサーをシームレスにロード、アップロードできるようにしました。例えば、GPIO（General Purpose I/O）ハードウェアのトレーサーは、
macOSが各GPIOピンの状態や構成をいつ切り替えたかを教えてくれます。このようにして、生のレジスタの読み書きからより高度な機能まで、
ハードウェアに関する理解を深めることができます。これは、次に取り組んだハードウェアであるDCPにとって非常に重要なことでした。

## DCPのリバースエンジニアリング

アサヒリナックスの最大の課題は、M1のGPUを動作させることでした。一般に「GPU」と呼ばれるものは、実際にはまったく異なる2つのハードウェアです。
メモリ上でフレームのレンダリングを担当するGPUプロパーと、レンダリングされたフレームをメモリからディスプレイに送信するディスプレイコントローラです。

アリッサは、GPUのユーザースペースコンポーネント（ドローコールからシェーダーまで）のリバースエンジニアリングに尽力してきましたが、
メモリ管理やGPUへのコマンド送信を処理するハードウェアの最下位レベルにはまだ目を向けていません。しかし、GPUを使って何かをレンダリングする前には、
それを画面に表示する方法が必要です。これまでは、ファームウェアが提供するフレームバッファを使っていました。フレームバッファは、画面に表示する
ピクセルを書き込むための単なるメモリ領域ですが、これでは実際のデスクトップには対応できません。新しいフレームをティアリングせずに表示したり、
マウスカーソルなどのハードウェアスプライトをサポートしたり、解像度を切り替えて複数の出力を設定するなどの機能が必要です。これが
ディスプレイコントローラーの仕事です。

多くのモバイルSoCでは、ディスプレイコントローラーは単純なレジスタを持つハードウェアに過ぎません。これはM1でも同様ですが、アップルはこれに
一工夫を加えることにしました。ディスプレイエンジンに、独自のファームウェア（システムブートローダで初期化される）を実行するコプロセッサ
（DCPと呼ばれる）を追加し、ディスプレイドライバのほとんどをコプロセッサに移したのです。しかし、自然なドライバーの境界ではなく......
macOSのC++ドライバーの半分をDCPに移し、リモートプロシージャコールのインターフェースを作って、それぞれがもう一方のCPUのC++オブジェクトの
メソッドを呼び出せるようにしたのです。物事を複雑にしすぎているといえばそうかもしれません。

これをリバースエンジニアリングするのは大変な作業ですが、ハイパーバイザーのおかげで、レイヤーごとに仕組みを理解していくことができます。
最下層のDCPは、アップルが「ASC」と呼ぶ、コプロセッサのインスタンスです（M1には約12個のASCが搭載されています）。ASCプロセッサは独自の
ファームウェアを実行し、メールボックスインターフェースを介してメインCPUと通信します。メールボックスインターフェースとは、それぞれの側が
64ビットのメッセージを相手に送ることができるシンプルなメッセージキューのことで、「エンドポイント」というタグが付けられています。

Appleは、このシンプルなインターフェースの上に、Apple独自のRTOSであるRTKitを搭載したすべてのASCプロセッサに対して、共有のエンドポイントを
使用しています。このインターフェースは、syslogメッセージやクラッシュダンプをASCからメインCPUに送信したり、エンドポイント（サービス）を
初期化するなどの機能を提供します。そこで、これらのメッセージを理解し、ハイパーバイザーのコンソールに直接syslogメッセージを出力するなどの
機能を持つトレーサーを作りました。

さらに、DCPには複数のエンドポイントが実装されており、そのうちの1つが「メイン」インターフェースとして機能しています。このインターフェース自体が、
双方向のリモートメソッドコールをサポートしています。DCPはしばしば、呼び出しを受けた後にメインCPUに同期コールバックを発行し、メインCPUは
さらに同期DCPコールを発行することができます。つまり、実行コールスタックはCPUとDCPの境界を越えて拡張されるのです。このインターフェースは、
非同期リエントランシーもサポートしており、複数の「チャンネル」を持っているので、例えば、DCPは別の動作中であっても、いつでもメインCPUに
非同期メッセージを送信することができます。

メソッドコール自体は、共有メモリ上のバッファを介して、引数と戻りデータを送信します。これらのバッファは、整数のような単純な型、入力、出力、
または両方向にデータを渡すことができるポインタ、より複雑な固定構造、さらにはJSONのような高レベルのデータ構造のための2つの異なる
シリアライズフォーマット（IOKitを非難）をエンコードします。私たちのdcpトレーサーは、これらのバッファを取得してトレースファイルにダンプし、
プロトコルの理解を深めるためにオフラインで分析できるようにしています。

次に、このRPCプロトコルとマーシャリングシステムのPython実装を構築しました。この実装は、ハイパーバイザーからのDCPログを解析してmacOSの
動作を理解すること、DCPドライバのプロトタイプを完全にPythonで構築すること、そして将来的にはLinuxカーネルのDCPドライバ用のマーシャリングコードを
自動生成するために使用すること、という3つの目的を持っています。

ハイパーバイザーのDCPトレースをこのデコーダに通すことで、macOSとDCPの間でやり取りされるすべてのメソッドコールのトレースを得ることができます。

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

そして、すべてがどのように組み合わされているかという知識を得て、ディスプレイバッファのスワップを最終的にディスプレイに
送信するための十分なインターフェースを実装することができます。これにより、ダブルバッファリングによるティアフリーグラフィックス、
ハードウェアによるマウスポインタのアクセラレーション、フレームバッファのスケーリングとコンポジットなどを実装することができます。

![image](https://asahilinux.org/img/blog/2021/08/asahi_dcp_coming_soon.png)
試作したDCPドライバーを使った最初のフレーム（スクリーンショットはHDMIキャプチャーで撮影したもの）

さらに、DCPのインターフェースは安定しておらず、macOSのバージョンごとに変更されています。これでようやく、Asahi Linuxプロジェクトの
疑問が解けました。それは、特定のバージョンのファームウェアしかサポートしないということです。ペアのファームウェアだけをサポートし、
リリースごとに変更する余裕のあるmacOSとは異なり、Linuxでは、ファームウェアをアップグレードせずにカーネルをアップグレードできるように
するために、最初にサポートされたものから遡って、すべてのバージョンのファームウェアをサポートする必要があります。Appleがリリースする
すべてのDCPファームウェアバージョンをサポートしようとするのは、メンテナンス上の悪夢ですので、代わりに、Linuxでサポートされることが
祝福されている特定の「ゴールデン」ファームウェアバージョンを選びます。心配しないでください。macOSをアップグレードできないということでは
ありません。このファームウェアは、システムごとではなく、OSごとに提供されているので、Linuxは、兄弟であるmacOSのインストールとは異なる
ファームウェアバンドルを使用することができます。

初期のカーネルDCPのサポートには、macOS 12 "Monterey"（現在パブリックベータ版）でリリースされたファームウェアが必要になると予想しています。
タイミングによっては、12.0またはそれより新しいポイントリリースが必要になるかもしれません。バグフィックスや新しいハードウェアへの対応など、
必要に応じて新しいバージョンのファームウェアを追加していきます。

この複雑なDCPインターフェースを使用することの利点は、DCPが実際に膨大な量のコードを実行しているということです - DCPのファームウェアは
7MBを超えています。DCPのファームウェアは7MB以上もあります！DisplayPortリンクトレーニング、リアルタイムのメモリ帯域幅計算、Mac miniの
DisplayPort-HDMIコンバーターの処理、有効なビデオモードの列挙とモード切り替えの実行など、複雑なアルゴリズムを実装しています。これらを
すべてリバースエンジニアリングして生のハードウェアドライバとして実装するのは大変な作業なので、最終的にはこの上位のインターフェイスを使い、
汚れ仕事はすべてDCPに任せる方が時間の節約になる可能性が高いです。特に、新しいApple Silicon製チップは、共有のDCPコードベースを使用し、
どのmacOSバージョンでも同じインターフェイスを使用している可能性が高いので、比較的マイナーなDCPファームウェアABIのアップデートだけで、
新しいチップを「無料で」サポートすることができます。

ファームウェアは、OSに制御を移す前にiBootによってロードされ、Appleによって署名されているため、独自のファームウェアを書くことはできません。
しかし、生のディスプレイ・コントローラー・レジスタにアクセスできるので、DCPファームウェア自体がサポートしていない機能やトリックを実装する余地が
あります。また、ファームウェアはIOMMUの背後で安全に動作します。つまり、Appleを信頼する必要はなく、
（悪名高いIntel Management Engineとは異なり）システムを危険にさらしたり、バックドアを仕掛けたりすることはできません。

## インストーラーの作成

m1n1やLinuxパッチを自分で試してみたいという冒険心のある方は、開発者向けのクイックスタートガイドをご覧になったことがあると思います。
そこには、膨大で退屈な手動インストールのプロセスが書かれています。

アップルのシリコンマシンでOSを起動するためには、本物のmacOSをインストールしたように見えなければなりません。つまり、APFSコンテナの中に
複数のボリュームがあり、その中に特定のディレクトリ構造やファイルが含まれていなければならないのです。これまでは、macOSを実際に別の
パーティションに2回目のインストールを行い、そのカーネルをm1n1に置き換えるのが最もシンプルな方法でした。これは言うまでもなく、
インストールに時間がかかるため、非常に面倒な作業です。また、アップグレード可能なmacOSのインストールに必要な約70GBのディスクスペースが
無駄になります。また、特定のバージョンのmacOSをインストールすることも難しくなります。これは、特定のファームウェアバンドルの使用を
求めるようになったときに問題となるでしょう。これでは、初期の開発以上のものには明らかに対応できません。

必要なのは、Appleのツールが起動可能なOSとして認識するために必要なブートローダコンポーネントとメタデータを含む「macOS」のインストールを
ゼロから作成する方法です。実際のmacOSルートファイルシステムを含む必要はありませんが、これらのコンポーネントは必要です。

    バージョン情報とメタデータのファイル
    iBoot2とその構成（デバイスツリーを含む
    OSに対応したファームウェア（DCP、AGXなど）。
    macOSのリカバリー環境（約1GBのmacOSシステムイメージ）
    カスタムカーネルとしてインストールされたm1n1（macOS XNUカーネルをオーバーライドする
    リカバリ操作のためのユーザー認証に関連するファイル

このため、macOSのインストールプロセスのこの部分を再現し、ユーザーにm1n1のインストールプロセスを案内することができるプロトタイプの
インストーラーを作成しました。このインストーラは、Apple Silicon上でmacOSのリリースごとに公開されているAppleの公式復元イメージを
使用しています。13GB以上のOSフルイメージをダウンロードするのではなく、必要なコンポーネント（最大のものは1GBのリカバリーイメージ）のみを
オンデマンドでストリーミングします。

インストーラを起動するには、ユーザーはmacOSまたはリカバリーモードからシェルスクリプト（curl | shスタイル）を実行し、プロンプトに
従うだけです。

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
これはm1n1だけをインストールするインストーラーの最初の試作品で、私たちの探求に参加したいと思っている開発者にとっては
非常に便利なものです。最終的には、よりユーザーフレンドリーなバージョンとして、Linux用にドライブを分割したり、スペースを
確保するためにmacOSのサイズを変更したり、好みのディストリビューションをインストールする方法も案内する予定です。将来的には、
グラフィカルなmacOSアプリになるかもしれませんね。

![image](https://asahilinux.org/img/blog/2021/08/asahi_bootpicker.png)
内蔵のブートピッカーを使って、2つのバージョンのmacOSとAsahi Linuxをトリプルブートすることが可能

## カーネルドライバーの追加

Svenは、M1のIOMMU(I/O Memory Management Unit)であるDARTのLinuxドライバをひたすら開発しています。
このドライバは、PCIe、USB、DCPなど、あらゆる種類のハードウェアを動作させるために必要です。このドライバはアップストリームに承認され、
Linux 5.15に搭載されることになりました。

このドライバが入ったことで、最小限の追加パッチとドライバでUSBとPCIeを動作させることができるようになりました。他にも様々な依存関係
（雑多なもののためのGPIO、適切なUSB-PDサポートのためのI²C、ラップトップでのタッチパッド/キーボードサポートのためのSPI、
NVMeサポートのパッチ）があり、それらは様々なツリーに散らばっており、人々が取り組んでいます。次に私たちは、これらのシンプルな
ドライバーを磨き上げ、開発を継続し、新しい開発者に安定した基盤を提供するために使用できる、クリーンで実用的なリファレンスツリーを
まとめることに注力します。現在の状態では、Asahi Linuxを（非アクセラレータの）GUIを備えた開発マシンとして使用することができますが、
まだ粗削りです。技術的にはシンプルなドライバをどう実装するか、お役所的な部分もありますので、アップストリームにはもう少し時間がかかりますが、
それほど時間はかからないでしょう。

これができたら...次はGPUカーネルドライバに取り組みましょう! これからが楽しみです :-)

#### marcan - 2021-08-14
