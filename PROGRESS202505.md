[Progress Report: Linux 6.15](https://asahilinux.org/2025/05/progress-report-6-15/)の非公式日本語訳です。

---
# 進捗報告: Linux 6.15

- [前回](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS202503.md)

Linux 6.15が間もなくリリースされます。つまり、最新の進捗報告をお届けする時間が来ました！
影で非常に忙しかったです。みなさまに共有できるエキサイティングな進展がありますよ。

## Fedora Asahi Remix 42のリリース
前回、Fedora Asahi Remix 42がリリースに近づいていることをお知らせしました。そのあと[リリース](https://fedoramagazine.org/fedora-asahi-remix-42-is-now-available/)され、
インストール可能になりました！ Asahi Installerは 現在、デフォルトで Fedora Asahi Remix 42 のイメージを提供しています。
FAR 40 および 41 の既存ユーザーは、`dnf system-upgrade` または Plasma の Discover を使用してアップグレードし、お気に入りのソフトウェアの
最新バージョンを楽しむことをお勧めします。また、Fedora Asahi Remix は [Fedora Linux のライフサイクルポリシー](https://docs.fedoraproject.org/en-US/releases/lifecycle/)に従っているため、
Fedora Asahi Remix 40 は完全にサポート終了（EOL）となりました。

## フォークを減らしスプーンを増やす
私たちのグラフィックスドライバーのユーザースペースAPI（uAPI）が　Linux　カーネルにマージされたことを発表できて嬉しく思います。
この大きなマイルストーンにより、Apple Silicon　での　OpenGL、OpenCL、Vulkan 対応を上流の Mesa で有効にすることができるようになりました。
これは、グラフィックスドライバーの uAPI がドライバー自体とは独立してカーネルにマージされた唯一の例であり、Rust の抽象化が上流に進む間に
上流の Mesa の有効化を促進するために、カーネルのグラフィックスサブシステム（DRM）メンテナーによって親切に許されました。
この一回限りの例外を可能にしてくれたカーネルコミュニティとの緊密な協力に感謝しています。

これにより、まもなく、Mesa、virglrenderer、および Flatpak ランタイムのフォークを廃止します。これらのフォークを排除することで、
メンテナンスの負担が軽減され、上流の Mesa で直接作業することで、ユーザースペースのグラフィックススタックに取り組む人々の開発体験が向上します。
また、[Debian](https://salsa.debian.org/xorg-team/lib/mesa/-/commit/bcd9afe05d2e31459eb8c1f54b6dda2a257cbf14)や
[Gentoo](https://github.com/gentoo/gentoo/commit/23e382acf4f7d75e49bc694f409c92385283632f)、
[Freedesktop SDK](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/commit/13e0add938f4c74887a08ad0ef6493502a8d3913)などの
他のディストリビューションが、追加のパッケージング負担なしに Apple Silicon のユーザースペースグラフィックスサポートを提供できるようになります。

Fedora Asahi Remix は、Fedora Linux 43 ベースのリリースでフォークされたパッケージを廃止します。これはユーザー介入なしに行われる予定です。
Fedora Rawhide ではこの移行が混乱を招く可能性がありますが、Rawhide はエンドユーザーシステムでの使用をサポートまたは期待されていません。

uAPI の上流化は、長い間裏で進行していた取り組みです。uAPI に1つの変更を加えるには、カーネルドライバー、Mesa、virglrenderer に相応の変更を加える
必要があります。これらの変更は、Mesa と virglrenderer が uAPI に依存してカーネルドライバーと通信するため、同期する必要があります。鋭い観察者は、
[uAPI](https://lore.kernel.org/asahi/20250326-agx-uapi-v5-1-04fccfc9e631@rosenzweig.io/)の
[多数](https://lore.kernel.org/asahi/20250310-agx-uapi-v1-1-86c80905004e@rosenzweig.io/)の
[バージョン](https://lore.kernel.org/asahi/20250313-agx-uapi-v2-1-59cc53a59ea3@rosenzweig.io/)
[が](https://lore.kernel.org/asahi/Z-Fn4niI6_Yd06Ze@blossom/) 
[カーネル メーリングリスト](https://lore.kernel.org/asahi/20250327-agx-uapi-v6-1-df6b878a61b2@rosenzweig.io/)に
[提出された](https://lore.kernel.org/asahi/20250408-agx-uapi-v7-1-ad122d4f7324@rosenzweig.io/)ことに
最終的にマージされる前に気づいたかもしれません。バージョン間の変更の一部は
本質的に根本的であり、ユーザースペースコンポーネントとカーネルドライバー自体の大幅な再作業を必要としました。
Alyssa 氏と Janne 氏は、過去数ヶ月間にわたってこの取り組みに無数の時間を費やし、変更を加え、テストし、変更を変更し、
新しい変更をテストし、繰り返しました。いつものように、この作業を完了するために多くの時間と労力を注いでくれたことに
深い感謝と敬意を表します。

## さらなるカーネルの上流化！
過去数ヶ月間にさらに多くのカーネルパッチが上流に取り込まれました。Linux 6.15 では、Apple Display Pipe（ADP）
ディスプレイコントローラーと Z2 タッチスクリーンデジタイザードライバーが導入され、M1 および M2 13インチ MacBook Pro　の
上流カーネルで　Touchbar 対応が可能になりました。

また、Apple の SoC のさまざまな機能ブロックに対応するための重要なパッチが上流に取り込まれています。
T6020 SoC（M2 Pro）のPCIeコントローラー対応がマージされ、M2 Pro Mac mini　の　USB-A　ポート、およびすべての
M2 Pro　デバイスの　WiFi　と　Bluetooth　に対応するための基盤が整いました。WiFi/BT　はさらにシステム管理コントローラーに
依存しています。SMC　ドライバーを上流化するための作業が進行中です。

Linux 6.15　には、オーディオ、特に　TAS2764　および　TAS2770　スピーカーアンプチップに関するいくつかのパッチも含まれています。
これらのパッチは、Apple Silicon Mac　に搭載されているApple固有のバリエーションの基本的な対応を追加します。

## triforce ができない
前回の更新で、ほとんどのラップトップのマイク対応をリリースしました。その後、M1 および M2 13インチ MacBook Pro 対応追加しました。
残念ながら、マイクスタックの広範なリリースにより、多くの問題が明らかになりました。M2 Pro/Max 機器の Always-On Processor（AOP）が
Apple Silicon ファミリーの他の部分とわずかに異なることがわかり、現在、これらの SoCを 搭載したラップトップではマイクが機能しません。
この問題を解決するための作業が進行中ですので、もうしばらくお待ちください！

前回説明したように、Triforce は、わずかな事前知識でビームフォーマーを実装しようとした私の素朴な試みです。合理的な時間枠で何かを
動作させるために急いで、いくつかの疑わしい _一時的な_ エンジニアリング上の決定を下しました。すぐにこれらを元に戻す時間が
あるはずだと思っていました。そうですよね？

一時的な解決策ほど永続的なものはありません。生活に追われ、問題を修正する時間がありませんでした。まあ、性能はかなり悪いですが、
十分に動作します...

Triforceを構築する際に私が下した仮定の1つは、PipeWireの「クォンタム」（バッファサイズ）が常に1024サンプルであるということでした。当時、私はこれがすべてのApple Silicon Macで真実であると思っていました。実際には、それは悪い仮定でした。トライフォースが1024サンプルより小さい入力バッファを見ると、グラフに何も返さず、マイクを効果的にミュートします。
Frédéric Bour はFedora Linux 42でこの問題に遭遇し、修正の過程で、私が開発中に下した他の多くの悪い選択も修正してくれました。最終的な結果として、トライフォースは奇妙なPipeWire構成により柔軟に対応できるようになり、約4倍高速になりました！この件で私の怠慢を拾ってくれたFrédéricに心から感謝します。





#### James Calligeros - 2025-05-15
