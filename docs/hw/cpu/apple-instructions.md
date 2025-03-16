---
title: Apple プロプライエタリ命令
summry:
A46 命令セットに対する Apple のプロプライエタリな拡張
---

2025/3/1時点の[apple-instructions](https://github.com/AsahiLinux/docs/blob/main/docs/hw/cpu/apple-instructions.md)の翻訳

---
Appleのプロプライエタリな命令が0x0020xxxxの範囲にあります。

```
00200000 - 002007ff MUL53、 https://gist.github.com/TrungNguyen1909/5b323edda9a21550a1621af506e8ce5f を参照

00200800｜rD << 5｜rS wkdmc、 メモリページの圧縮
   - rS は圧縮元ページアドレス（ページ境界にアライン、下位ビットは無視）
   - rDは圧縮先データアドレス（64b境界にアライン、下位ビットは無視）
   - ステータス/情報はrSで返す

00200c00｜rD << 5｜rS wkdmd、メモリページの伸長
   - rSは圧縮済データアドレス（64b境界にアライン、下位ビットは無視）
   - rDは圧縮データ伸長先アドレス（ページ境界にアライン、下位ビットは無視）
   - ステータス/情報はrSで返す

00201000 - 002012df AMX、 https://gist.github.com/dougallj/7a75a3be1ec69ca550e7c36dc75e0d6f を参照
   AMXが有効でない場合(デフォルト)、ESR_EL2 = 0xfe000003でフォールト発生

   ...222～23f 未知の命令の『ホール(hole)』
    
002012e0 - 0020143f 未知の命令によるフォールト

*00201400 gexit　　　　　　　ガードモードを終了。macOSで使用。何らかの有効化が必要（デフォルトではフォールト）
*00201420 genter | imm5 　 ガードモードを開始。macOSで使用。何らかの有効化が必要（デフォルトではフォールト）
    imm5 は ESR_GLx[5:0] に保存

00201440｜rA at_as1elx、アドレスを変換。同じレジスターで返す。
   [63:56] 変換用 MAIR 属性 (インデックスではない！)
   [??:12] 物理アドレス
   [11:00] フラグ/ステータス/その他。 0x80x = unmapped, x はフォールトが発生したPTレベルに応じて変化？

これはPAR_EL1システムレジスタと同じのようで、ARMの*公式*翻訳アドレス命令の出力として使用されるらしいです。

00201460                       sdsb osh
00201461                       sdsb nsh
00201462                       sdsb ish - iBootトランポリンが使用
00201463                       sdsb sy

00201464 ~ 未知の命令によるフォールト


```
