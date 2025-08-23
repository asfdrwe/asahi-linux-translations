---
title: Apple Device Tree (ADT)
summary:
Apple Device Tree, Apple Silicon 機器で使われるハードウェア検出・初期化システム
---

2025/8/22時点の[adt](https://github.com/AsahiLinux/docs/blob/main/docs/fw/adt.md)の翻訳

---

Appleのファームウェアはカーネルを起動するときにバイナリ形式のDeviceTreeを渡します。このフォーマットは、Linuxが想定する
Open Firmware標準に非常に似ていますが、異なります。

LinuxのDeviceTreeと同様に、AppleのDeviceTree（ADT）は、ノードの階層にいくつかの型付けされていないバイト配列
（プロパティ）をエンコードしています。これらは、利用可能なハードウェアを記述したり、Appleがファームウェアがカーネルに
伝える必要があると考えるその他の情報を提供します。これにはシリアル番号やWiFiキーのような識別情報や秘密情報が含まれます。

ADTとLinuxのDTの主な違いはバイトオーダーです。プロパティは型付けされていないため、自動的に補正することはできません。

## ADTの取得

ハードウェアがある場合、いくつかの方法でADTにアクセスすることができます。

### オプション1：m1n1デバッグコンソール経由
最も簡単な方法は、adt.pyを使用してm1n1を使用することでしょう。

```
cd m1n1/proxyclient ; python -m m1n1.adt --retrieve dt.bin
```

これにより、生の（バイナリの）ADTを含む『dt.bin』というファイルが書き込まれ、デコードされたADTが表示されます。

### オプション2: macOS im4pファイル経由 (注意: これらはブート時にiBootによって埋められる詳細が欠落)
### img4lib
xerub氏の img4lib を入手します

```
git clone https://github.com/xerub/img4lib
cd img4lib
make -C lzfse
make
make install
```

### img4tool
tihmstar氏の img4tool を入手します (作者のlibgeneralに加え autoconf, automake, libtool, pkg-config, openssl, libplist-2.0 も必要です)。

```
git clone https://github.com/tihmstar/libgeneral.git
git clone https://github.com/tihmstar/img4tool.git
```

次にそれぞれに対して

```
./autogen.sh
make
make install
```

### Device Tree ファイルを取得
im4pファイルを下記ディレクトリからコピーしてください。Machine 'j' モデルの詳細については、[デバイス](../hw/devices/device-list.md)を参照してください。

`/System/Volumes/Preboot/[UUID]/restore/Firmware/all_flash/DeviceTree.{model}.im4p`

ディレクトリが存在しない場合、リカバリーモードでcsrutilを無効化し、設定からターミナルですべてのファイルにアクセスできるようにするか、
またはシンボリックリンクされている可能性があるので `Volumes/Macintosh HD/` から起動してみてください。
それでもアクセスできない場合は、以前からの`sudo find . -type f -name '*.im4p'`を試してみてください。

そして、img4tool を使って dt.bin を抽出します。

```
img4tool -e DeviceTree.j274ap.im4p -o j274.bin
```

img4libで同様の処理をするために、以下を実行します。

```
img4 -i DeviceTree.j274ap.im4p -o j274.bin
```


### オプション3：macOSより

macOSから直接ADTのテキスト表現を取得するには、次のように実行します。

```
ioreg -p IODeviceTree -l | cat
```

これはデコードする必要はありませんが、m1n1を使う(後述)よりはるかに少ない情報しか出力されません。

## ADTのデコード

m1n1インストール後([リポジトリページ](https://github.com/AsahiLinux/m1n1) を参照)

`cd m1n1/proxyclient`

contruct python libraryを取得(construct.pyファイルでない、それはライブラリ)

`pip install construct`

j{*}.binを含むファイルをproxyclientディレクトリにコピーし、次のように抽出:

`python -m m1n1.adt j{*}.bin`

また-aオプションでメモリマップを取得することができます。

`python -m m1n1.adt -a j{*}.bin` 

他に方法ありますか？
