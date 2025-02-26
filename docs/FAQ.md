2025/2/20時点の[FAQ](https://github.com/AsahiLinux/docs/blob/main/docs/FAQ.md)の翻訳

訳注: Asahi Linux内のページへのリンクは対応する日本語訳に置き換え、Arch Linux wikiへのリンクは対応する日本語Arch Linux wikiへのリンクへ置き換え

---

## Asahi Linuxはいつ『完成』しますか？

--> [『Asahi-Linuxはいつ完成するの?』](When-will-Asahi-Linux-be-done.md)

## $これはまだ動かないのですか？

--> [機能対応](Feature-Support.md)

## どのようにインストールしますか？

手順はウェブサイトをご覧ください: https://asahilinux.org/

訳注: 
macOSから
```
curl https://alx.sh | sh
```
としてください。その他文書の日本語訳は https://github.com/asfdrwe/asahi-linux-translations を参照してください。

## どのようにアンインストールするかと失敗したインストールをクリーンアップをどのようにしますか？

自動アンインストーラはありませんが、手動でパーティションを削除する方法については [パーティション分割のチートシート](Partitioning-cheatsheet.md) を参照してください。

## Asahi Linux をデュアルブートできますか？

はい！インストーラーはすでにデュアルブートするよう設定します。macOS を自身で消去することもできますが、*お勧めしません*。

## Asahi Linuxを純粋にUSBドライブから起動できますか？

いいえ、残念ながらできません。Apple Silicon のハードウェアは USB ストレージからの起動に全く対応していません。内部ストレージに変更を加えずに
USB からブートすることは、物理的に完全に不可能です。これはセキュリティ上の理由から設計されています。

## ディスクの空き容量が 40GB以上 あるのですが、インストーラが足りないと言っています！

インストーラは、macOS のアップグレードがうまくいくように、 *常に 38GB のディスクスペースを* 空けています。つまり、この 38GB に加えて、
新しい OS のための十分なディスクスペースが必要なのです。

このチェックをスキップしたい場合は、最初にエキスパートモードを有効にしてください。十分な空きディスク容量が残っていない場合、
macOSのアップデートができない可能性があることに留意してください！


## iPad に Asahi Linux をインストールできますか？

いいえ、iPad（およびiPhoneやその他のデバイス）はカスタムOSカーネルの実行に対応していません。システムの設計上、たとえユーザ空間で任意のコードを
実行できたとしても（例えば脱獄によって）、Linuxカーネルの実行には役立たないでしょう。ブートROM脆弱性が存在すれば、役に立つかもしれませんが、
これらの機器に対応することは、Asahi Linux のプロジェクトの目標ではありません。

## よくある問題

### インストーラの macOS リサイズステップ中にエラーが発生します

エラーメッセージをよく読んでください。以下のいずれかになります。

#### "Storage system verify or repair failed" 
『ストレージシステムの検証または修復に失敗しました』

APFS ファイルシステムが破損しており (インストーラが原因ではありません)、macOS パーティションのサイズ変更が正常に行えません。
詳細と修正手順については、[このissue](https://github.com/AsahiLinux/asahi-installer/issues/81) を参照してください。

#### "Your APFS Container resize request is below the APFS-system-imposed minimal container size (perhaps caused by APFS Snapshot usage by Time Machine)"
『APFS コンテナのサイズ変更要求はAPFS-system-imposed最小コンテナサイズを下回っています (おそらく Time Machine による APFS Snapshot 使用が原因) 』 というメッセージが表示されます。

このメッセージが示すように、これはTime Machineのスナップショットがディスクの『空き』スペースを占有することが原因です。詳細と修正手順については、[このissue](https://github.com/AsahiLinux/asahi-installer/issues/86) を参照してください。

### Disk Utilityがインストール後/アンインストール時/その他の時に動作しません

Disk Utilityは使わないでください。どうしようもなくて、本当にシンプルなパーティションセットアップにしか使えません。代わりに
コマンドラインでパーティションを管理する方法は [パーティション分割のチートシート](Partitioning-cheatsheet.md)  をご覧ください。

## 新機能やアップデートを取得するために再インストールする必要がありますか？

いいえ！`dnf upgrade` を使ってシステムをアップグレードするだけです。カーネルアップデートは再起動が必要です。
`needrestart` のようなツールを使って、古いサービスや古いカーネルが動いていないかどうか調べてください。 

## キーボードのキーの2つが入れ替わっています
これは一部のアップルキーボードに固有の性質です。この問題を解決するには、https://wiki.archlinux.jp/index.php/Apple_Keyboard をお読みください。

## AsahiではなくmacOSにブートする状態のままです!
OSXがデフォルトのブートメディアとして設定されている可能性が場合があります。シャットダウンし、電源ボタンを15秒間押したまま電源を入れ、
One True Recovery（1TR）に入ります。Optionキーを押しつづけてAsahi Linuxを選択するか、設定ページに入ってOSXをアンロックし、
ブートローダーを設定してAsahiに再起動させてください。

## Xorgで性能・テアリング(訳注:映像の乱れ)・機能の問題が発生しています

Xorgの使用を中止し、Waylandに切り替えてください。Xorgは主要なディスプレイサーバーとしてメンテナンスされておらず、そのアーキテクチャは
Apple Silicon機器に存在するような最新のディスプレイハードウェアとは相容れないものです。Xorgとその独自性に時間を費やすための開発枠がありません。
ディストリビューションや下流のデスクトップ環境ではすでにXorgへの対応が終了しています。望む限りXorgを使い続けるのは自由ですが、『起動して
基本的なデスクトップを正しく表示する』以上の対応をするつもりはありません。

[Fedora Asahi Remix](https://github.com/asfdrwe/asahi-linux-translations/blob/main/fedora.md) (フラグシップリファレンスディストリビューション)はこのような理由により
箱から出したらすぐWayland専用になっています。

## スクリーンリーダーソフトウェアやその他のアクセシビリティ機能がデフォルトで動作しない

2025年2月現在、アクセシビリティ機能はデフォルトで有効になっており、インストール後すぐにロック画面で利用できるようになるはずです。
これは以下のように実装されています [https://pagure.io/fedora-asahi/calamares-firstboot-config/pull-request/5](https://pagure.io/fedora-asahi/calamares-firstboot-config/pull-request/5)。

## 画面録画が遅いです

GPUドライバがインストールされていることを確認してください。それでも画面録画が遅い場合は、CPUからGPUディスプレイサーフェスを直接読み出したり
コピーしたりする画面録画アプリやコンポジターを使用していると思われます。GPUディスプレイサーフェスはGPUアクセスに最適化されており、圧縮された
プライマリディスプレイフレームバッファに切り替わると、CPUの直接読み出しは全く機能しなくなる可能性があります。つまり、
[kmsgrab](http://underpop.online.fr/f/ffmpeg/help/kmsgrab.htm.gz) のようなアプローチは根本的な欠陥があり、性能が悪く、
将来完全に機能しなくなるでしょう。GPUテクスチャを正しく共有し、CPUエンコード用に読み出しを最適化するディスプレイコンポジターと録画アプリを
使用する必要があります。KDEのKWinとOBSは、KDEのスタンドアローンのSpectacleスクリーンショット・録画アプリと同様に、うまく動作することが知られています。

## Chromium / VS Code / Slack / Discord / その他のElectronアプリまたはChromeベースのブラウザが、アップデート後にレンダリングできなくなりました
これは、[Electron](https://github.com/electron/electron/issues/40366)などのすべての Chromium ベースの
フレームワークに影響する [Chromium 上流のバグ](https://bugs.chromium.org/p/chromium/issues/detail?id=1442633) です。
シェーダーキャッシュを手動で削除する必要があります（例えば：`~/.config/Slack/GPUCache`）。修正がバックポートされ、
ユーザーにリリースされるまで、こちら側では何もできません。
