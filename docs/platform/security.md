---
title: Apple Silicon プラットフォームセキュリティ
---

2025/3/9時点の[security](https://github.com/AsahiLinux/docs/blob/main/docs/platform/security.md)の翻訳

訳注: 
- 文書へのリンクは対応する日本語訳へのリンクに書き換え
- セキュリティモードの訳語は[Appleの文書](https://support.apple.com/ja-jp/guide/security/sec7d92dc49f/web)の訳語に従う

---
# はじめに
Apple Siliconプラットフォームは、適切に構成されたシステムを提供するために、1から設計されました。
セキュリティモデルは『スイスチーズモデル(訳注:[スイスチーズモデル](https://ja.wikipedia.org/wiki/%E3%82%B9%E3%82%A4%E3%82%B9%E3%83%81%E3%83%BC%E3%82%BA%E3%83%A2%E3%83%87%E3%83%AB))』に基づいています。つまり、単一のセキュリティメカニズムだけでは
許容できるレベルのセキュリティを保証できないため、メカニズムを重ねて互いの穴をカバーしています。

プラットフォームのセキュリティ機能は、『Secure Enclave Processor（SEP）』によって統括されています。SEPの機能の概要やさまざまなブートポリシー、
ブートピッカー自体の概要は、[Apple Siliconの紹介](introduction.md) にあります。
このページでは、ユーザやシステム管理者が興味を持ちそうな概念を推測し、明らかにすることを試みます。
ユーザやシステムメンテナの興味を引くような概念を拡張して、明らかにすることを目的としています。

高レベルでは、完全なセキュリティモードにおけるApple Siliconのセキュリティモデルは、6つの重要な概念で構成されています。

1. システムとブートクリティカルデータの完全性を常に保証
2. セキュリティポリシーはコンテナごとに設定可能
3. ハードウェアとユーザはすべてのセキュリティ操作のためのトラストルートを形成
4. あるコンテナのセキュリティとブートポリシーは他のコンテナに影響を与えない
5. すべてのデータは静止状態で透過的に暗号化
6. ユーザーデータの復号は認証の後ろに門を設置することが可能

この文書では、完全なセキュリティモード(Full Security mode)の標準的な macOS コンテナにおけるセキュリティモデルの実装について説明し、
Appleがフルセキュリティモードで提供しているmacOSの保証を侵害することなく、ユーザーが任意のコード (サードパーティの
オペレーティングシステムなど) を実行できるようどのようにシステムが設計されているか説明します。
これは、非常に賢い方法でPCのセキュアブートの『全てか無か』のアプローチから離れたこのプラットフォーム独自のセールスポイントで
す。PCとmacOSを同様のことをするときに、なぜ注意しなければならないかを説明するのに最適な方法です。

## 目次
* [はじめに](#はじめに)
* [システムデータの完全性](#システムデータの完全性)
* [コンテナ単位のセキュリティポリシー](#コンテナ単位のセキュリティポリシー)
* [ユーザートラストルート](#ユーザートラストルート)
* [ペアリング・リカバリ](#ペアリング・リカバリ)
* [ディスクの暗号化](#ディスクの暗号化)
* [Appleでの暗黙の了解](#Appleでの暗黙の了解)
  - [私たちがどのようにこの合意を守っているか](#私たちがどのようにこの合意を守っているか)

## システムデータの完全性
このプラットフォームは、システムデータの完全性を検証し、維持するために多大な努力を払っています。すべてのシステムファイルは、
最低でもAppleによって署名され、ハッシュ化されています。sepOS(や古いバージョンではiBoot)などの重要なファームウェアコンポーネントは暗号化もされています。
システムは、これらのコンポーネントの実行を許可する前に、すべてのコンポーネントの整合性を検証します。これらのコンポーネントのいずれかが
何らかの理由で検証に失敗した場合、システムは起動に失敗し、ユーザーに機器を復元するよう指示します。

このモデルは、ファームウェアとブートローダから、macOS 自体にまで及びます。SEPは、各APFSコンテナのブートポリシーを維持し、
ブートが許可されるmacOS kernelcacheとシステムデータボリュームを登録します。
kernelcacheはAppleによって署名され、そのハッシュはSEPのBoot Policyに記録されます。
提供された kernelcache のハッシュが SEP のブートポリシー内のものと不一致であるか、
Apple によって署名されていない場合、システムはその APFS コンテナのブートを拒否します。

OSスナップショット自体は、ハッシュ化されたディスクイメージです。その中の各ファイルもハッシュ化され、これらのハッシュのマークル木
(訳注:[マークル木](https://ja.wikipedia.org/wiki/%E3%83%8F%E3%83%83%E3%82%B7%E3%83%A5%E6%9C%A8))は、
ボリューム・シールと呼ばれる最終的なハッシュを計算するために使用され、ボリュームに署名するために使用されます。もし、イメージ自体の
ハッシュやシールがSEPのブートポリシーと一致しない場合、システムは起動しません。
変更可能なユーザーデータは、コンテナ上のまったく別のボリュームに保存され、スナップショットと統合され、1つのファイルシステムとして
スナップショットが検証された後にのみ、単一のファイルシステムとしてユーザーに表示されます。どのファイルも、OSスナップショットと
ユーザーデータボリュームを交差させることはできず、フルセキュリティモードではスナップショットを変更することは一切不可能です。
この機能はSealed System Volume (SSV)として知られています。

登録されたカーネルキャッシュは、信頼されていないルートに方向転換するために使用することもできません。kernelcacheは
ブートポリシーで登録されているOSスナップショットをマウントして起動するためにのみ使用できます。
SEPは、kernelcacheがマウントすることを許可されていないOSスナップショットをマウントしようとした場合、ブートを停止します。
これにより、侵害された1つのコンテナがシステム全体を取得することを防ぎます。

## コンテナ単位のセキュリティポリシー
Appleは、このレベルのロックダウンがすべてのユーザにとってのすべてのケースに適しているわけではないことを知っています。もし誰かが
例えば、カーネル拡張を開発し、テストしたい場合はどうでしょうか？Intel Mac では、System Integrity Protectionを無効にすれば、
システムファイルの改ざんは可能です。Apple Siliconでは、Appleはより粒度の細かいアプローチを取っており、
SEP はコンテナ単位でセキュリティポリシーを追跡することができます。
これにより、ユーザーは完全に安全なmacOSのインストールを維持しながら、より制限の弱いな環境を試すことができます。

Appleは、この機能の意図はmacOSのセキュリティ保証に妥協することなくサードパーティ製OSのインストールを許可することであると
言及しており、Microsoft が Windows をこのプラットフォームに移植することを歓迎するとさえ述べています。

有効な macOS システムとして認識された APFS コンテナ (特定のファイルシステム構造が存在する必要あり) は
セキュリティ制限なしモード(Permissive Security mode)にすることができます。これはしばしば、ユーザーが任意の
コードを『実行』できるようになると宣伝されますが、ここにはいくつかのニュアンスがあります。
*実際に*許可されているのは、Apple によって署名されていない macOS カーネルキャッシュ
をコンテナのブートポリシーに登録し、SSV を無効にすることです。
ユーザーはカーネルキャッシュをSEPに提供し、SEPはそれをハッシュ化し、署名し、コンテナ用のBoot Policyを作成します。

他のシステム整合性保証は、セキュリティ制限なしコンテナにも適用されます。システムは、ファームウェアまたはブート
ファームウェアまたはブートローダが侵害された場合と同様に、ユーザー提供のカーネルキャッシュが登録後に改ざんされた場合にも、
システムはブートを拒否します。ペアリングされたrecoveryOSスナップショットは、Appleによって署名されている必要があります。
もしユーザが登録後にカーネルキャッシュを再び変更したい場合は、別のものを登録する必要があります。
これにより、プラットフォームはバイナリを信頼していることが保証されます。

しかし、悪意のあるものがあなたのセキュリティ制限なしコンテナに入り込んだらどうでしょうか？新しいkernelcacheを登録できるのではないでしょうか？
新しい kernelcache を登録し、鼻の下から黙ってコンテナを取得することはできないでしょうか？当然のことながら、Appleはこのことを考えており、
この攻撃ベクトルを軽減する非常にシンプルな方法を持っています。

## ユーザートラストルート
SEP は暗号化操作のハードウェアトラストルートですが、実はもう 1 つのコンポーネントがあり、
コンテナのセキュリティやブート設定を変更する操作のトラストルートを形成しています。ユーザーです。

ユーザーとSEPとのやり取りが、プラットフォームのセキュリティモデルに対する究極のトラストルートを形成します。
Apple Silicon 機器の初期セットアップ時に、SEP は一連の『機器所有者』資格情報を登録します。
これは、完全なセキュリティコンテナ内のマシンに設定された最初の macOS アカウントの資格情報です。
この機器所有者資格情報のセットは、
コンテナのセキュリティまたはブートポリシーの状態を変更するあらゆる操作の認証と署名に使用されます。これだけでは、
ユーザーから『真の』ハードウェアトラストルートが作成されるわけではありません。誰かが機器所有者の資格情報を
取得する可能性があるためです。
しかし、SEP は十分に賢いので、機器所有者が本人であることを確認するために、もう 1 つチェックを行います。

Apple Silicon 機器はさまざまな状態で起動することができ、SEPやiBootによって制御されます。その状態のうちの1つ
1TRは、物理的な電源ボタンを押しながら、機器所有者の資格情報で認証することによってのみ、コールドブート から移行できます。
これらの条件の両方が意図的に満たされたことをSEPが満足しない場合（例えば、電源ボタンが早期に離された場合）、1TRに移行することはできません。
セキュリティ操作コンテナをセキュリティ制限なしに降格させるなどのセキュリティ操作は、1TRから*しか*行えません。
SEP は、要求が有効な 1TR 環境から発生したことを確認できない場合、トランザクションの完了を拒否します。
カスタムブートオブジェクトのインストールなど、一部の操作では、機器所有者の資格情報を2回目に入力する必要があります。

これにより、SEPは、セキュリティポリシーに署名するために、ユーザー自身からハードウェアトラストルートを形成し、機
器所有者が要求される操作を明示的に信頼することを自分自身に保証します。
これは、あらゆるコンテナのブートプロセスを危険にさらす可能性のある、ローカルで実行されるかリモートで実行されるかにかかわらず、
ほとんどのクラスの攻撃を軽減するのに十分です。

## ペアリング・リカバリ
こちらは非常にシンプルです。macOS 12 からは、1TR で起動すると、blessされた APFS コンテナのペアードリカバリースナップショットが表示されます。
SEPではblessされたコンテナに対してのみ変更を加えることができます。
これは、悪意のあるものが物理的な存在と機器所有者の両方のチェックを迂回できる程度までコンテナが侵害された場合に、被害の拡大を抑えることができます。

## ディスクの暗号化
T2 の導入以来、Mac にはかなり強固で透過的なボリュームごとの暗号化が備わっています。このため
Apple が T2 以降の Mac の欠陥 SSD を完全なデータ損失なしに交換できません。
このシステムのデフォルトの状態は、データが静止状態で暗号化された状態です。
機器は動作中のデータを独自のキーで透過的に復号化しますが、データはディスク上では暗号化されたままです。
これによるパフォーマンスの低下はほとんどありません。Apple Siliconマシンでは、SEPは、各 APFS ボリュームごとに
ボリューム暗号化キー（VEK）と eXtended Anti-Replay Token（xART）を生成します。xARTは、replay攻撃によるVEKの取得を防止します。
VEK は SEP 自身のエアギャップメモリ(訳注: [エアギャップ](https://ja.wikipedia.org/wiki/%E3%82%A8%E3%82%A2%E3%82%AE%E3%83%A3%E3%83%83%E3%83%97) )に格納され、アプリケーションコアからは決して読み取ることができません。

ボリューム上のデータは動作中に暗号化/復号化されます。sepOSは鍵を直接NVMeコントローラに渡し、
鍵のキャプチャに対する防御を強化するために、アプリケーションコアを迂回します。重要なのは、機器の電源が入り、
SEP が iBoot とシステムファームウェアの状態に満足すると、鍵はラップされなくなり、 **すべてのデータにアクセスできる**ようになります。
ここで、macOSのFileVaultが登場します。

APFS ボリュームで FileVault が有効な場合、VEK と xART は鍵暗号化鍵 (KEK) でラップされ、
この鍵は、該当する macOS コンテナのユーザー資格情報によってバックアップされます。
保護されたコンテナのユーザーデータボリュームは、起動時にこれらの認証情報が提供されるまで、機器は読み取ることができません。
Apple Silicon 機器でこれを有効にすると、必要な操作は KEKとリカバリーキー を生成することだけなので、瞬時に実行できます。
システムスナップショット、プリブート、およびリカバリの各ボリュームは、FileVault では保護されません。
これらのパーティションは、SEPによってバックアップされた不変のものであり、ユーザーデータは含まれないので、
FileVault のメリットは特にありません。すべての暗号化キーは、機器所有者が機器を要求したときに、SEPによって破棄され、
データ復旧プログラムによっても、残存データを解読できないことが保証されます。

LVMの上にLUKSを乗せることで、これを非常にうまくエミュレートでき、効果的なフルディスク暗号化を実現できます。
APFSスタブを無視する余裕もあります。というのは、APFSスタブで気になるのはm1n1とrecoveryOSだけで、どちらも検証済み、
署名済み、改ざん防止済みだからです。既存の弱点は `/boot` が平文で保存されなければならないことです。
セキュアブートやメジャーブート(訳注:[Measured Boot](https://learn.microsoft.com/ja-jp/windows/security/information-protection/secure-the-windows-10-boot-process))の類似品で、カーネルやinitramfsの整合性を保証できるものは、今のところありません。
ユーザはブートローダをリムーバブルメディアに向けるという選択肢があり、この問題を軽減するのに役立ちます。
それ以外では、LUKS と LVM をサポートするブートローダなら、ユーザが自分の好きなようにパーティションを セットアップしたら、
標準的な `cryptsetup` ワークフローで動作するでしょう。

## Appleでの暗黙の了解
セキュリティモデルを文書化するときに、AppleはXNUカーネル開発者が自分の変更を2つ目のmacOSでテストすることを例にしています。
ですが、このプラットフォームのセキュリティモデルは、
AppleがmacOS自体に保証しているセキュリティを一切損なわない形で、サードパーティ製OSがmacOSと共存できるように設計されていることは明らかです。
Appleのまわりでこのような取り組みに積極的に敵対しているとか、信頼できないコードを実行するためにはセキュリティを回避したり、
脱獄(訳注:[jailbreak](https://ja.wikipedia.org/wiki/Jailbreak))しなければならないといった噂が流れていますが、根拠がなく誤りです。
実際、macOS以外のバイナリの実行を向上させる*だけ*の方法で、Appleは彼らのセキュリティツールを*向上*させるために労力と時間を費やしてきました。
その一例としてmacOS 12.1以降Boot Policy設定ツールに、生のAArch64コードを適切なMach-Oフォーマットでラップする機能を提供しています。
これはmacOSのカーネルキャッシュでないブートオブジェクトを登録するためにのみ必要です。

とはいえ約束事は必然的に2つに分かれます。Apple Silicon Macが本質的にiDeviceをスケールアップしたものであることを考えると、
Appleはわざわざそのセキュリティモデルをより柔軟なものに変更したのであり、その理由を推測するのは難しいことではありません。
Asahiのようなプロジェクトが不可避であることを、会社のどこかのレベルでは、同意によるものであろうとなかろうと、
誰かが十分に認識していたはずです。
脱獄コミュニティが任意のコードを実行させようとすることで、Appleのプライバシーとセキュリティのマーケティング保証に
恥をかかせるリスクを冒すよりも、Appleは私たちにサンドボックスに触れる必要なく簡単にそれを行うためのツールを与えてくれました。
よって、私たちはこの機能が1つの簡単な条件付きで提供されていると考えています。
どんな状況でも、macOSのコンテナを汚染しようとしないことです。

### 私たちはどのようにこの契約を守っているか
Asahi Linuxは、有効なOSとして認識されるような正しいファイル構造を持つ小さなAPFSコンテナとボリュームセットを作成します。
そして、Appleのツールを使ってセキュリティを制限なしに設定し、m1n1をその署名されたブートオブジェクトとして登録します。
私たちは*他のどの*OSボリュームのセキュリティ設定も変更することはありませんし、今後も変更することはありません。
AppleのセキュリティポリシーがAsahiボリュームに影響を与えることはありません。
詳しくは [Apple Silicon MacでのオープンOSエコシステム](open-os-interop.md)をご覧ください。
