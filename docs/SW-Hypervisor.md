2025/3/1時点の[SW-Hypervisor](https://github.com/AsahiLinux/docs/blob/main/docs/SW-Hypervisor.md)の翻訳

訳注:Asahi Linux内のページへのリンクは対応する日本語訳に置き換え

---
# m1n1ハイパーバイザーでのmacOSの実行

Apple から入手した開発用カーネルを実行することができます。この場合デバッグシンボルを得ることができますし、macOS インストール内の
純正カーネルを使用することもできます。

## 準備
既存の macOSインストール を使用してもよいですし、 代わりにmacOS のセカンドコピーをインストールすることもできます。

macOS のセカンドコピーをインストールするには、いくつかのステップを完了する必要があります:

1. macOS パーティションにセカンドボリュームを作成します。

        diskutil apfs addVolume disk4 APFS macOSTest -mountpoint /Volumes/macOSTest

disk4 とボリューム名 (つまり macOSTest) を特定のシステム/環境設定用に変更します。 
_注: このロールをシステムロールにしてはいけません。既存のシステム が混乱します (1TR内に有効なユーザーが存在しない)_

2. macOSをダウンロードしインストールします。macOSの特定のバージョンのインストーラをダウンロードするには次のコマンドを使用します。

        softwareupdate --fetch-full-installer --full-installer-version 12.3

12.3は必要なバージョンに置き換えます。インストーラはアプリケーションフォルダ内にあり、保存したい場合はここからコピーしてください。
そうでない場合は、一度インストールすると自動的に削除されます。

残念ながら、AppleのCDNは限られたバージョン用のフルインストーラパッケージしか保持しておらず、12.3はもうありません。
_注：現在、ファームウェアのバージョンは13.5となっており、通常通り利用できます。12.3をインストールする必要はありません_

### アーカイブされたInstallAssistant.pkgの使用

Montery 12.3 の InstallAssistant.pkg は [こちら](https://archive.org/details/12.3-21-e-230-release)にアーカイブされていますが、
ダブルクリックでインストールしようとすると、ファイルサイズ約 40MB のオンライン版の `Install macOS Monterey.app` がインストールされるようです。
それを実行すると、最新版のmacOSがインストールされます。

しかし、コマンドラインからインストールすると、次のように正しくインストールされるようです：

        sudo installer -pkg 12.3\ 21E230\ \(Release\).pkg -target /

`アプリケーション`フォルダ内の `Install macOS Monterey.app` が 12GB以下 であることを確認してください。

## macOS の開発用カーネルの取得と kernelcache の作成

1. macOS開発者アカウントを作成（icloudアカウントが必要)
2. Appleの[ここ](https://developer.apple.com/download/more/)からMac OS Kernel Debug Kit (KDK)をダウンロード。使用中のMac OSのバージョンに合わせてダウンロード
3. KDKをMac OSにインストール。KDKは `/Library/Developer/KDKs/KDK_<MACOS_VERSION>_<KDK_VERSION>.kdk` にインストールされる
4. kernelsディレクトリに移動：

        cd /Library/Developer/KDKs/KDK_<MACOS_VERSION>_<KDK_VERSION>.kdk/System/Library/Kernels
   
5. KDKフォルダに切り替えて、以下のコマンドを実行:

        kmutil create -z -n boot -a arm64e -B ~/dev.kc.macho -V development \
          -k kernel.development.t8101 \
          -r /System/Library/Extensions/ \
          -r /System/Library/DriverExtensions \
          -x $(kmutil inspect -V release --no-header | grep -v "SEPHiber" | awk '{print " -b "$1; }')

    `-B` designates the output file, our kernel cache is written to `dev.kc.macho` in the home directory

    `-k` must match a kernel in the kernels directory

## セキュリティ機能を無効にしてmacOSボリュームを準備
0. macOSボリュームをデフォルトのブート対象に設定
1. 1trに入り、ターミナルを起動
2. boot policyでほとんどのセキュリティ機能を無効化。`bputil -nkcas`。 UUIDを取得するために `diskutil info [disk name]` を使用
3. SIP を無効化 (bputilがリセットする): `csrutil disable`
4. カスタムブートオブジェクトとして[m1n1](m1n1-User-Guide.md)をインストール

        kmutil configure-boot \
          -c build/m1n1.bin \
          --raw \
          --entry-point 2048 \
          --lowest-virtual-address 0 \
          -v /Volumes/macOSTest

## m1n1 ハイパーバイザー下での開発用カーネルの起動

1. kernelcache を開発マシンにコピー
2. デバッグ用 DWARF を `/Library/Developer/KDKs/KDK_<MACOS_VERSION>_<KDK_VERSION>.kdk/System/Library/Kernels/kernel.development.t8101.dSYM/Contents/Resources/DWARF/kernel.development.t8101` からコピー
3. 実行 

        python3 proxyclient/tools/run_guest.py \
          -s <PATH_TO_DEBUG_DWARF> \
          <PATH_TO_DEVELOPMENT_KERNEL_CACHE> \
          -- "debug=0x14e serial=3 apcie=0xfffffffe -enable-kprintf-spam wdt=-1 clpc=0"

m1n1 ハイパーバイザー下でmacOS が起動

注: KDK for macOS 11.3からはt8101ファイル（カーネルシンボルとドワーフシンボルの両方）が利用できます。ハイパーバイザーでの macOS の起動に関する Marcan 氏のストリームは
11.3で行われました。
これらのノートは、macOS 11.5.2でも検証されており、MacBookAirのwifiネットワークでログインできるようになっています:

- Kernel version:  

```
$ uname -a 
> Darwin MacBook-Air.home 20.6.0 Darwin Kernel Version 20.6.0: Wed Jun 23 00:26:27 PDT 2021; root:xnu-7195.141.2~5/RELEASE_ARM64_T8101 arm64
```
- macOS version: 

```
$ sw_vers
> ProductName:	macOS / ProductVersion:	11.5.2 / BuildVersion:	20G95
```

アップルロゴ（レインボーバージョン）が表示されるがプログレスバーが表示されない場合は、ブートプロセスの初期段階でmacOSがパニックを起こしている可能性があります。
これは、kernel cache（kernel+extensions）とmacOSのroot fsの間でmacOSのバージョンが一致していないことが原因です。
ブートプロセスがどこで止まっているかを調べるには、minicom/picocom などのシリアルユーティリティーを 115200 ボーレートで起動します（`picocom -b 115200 /dev/ttyACM1` のように）。
起動時には、1つのCPUとハイパーバイザで我慢してください。トレースしている内容によっては通常よりも遅くなりますが予想通りです。

以下は、macOS `11.5.2` と m1n1 バージョンのコミット `bd5211909e36944cb376d66c909544ad23c203bc` を使って実験した結果です:
- run_guestコマンド launch(t0)からカーネルのロード開始まで: 9秒
- t0からログイン画面まで（キーボードやマウスカーソルが最初から動いていない状態）：約2分
- キーボードとマウスのカーソルが動いた状態: 約2分35秒
- パスワードを入力してからデスクトップとメニューバーが表示されるまで：約2分

## インストールしたmacOSからmacOS純正カーネルを実行

1. macOSを起動
2. `kernelcache` を`/System/Volumes/Preboot/(UUID)/boot/(long hash)/System/Library/Caches/com.apple.kernelcaches/kernelcache` から探す
3. このファイルのコピーをどこかに作成
4. img4tool (https://github.com/tihmstar/img4tool) のコピーを入手 (またはビルド)
5. im4p イメージを抽出:

        img4tool -e -p out.im4p kernelcache

6. im4p から machO を抽出:

        img4tool -e -o kernel.macho out.im4p

7. これで上記と同様の方法でmacOSを実行可能(デバッグDWARFはなし)

        python3 proxyclient/tools/run_guest.py \
          <PATH_TO_EXTRACTED_MACHO> \
          -- "debug=0x14e serial=3 apcie=0xfffffffe -enable-kprintf-spam wdt=-1 clpc=0"

## m1n1 ハイパーバイザツリーの更新
ハイパーバイザー/m1n1 ABI は * 安定していません* 。上記のように新しい m1n1 ビルドをインストールした場合、時間を節約するために 
`run_guest.py` を直接使用できます。ただし、m1n1 の git ツリーを更新したらすぐに、 `run_guest.py` の前に更新した m1n1 をビルドし

```
python tools/chainload.py -r ../build/m1n1.bin
```

を実行して、ABI が同期していることを確認しなければなりません。
これを行わないと、ABIの不整合によるランダムなエラーやクラッシュが発生します。

## GDB/LLDBの使用
`gdbserver` コマンドは GDB または LLDB に接続できるサーバーの実装を起動します。LLDBがより推奨されます。というのは
ポインタ認証とDarwinカーネルdyldに対応しているからです。
LLDBでシンボルを取得するには、カーネル拡張をロードする必要があります。以下のシェルスクリプトは `target.lldb` を生成します。
これはターゲットを設定し、カーネル拡張をロードする便利なLLDBスクリプトです:

```sh
echo target create -s kernel.development.t8101.dSYM kernel.development.t8101 > target.lldb
for k in $(find Extensions); do [ "$(file -b --mime-type $k)" != application/x-mach-binary ] || printf 'image add %q\n' $k; done >> target.lldb
```

以下のLLDB用コマンドは、生成されたスクリプトをロードし、m1n1に接続します:

```
command source -e false target.lldb
process connect unix-connect:///tmp/.m1n1-unix
```

GDB/LLDBと干渉するハイパーバイザーのコンソールコマンドを実行しないでください。例えば、
ハイパーバイザーコンソールと GDB/LLDB の両方から同時にブレークポイントを編集しないでください。

# 情報源
kernelcache作成の情報源: 
[https://kernelshaman.blogspot.com/2021/02/building-xnu-for-macos-112-intel-apple.html](https://kernelshaman.blogspot.com/2021/02/building-xnu-for-macos-112-intel-apple.html)
