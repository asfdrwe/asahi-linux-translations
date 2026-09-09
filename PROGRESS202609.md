[M2: Episode 1 (or, Asahi Linux on M3)](https://asahilinux.org/2026/09/m2-episode-1/)の非公式日本語訳です。

---
# M2: エピソード 1 (もしくは M3 での Asahi Linux)

- [前回](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS202608.md)

進捗報告以外のブログ投稿を書くのは久しぶりですが、今日はちょうど良い機会です。M3 シリーズのマシンへの対応がインストーラに統合されました。つまり、**Asahi Linux は M3 シリーズの SoC を搭載した Mac に公式対応するようになりました！**

M3 シリーズの SoC およびそれらを搭載したマシンへの Linux 対応は、現在 M1 および M2 シリーズ機器で対応している機能のほとんどがそのまま動作する状態になっています。これには Web カメラ、内蔵マイク、USB（ハードウェアの上限である USB 3 10 Gb/s まで）、ハードウェアアクセラレーションによるビデオデコード（**AV1 のサポートを含む**）、WiFi、Bluetooth、その他多数の機能が含まれます！主な例外は完全な DCP 対応と GPU のみで、これらについては[今後数ヶ月](https://indico.freedesktop.org/event/12/contributions/532/)でさらにニュースをお伝えします。**現時点ではパフォーマンスや電力効率の良い 3D アクセラレーションを期待しないでください。**

この作業の多くがまだ新しいものであるため、インストーラの Expert モードの後ろ側に配置しています。M3 シリーズ機器で Asahi Linux を試してみたいユーザーは、macOS のターミナルで次のコマンドを実行し、プロンプトに従ってください。

```sh
curl -L https://alx.sh/ | EXPERT=1 sh
```

インストールが完了したら、システムをアップグレード（`dnf upgrade --refresh`）することを忘れないでください。大きな後退やその他の致命的な問題がなければ、数週間後の Fedora Linux 45 ベータリリースに合わせて Expert 要件を外す予定です。

ただし、以下の既知の制限事項に注意してください：

- スリープは現在、ファームウェア提供のフレームバッファの制限により**動作しない**。これは M3 向けに完全な DCP 対応が実装され次第対応
- 同様に、DCP 対応がないため、装備されている MacBook の HDMI ポートは現在無効化
- MacBook と iMac（M3、M3 Pro、M3 Max）対応しているが、Mac Studio（M3 Ultra）はまだ未対応

いつものように、[OpenCollective](https://opencollective.com/AsahiLinux) と [GitHub Sponsors](https://github.com/sponsors/AsahiLinux) の両方でご支援いただいている寛大なサポーターの皆さんに感謝します。M3 への対応は、複数の人々による長年のハードワークの集大成であり、その多くは皆さんのサポートのおかげでようやく M3 ハードウェアにアクセスできるようになった人々です。

みなさまのフィードバックを楽しみにしており、私たちがお届けするのを楽しんだのと同じくらい、みなさまが M3 機器で Asahi Linux を楽しんでくれることを願っています。さらなるアップデートをお待ちください。Happy computing！

#### James Calligeros · 2026-09-06
