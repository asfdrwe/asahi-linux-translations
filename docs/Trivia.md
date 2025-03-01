2025/3/1時点の[Trivia](https://github.com/AsahiLinux/docs/blob/main/docs/Trivia.md)の翻訳

---
このプラットフォームとそのレガシーに関するランダムな楽しいトリビア一覧。

# iPhoneレガシー
## M1はiPhone5だ
M1 Mac Miniをディスプレイを接続せずに起動した場合、またはmacOS 12.0以降では無条件に、iBootはディスプレイを初期化しません。
代わりに640×1136の偽フレームバッファを作成します。これはiPhone 5の画面解像度です。

するとこんなことが起こります。

![Framebuffer](https://github.com/AsahiLinux/docs/blob/main/docs/assets/m1_iphone_5_fb.png)

## どうしてもSamsungから離れられない
M1のシリアルポートペリフェラルはSamsungの初代iPhoneのSoC（S5L8900）のものと同一で、Linuxでも同じSamsungのUARTドライバを
使っているほどです。最近のはAppleによる再実装されたものなのか昔の同一のSamsungのIPをそのままライセンスしているものなのかは分かりません。

実際のところApple A4がAppleによる最初の『自社設計』だというのは、主にマーケティングによるものです。AppleのSoCは、
サードパーティ製IPからApple製IPへの移行をゆっくりと進めてきましたが、今でもサードパーティ製のブロックを
ふんだんに使っています。サードパーティ設計と自社設計の間に明確な線引きはないのです。

# PowerPCレガシー

## HIDden レジスタ
AppleのCPUコアは雑多な設定や[chiken bit](https://en.wiktionary.org/wiki/chicken_bit)レジスタを『HIDx』レジスタと呼び、
これは『Hardware Implementation Dependent(ハードウェア実装依存)』レジスタを意味します。この名前は同じ目的のためにIBMが
PowerPC CPUで最初に使用しました。

## Power Mac G5まで遡る(DARTing back)
AppleのSoCに搭載されているIOMMUは『DART』と呼ばれています。これは『Device Address Resolution Table』を表し、Power Mac G5
システムのU3HホストブリッジのIOMMUの名称でした。しかし、実際の詳細とは無関係なので、これに関して共有コードはなく、名前が同じなだけです。

## AmigaOne X1000 と M1 Mac の共通点は何?

M1や最近のAppleのSoCのI²CペリフェラルはP. A. Semi社のPWRficient PA6T-1682MチップのI²Cペリフェラルの修正版です。
AppleはSoC/CPU設計チームを立ち上げるために同社を買収し、あるIPブロックはそのままで十分だと判断したのです。
これはAmigaOne X1000に使われているCPUと同じもので、私たちは既存のLinuxドライバを拡張して両方のプラットフォームに
対応するようにしました。

# x86レガシー

## M1はネイティブにx86コードを実行
Mac Miniと14インチ/16インチMacBook Proに搭載されているDisplayPort to HDMIブリッジチップ（MDCP29xx）はV186 CPUコアを
使用しています。これはIntel 80186クローンであり、古き良き16ビットx86リアルモードコードを実行します。そう、MS-DOS時代のx86が
あなたのMacに搭載されているのです。

# 細部へのこだわり

## 助けて!
Mac の [SecureROM](SW-Boot.md#stage-0-securerom) は
小さく、それ自身では大したことは
できません。Mac mini では画面に画像を表示することはできません。しかし電源LEDを制御することはできます。
[DFUモード](Glossary.md#d)でMacを起動すると、
LEDは白ではなく琥珀色になります。Macを通常通り起動したが初期起動に失敗した場合（リストア操作の失敗など）、電源LEDは琥珀色になり、
短い点滅3回、長い点滅3回、短い点滅3回、一時停止、それを繰り返すパターンで点滅します。これは
[SOSのモールス信号](https://en.wikipedia.org/wiki/Morse_code#Applications_for_the_general_public)です！
Macは静かに救出を求めています...
