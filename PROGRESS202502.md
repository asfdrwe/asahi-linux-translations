[Passing the torch on Asahi Linux](https://asahilinux.org/2025/02/passing-the-torch/)の非公式日本語訳です。

訳注: 関連する文書へのリンクは対応する日本語訳へのリンクに変更

---
# Asahi Linux のバトンタッチ

- [前回](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS202412.md)

謹んでAsahi Linuxの創設者であるHector Martin(marcan)氏の辞任を発表します。
marcan氏の声明は[marcan氏ブログ](https://github.com/asfdrwe/asahi-linux-translations/blob/main/marcan.md)に掲載されています。
Asahi Linux は Apple Silicon にLinux を提供し、オーディオ、ウェブカメラ、グラフィックアクセラレーションなどに対応してきました。
残された開発者として、私たちはこれを持続可能なプロジェクトガバナンスを構築する機会と捉えています。どんなに優秀な個人であっても、
大きなプロジェクトは一人の肩にかかっては成り立ちません。ですから、1人の後任ではなく…7人の後任がいます。

- [Alyssa Rosenzweig氏](https://rosenzweig.io/) グラフィック開発
- [chaos_princess氏](https://social.treehouse.systems/@chaos_princess) カーネル開発
- [Davide Cavalca氏](https://github.com/davide125) Fedora開発
- [Neal Gompa氏](https://royalgeekworld.com/) Fedora開発
- [James Calligeros氏](https://social.treehouse.systems/@chadmed) オーディオ開発
- [Janne Grunau氏](https://social.treehouse.systems/@janne) カーネル開発
- [Sven Peter氏](https://social.treehouse.systems/@sven) カーネル開発

プロジェクトの意思決定に関しては、私たちの[新しいガバナンス](https://github.com/asfdrwe/asahi-linux-translations/blob/main/governance.md)に従って、
平等な権限を共有します。誰の貢献も永遠には続きません。このガバナンスの変更により、開発者の出入りがあってもプロジェクトを存続させることができるようになります。

Asahi Linux は主にボランティアの貢献者に依存しています。個人で Patreon や GitHub Sponsors のアカウントを持っている貢献者もいますが、個人の資金源では
チームを維持することはできません。今後、新しい財政スポンサーである Open Source Collective は代わりにプロジェクト全体への寄付を推進してまいります。

したがって、私たちの[Open Collective](https://opencollective.com/asahilinux)は、プロジェクトの主要な資金源として marcan氏の Patreon に
取って代わります。Patreon はまもなく終了します。4年前、みなさまの Patreon の支援によってこのプロジェクトは実現しました。marcan 氏の
Patreonが終了してしまうので、本日より今後何年にもわたってプロジェクトを可能にするためにみなさまの支援をお願いします。みなさまからのご支援により
ハードウェアの購入や開発者の時間を確保することができます。プロジェクト継続のため、Open Collective への参加をご検討ください。

2025年には何が期待できますか？

優先事項はカーネル上流です。私たちの下流のLinuxツリーには上流のLinuxにはまだ存在しない Apple Silicon に必要なパッチが1000以上含まれています。
上流カーネルの動きは速く、統合競合(merge conflict)や後退(regression)と戦いながら、私たちの変更を常に上流上にリベースする必要があります。
Janne 氏や Neal 氏や marcan 氏は何年も私たちのツリーをリベースしてきましたが、これだけ多くのパッチがあると手間がかかります。さらにパッチを追加する前に、
長期的に持続可能であるようパッチのスタックを減らす必要があります。このプロセスがどうなるかは予測できませんが、自分の役割を果たすことに全力を尽くします。

もうひとつの持続可能性の問題はテストです。対応するすべての機能が対応するすべてのハードウェア上で動作することを時間の経過とともに後退することなく
保証しなければなりません。より多くの機能とハードウェアに対応するにつれて、テスト要件は爆発的に増加します。残念なことに、手作業でのテストには時間がかかり、
バグが抜け落ちることもあります。その解決策が、多くの機器上で Asahi Linuxを自動的にテストする継続的インテグレーション（CI）です。上流と同様、
このインフラ構築は派手ではありませんが、プロジェクトの長期的な健全性を守ることになります。

M3とM4の位置づけはどうなりますか？上流とCIが進展するまで、コアチームは新しいハードウェアに優先順位をつけることはできません。とはいえ、コミュニティメンバーの中には、
基礎が固まったときに備えてリバースエンジニアリングに精を出している者もいます。

もちろん、M1やM2機器向けの新機能も登場します。2025年にリリース予定のものは…

- DP altモード 物理的なHDMIポートのないラップトップでUSB-C経由の外部モニターに必要です
- DirectX 12を可能にするVulkanドライバーのスパースイメージ しばらくの間はAsahi LinuxでDirectX 11のゲームを楽しむことができます
- 内蔵マイク 外部マイクは3.5mmジャック経由ですでに動作しており、内蔵マイクも近日中に登場する予定です

いつ頃でしょうか？一部のノートパソコンに関しては、ほんの数日です！マイク対応は、James氏とchaos_princess氏とEileen Yoon氏の協力により実現しました。
Apple Silicon では、マイクは Always-On Processor (AOP) や Secure Enclave (SEP) を含む複数のハードウェア・ブロックのカーネル対応と、
オーディオサウンドを確実にするためのビームフォーミングのユーザー空間対応を必要とします。samples-in と samples-out だけなのではないのです。
しかし、この3人はチャレンジ精神旺盛でした。

今日のニュースはほろ苦いです。私たちは、このプロジェクトを立ち上げ、この数年間精力的に取り組んでくれたmarcan氏に感謝しています。私たちのコミュニティは
彼を失うことになります。それでも、みなさまのサポートがあれば、このプロジェクトには明るい未来が待っているでしょう。

#### Asahi Linuxチーム - 2025-02-13
