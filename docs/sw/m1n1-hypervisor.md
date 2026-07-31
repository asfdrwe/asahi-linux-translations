---
title: m1n1 ハイパーバイザー
---

2026/7/31の[m1n1-hypervisor](https://github.com/AsahiLinux/docs/blob/main/docs/sw/m1n1-hypervisor.md)の翻訳

訳注:Asahi Linux内のページへのリンクは対応する日本語訳に置き換え

---

# m1n1ハイパーバイザーでのmacOSの実行

Apple から入手した開発用カーネルを実行することができます。この場合デバッグシンボルを得ることができますし、macOS インストール内の純正カーネルを使用することもできます。

## 対応 macOS バージョン

Apple はバージョン間で ABI 互換性を保証していないため、逆解析の対象としても、ハイパーバイザーのゲストとしても、すべての macOS バージョンに対応することはできませんし、対応するつもりもありません。現在対応している対象は以下の通りです：

- macOS 13.5（M1およびM2シリーズ）
- macOS 14.8.3（M1〜M3シリーズ）

多数のバグの存在と XNU を起動するために SPTM が動作している必要があるので、M4 およびそれ以降の機器向けの対象バージョンはまだ定めてられていません。

## 準備
メインの macOS インストールのブートオブジェクトを非破壊的に上書きすることは可能ですが、この方法は推奨されません。
データ損失のリスクが高まるだけでなく、特定の macOS バージョンを対象にするたびに機器全体を DFU復元しなければならなくなるため、非常に手間がかかります。代わりに、macOS の2つ目のコピーをインストールします。

1. macOS パーティションにセカンドボリュームを作成します。

        diskutil apfs addVolume disk4 APFS macOSTest -mountpoint /Volumes/macOSTest

disk4 とボリューム名 (つまり macOSTest) を特定のシステム/環境設定用に変更します。 

_注: このロールをシステムロールにしてはいけません。既存のシステム が混乱します (1TR内に有効なユーザーが存在しない)_

2. macOS をダウンロードしインストールします。macOS の特定のバージョンのインストーラをダウンロードするには次のコマンドを使用します。

        softwareupdate --fetch-full-installer --full-installer-version 14.8.3

`14.8.3` は必要なバージョンに置き換えます。インストーラはアプリケーションフォルダ内にあり、保存したい場合はここからコピーしてください。
そうでない場合は、一度インストールすると自動的に削除されます。

### アーカイブされたInstallAssistant.pkgの使用

残念ながら、Apple は CDN 上に macOS インストーラーを無期限に保持していません。上記の方法でフルインストーラーを取得できない場合は、アーカイブされた InstallAssistant.pkg を使用する必要があります。関連する InstallAssistant バージョンの動作確認済みアーカイブは以下の通りです：

| macOS バージョン | リンク |
| ------------- | --------------------------------------------------------------------- |
| 13.5 Ventura  | [archive.org](https://archive.org/details/install-assistant_20240930) |
| 14.8.3 Sonoma | [archive.org](https://archive.org/details/install-assistant_20250207) |

InstallAssistant.pkg をダウンロードして実行してください。これにより、`Install macOS [version].app` アプリケーションが `/Applications` に展開されます。インストールされたアプリケーションを実行し、画面の指示に従って、先ほど作成した APFS ボリュームに macOS をインストールします。

## カーネルの取得

m1n1 に渡すための XNU イメージが必要です。上記でインストールした macOS バージョンに同梱されている製品版カーネルを使用するか、または strip されていない Kernel Debug Kit（KDK）カーネルを使用することができます。

### 製品版カーネル

製品版カーネルを使用するのが最も簡単です。Apple ID や特別な準備作業は不要です。

1. 上記でインストールしたmacOSのコピーに起動
2. `/System/Volumes/Preboot/[UUID]/boot/[hash]/System/Library/Caches/com.apple.kernelcaches/` から `kernelcache` ファイルを取得し、ホストマシンにコピー
3. ホストマシンに [img4tool](https://github.com/tihmstar/img4tool) をインストール
4. IM4PからMach-Oバイナリを抽出する：
        ```sh
        img4tool -ep out.im4p kernelcache
        img4tool -eo kernelcache.macho out.im4p
        ```

### KDK カーネル

KDK の使用はより手間がかかりますが、ブラインドトレーシングでは不十分な特定のケースで有用な場合があります。

1. インストールした macOS バージョン用の macOS KDK を Apple の [こちら](https://developer.apple.com/download/more/) からダウンロード
   Apple ID と無料の Apple Developer Account が必要
3. KDK をインストール。インストール先は `/Library/Developer/KDK/KDK_[macOS ver]_[KDK ver].kdk/` 
4. ターミナルを開き、KDKのカーネルディレクトリに移動：
        `cd /Library/Developer/KDK/KDK_[macOS ver]_[KDK ver].kdk/Kernels`
5. 自分のマシン用のkernelcacheを作成：
        ```sh
        kmutil create -zn boot -a arm64e -B ~/dev.kc.macho -V development \
          -k kernel.development.t8103 \
          -r /System/Library/Extensions/ \
          -r /System/Library/DriverExtensions/ \
          -x $(kmutil inspect -V release --no-header | grep -v "SEPHiber" | awk '{print " -b "$1; }')
        ```

  `-B` は kernelcache が書き出される場所を指定します。
  `-k` は `Kernels` ディレクトリ内のファイルと一致させる必要があり、Mac の SoC に固有のものです。上記の例では T8103（M1）カーネルを使用しています。

Apple は KDK に各カーネル用の DWARF も同梱しています。これらは m1n1 に渡してシンボル解決に使用できます。  
場所は `/Library/Developer/KDKs/KDK_[macOS ver]_[KDK ver].kdk/System/Library/Kernels/kernel.development.[SoC].dSYM/Contents/Resources/DWARF/kernel.development.[SoC]` です。

これらを使用したい場合は、作成した kernelcache と一緒にホストマシンにコピーする必要があります。

## m1n1 を macOS カーネルとしてインストール

2 つ目の macOS コピーを起動ボリュームとして選択したときに、プラットフォームが XNU ではなく m1n1 にジャンプするように設定する必要があります。  
そのためには、その APFS ボリュームのセキュリティモデルをダウングレードして未署名コードの実行を許可し、m1n1 をそのブートオブジェクトとして構成します。

1. 2 つ目の macOS コピーがマシンのデフォルトの起動ディスクになっていることを確認
2. 1TR を起動して Terminal を開く
3. `diskutil info [disk]` を実行して、macOS ボリュームの UUID を確認
4. `bputil -nkcas` を実行してボリュームのセキュリティをダウングレード。正しい UUID を選択していることを確認
5. `bputil` を実行すると SIP がリセットされるため、`csrutil disable` を実行して明示的に無効化
6. 詳細ログを有効化：`nvram boot-args=-v`
7. [m1n1](m1n1-user-guide.md) をカスタムブートオブジェクトとしてインストール：

        ```sh
        kmutil configure-boot \
          -c build/m1n1.bin \
          --raw \
          --entry-point 2048 \
          --lowest-virtual-address 0 \
          -v /Volumes/macOSTest
        ```

## m1n1のゲストとしてXNUを起動

m1n1 をブートオブジェクトとしてインストールしたので、これで m1n1 のハイパーバイザーのゲストとして XNU を起動できます。Mac をシャットダウンし、ホストマシンに** DFU ポート経由で**接続してから、電源を入れてください。m1n1 が起動し、シリアルプロキシが開始されるはずです。

### 製品版カーネル

製品版カーネルをm1n1のゲストとして起動するのは非常に簡単です：

```sh
python3 proxyclient/tools/run_guest.py \
    path/to/kernelcache.macho \
    -- "debug=0x14e serial=3 apcie=0xfffffffe -enable-kprintf-spam wdt=-1 clpc=0"
```

### KDK カーネル

KDK を使用して開発用 kernelcache を作成した場合、m1n1 にデバッグシンボルを渡すことができます：

```sh
python3 proxyclient/tools/run_guest.py \
    -s path/to/DWARF \
    path/to/kernelcache.macho\
    -- "debug=0x14e serial=3 apcie=0xfffffffe -enable-kprintf-spam wdt=-1 clpc=0"
```


## ハイパーバイザーモジュールの使い方

m1n1 シェルだけでハードウェアをトレースすることは可能ですが、あまり使いやすくありません。代わりに、m1n1 は Python スクリプトの事前読み込みと実行に対応しています。これらのスクリプトは m1n1 の Python API に完全にアクセスできます。

主な用途は、特定のハードウェアブロック向けの高機能な自動トレーサーを構築することです。先行事例は `proxyclient/hv/` にあります。これらはモジュールとして `run_guest.py` に渡します：

```sh
python3 proxyclient/tools/run_guest.py \
    -m proxyclient/hv/trace_dcp.py \
    path/to/kernelcache.macho \
    -- "debug=0x14e serial=3 apcie=0xfffffffe -enable-kprintf-spam wdt=-1 clpc=0"
```

上記の例では、DCP トレーサーを事前読み込みし、m1n1 が XNU にジャンプする準備ができた時点ですぐに起動します。

## m1n1 の ABI 同期

m1n1 の ABI は安定しておらず、ソースツリー内のリソース（例：`proxyclient`）は、古い m1n1 ビルドとの後方互換性が保証されることはありません。ハイパーバイザーを使用する前に、常に作業ツリーからビルドした m1n1 をチェインロードして ABI 互換性を確保することが想定されています：

```sh
python3 proxyclient/tools/chainload.py build/m1n1.macho
```

## デバッガの使用

m1n1 はネットワーク経由のデバッグをサポートしています。ハイパーバイザーシェルで `gdbserver` を実行すると、GDB または LLDB から接続可能なデバッグサーバー実装が起動します。Mach-O、ポインタ認証、および XNU の dyld の使用に対するサポートが優れているため、LLDB の使用を推奨します。

すべてのカーネル拡張のシンボルを読み込むには、LLDB スクリプトが必要です。以下のシェルスクリプトを macOS 上で使用すると、この作業を行ってくれる LLDB スクリプトを生成できます：

```sh
echo 'target create -s kernel.development.t8103.dSYM kernel.development.t8103' > target.lldb
for k in $(find Extensions); do
    [ "$(file -b --mime-type $k)" != 'application/x-mach-binary' ] || printf 'image add %q\n' $k;
done >> target.lldb
```

上記は、macOS 上で起動しており、KDK の `Kernels` ディレクトリ内にいることを前提としています。

LLDB から、生成したスクリプトを実行し、m1n1 のデバッグサーバーに接続できます：

```
command source -e false target.lldb
process connect unix-connect:///tmp/.m1n1-unix
```

ハイパーバイザーシェルの組み込みデバッグ機能と外部デバッガを同時に使用しないでください。たとえば、LLDBを使用中にハイパーバイザーシェルからブレークポイントを追加・編集しないでください。

# 情報源
kernelcache作成の情報源: 
[https://kernelshaman.blogspot.com/2021/02/building-xnu-for-macos-112-intel-apple.html](https://kernelshaman.blogspot.com/2021/02/building-xnu-for-macos-112-intel-apple.html)
