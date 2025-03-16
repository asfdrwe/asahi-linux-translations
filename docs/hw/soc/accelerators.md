---
title: Apple Silicon アクセラレーター
---

2025/3/9時点の[Accelerator Engines](https://github.com/AsahiLinux/docs/blob/main/docs/hw/soc/accelerators.md)の翻訳

訳注: 
- 文書へのリンクは対応する日本語訳へのリンクに書き換え

---
このSoCにはいくつかのオンボードアクセラレータユニットがあります。ほとんどのアクセラレータは、pre-boot パーティション `/System/Volumes/Preboot/[UUID]/restore/Firmware` にあるファームウェアを実行しており、im4pファイルとしてパッケージされています。このファイルは、https://github.com/19h/ftab-dump/blob/master/rkos.py とddで抽出できます。

* ANEやAVE、ADTのim4pのどれも別のものと一緒に抽出されないようにアップデートしてください。
[ADT](../../fw/adt.md)にあるim4p抽出の手順に従うとよいでしょう。
ファームウェアに関する進捗状況表を作成できませんか? (訳注:wiki更新作業指示のメモ?)

## 名称

名称はどのくらい公式に使われているかによって以下のようにフォーマットされます。
* クエスチョンマーク付きの引用符で囲まれた名称:『<name>?』のように疑問符で囲まれた名称は独自に命名したものや起源が不明なもの
* **太字** で書かれた名称: **<name>**のような**太字**の名前は、Appleの公式ドキュメントに記載
* *イタリック*で表記されている名称: *<name>* は一般的な非公式名称または不確かだが安全な起源

### A
* **AGX**: 『Apple Graphics? Accel(x)lerator?』 (`gfx`経由) AppleのGPUシリーズの内部名称
* **AMX**: *Apple Matrix eXtensions*。ISAに部分的に組み込まれた行列コプロセッサ
* **ANE**: **Apple Neural Engine**。 畳み込みに基づくニューラルネットワークの実行アクセラレーション。GoogleのTPUを想起
* **AOP**: **Always On Processor**。『Hey siri』の起動や『その他のセンサー関連』
* **APR**: **APR ProRes**。ProResビデオエンコード＋デコードを担当
* **AVE**: **AVE Video Encoder**。.ビデオのエンコードを担当。表向きのAはAppleのことですが[要出典]、再帰的な頭字語
* **AVD**: **AVD Video Decoder**。ビデオのデコードを担当^

### D
* **DCP**: 『Display Compression Processor?』/『Display Control Processor?』。ある種のディスプレイポート/ディスプレイコントロール

### P
* **PMP**: 『Power Management Processor?』。 電源機能を処理

### S
* **SEP**: **Secure Enclave Processor**。M1に内蔵されたHSM/TPM/その他のデバイス。Touch IDやほとんどの暗号、ブートポリシーの決定などを担当。Linuxには無害だが望めばその機能を使うことができる。APとは対照的
