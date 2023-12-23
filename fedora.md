[fedora](https://asahilinux.org/fedora/)の2023年12月23日時点の非公式日本語訳です。

訳注: プロジェクトページへのリンクは対応する日本語訳へのリンクに置き換え。体裁は一致できていません。

---
# Fedora Asahi Remixの紹介
Apple Silicon Macのための最も洗練されたLinux®。

![far_laptop.svg](https://asahilinux.org/img/far_landing/far_laptop.svg)

## macOSからのインストール
```
curl https://alx.sh | sh
```

## Fedora Linux 39 + Apple Silicon = Fedora Asahi Remix
![fedora_remix.png](https://asahilinux.org/img/far_landing/fedora_remix.png)
Fedora Asahi Remixは、Asahi LinuxプロジェクトとFedoraプロジェクトとの複数年にわたる緊密な協力の成果です。
改善やバグフィックスをできるだけ早くユーザーに提供するために緊密に協力し、完全に統合されたディストリビューションを
お届けするために努力してきました。Asahiプラットフォーム固有のパッケージはすべて上流Fedoraに存在し、
Fedora Linux 39で完全に対応しています。

Fedoraの優れた64ビットARMサポートと成熟した開発プロセスにより、望まぬ予想外のことがない、堅実で高品質な
エクスペリエンスが期待できます。Fedora Asahi Remixは、最新のFedora LinuxリリースであるFedora Linux 39を
ベースにしています。すべてのM1、M2シリーズのMacBook、Mac Mini、Mac Studio、iMacに対応しています。

## Fedora Asahi Remix ❤️ KDE Plasma
![kde-logo-white-blue-rounded-source.svg](https://asahilinux.org/img/far_landing/kde-logo-white-blue-rounded-source.svg)
KDE Plasmaをフラッグシップデスクトップ環境として提供できることを誇りに思っています。最先端のWaylandサポートと高度なカスタマイズ性、
Appleハードウェア機能への幅広い対応により、KDE PlasmaはApple Siliconで使う喜びを感じさせてくれます。

Night Colorを使って、睡眠サイクルを妨げないようにしませんか？心配ありません、動作します。
トラックパッドの設定を微調整して、より快適に使いたいですか？すべてシステム設定にあります。
画面上のものが大きすぎたり小さすぎたりしていませんか？ディスプレイのスケールを5%刻みで調整できます。
KDEプロジェクトと協力して、プラットフォームサポートを改善するためのバグフィックスと改良を提供し、
Calamaresベースのカスタム初期セットアップウィザードも構築しました。

Fedora Linux 39には、最新のパッチと改良が加えられたKDE Plasma 5.27が付属しています。しかし、それだけではありません：
さらに改良されたKDE Plasma 6が登場するFedora Linux 40にご期待ください。

[GNOME](https://www.gnome.org/)を使いたいですか？ご心配なく、GNOME 45 でカバーできます。また、独自のデスクトップ構成を
ロールしたい場合やヘッドレス・サーバーをセットアップしたい場合、ServerとMinimalイメージを使えば、思い通りにセットアップできます。

## 100% Wayland 体験
![Wayland_Logo.svg](https://asahilinux.org/img/far_landing/Wayland_Logo.svg)
KDEオタクであってもGNOME愛好家であっても、Fedora Asahi Remixは100%Wayland環境を提供します。Appleハードウェアに完璧に
マッチする最新のデスクトップとディスプレイサーバーのテクノロジーをもたらします。macOSのように、ティアリングやグリッチの全くない、
バターのように滑らかなデスクトップが手に入ります。KDE Plasmaビルドでは、ディスプレイのスケールが異なる複数のディスプレイでも、
シームレスなHiDPI対応を体験できます。

また、Waylandエコシステムの今後の改善により、HDRやディスプレイノッチなどの新技術や、適切なディスプレイキャリブレーションに
対応できるようになります。実行するX11アプリがありますか？ご心配なく。XWaylandはレガシー・アプリケーションのブリッジとして
利用可能で、完全に対応しています。

## OpenGLは非推奨？違います
![OpenGL_ES_logo.svg](https://asahilinux.org/img/far_landing/OpenGL_ES_logo.svg)
Fedora Asahi Remixは、GPUアクセラレーテッドジオメトリーシェーダーとトランスフォームフィードバックを含む非準拠のOpenGL 3.3対応と、
Apple Silicon用の世界初で唯一の[認定準拠OpenGL ES 3.1実装](https://www.khronos.org/conformance/adopters/conformant-products/opengles#submission_1007)を
持っています。

オープンなグラフィックス標準に対応し、公式および業界標準のテスト・スイートでテストしています。つまり、アプリケーションや
ゲームが正しく動作し、意図したとおりにレンダリングされることを確信できるのです。さらに、OpenGL 4.xとVulkan対応も準備中です。
Metalのようなベンダー独自のAPIの上に重ねることで可能になることをはるかに超えて、Apple Siliconグラフィックスの可能性を最大限に引き出すことを目指しています。

## 視聴したことのある最高のLinuxラップトップオーディオ
![curves.svg](https://asahilinux.org/img/far_landing/curves.svg)
過去2年間、私たちはデスクトップLinuxエコシステムのための世界初の完全に統合されたDSPソリューションを開拓するために努力してきました。Fedora Asahi Remixを
インストールするだけで、セットアップ不要ですぐに高品質なオーディオを楽しむことができます。[PipeWire](https://pipewire.org/)および
[WirePlumber](https://gitlab.freedesktop.org/pipewire/wireplumber)プロジェクトと協力し、
完全自動かつ透過的なDSPコンフィギュレーションのサポートを追加し、8つ以上の異なる機種を個別に測定・校正し、それぞれにカスタマイズされたDSPフィルター構成を設計しました。

特製の[Bankstown](https://github.com/chadmed/bankstown)低音ブースト・テクノロジーと、ラウドネスとダイナミック・レンジをフルに安全に提供するための
オープンソースのSmart Amp実装により、Linuxラップトップで聴いたことのない最高のオーディオを実現しました。さらに、DSP処理のスケジューリングと消費電力も
最適化されているため、オーディオ再生中のバッテリー寿命も抜群です。

## 機器対応
| 機種       | チップ     | 対応機能      |対応作業中    |   備考          |
|-----------|:---------:|:------------:|:------------:|:------------:|
|MacBook Air|M1, M2     | ディスプレイ、キーボード (+バックライト)、トラックパッド、ヘッドセットジャック、スピーカー、カメラ、MagSafe*、USB Type C(USB 3.0)、Wi-Fi、Bluetooth|USB-C ディスプレイ、Thunderbolt / USB4、マイク、Touch ID| *M2モデルのみ)
|MacBook Pro|M1, M1 Pro, M1 Max, M2, M2 Pro, M2 Max| ディスプレイ*、キーボード (+バックライト)、トラックパッド、タッチバー†、ヘッドセットジャック、スピーカー、カメラ、MagSafe‡、USB Type C(USB 3.0)、HDMI‡、SD Card‡、Wi-Fi、Bluetooth|USB-C ディスプレイ、Thunderbolt / USB4、マイク、Touch ID|*14インチおよび16インチモデルではローカルディミングが可能、全モデル最大60HzリフレッシュレートでHDR/120Hzは未対応 †13インチモデルのみ ‡14インチおよび16インチモデルのみ
|Mac Mini   |M1, M2,M2 Pro|ヘッドセットジャック、スピーカー、USB Type A (3.0),USB Type C(USB 3.0)、HDMI、イーサネット(1/10 Gbps)、Wi-Fi、Bluetooth|USB-C ディスプレイ、Thunderbolt / USB4| |
|Mac Studio |M1 Max, M1 Ultra, M2 Max, M2 Ultra|ヘッドセットジャック、スピーカー、USB Type A (3.0),USB Type C(USB 3.0)、HDMI、SD Card、イーサネット(1/10 Gbps)、Wi-Fi、Bluetooth|USB-C ディスプレイ、Thunderbolt / USB4| |
|iMac       | M1        |ディスプレイ、ヘッドセットジャック、カメラ、USB Type C(USB 3.0)、イーサネット(1/10 Gbps)、Wi-Fi、Bluetooth|スピーカー、USB-C ディスプレイ、Thunderbolt / USB4| |
|Mac Pro    | 作業中     |
