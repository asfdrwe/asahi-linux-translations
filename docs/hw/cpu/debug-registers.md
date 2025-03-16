---
title: アプリケーションプロセッサデバッグレジスタ
summary:
  Apple 設計の ARM コアで見つかったデバッグレジスタ
---

2025/3/9時点の[HW-CPU-debug-registers](https://github.com/AsahiLinux/docs/blob/main/docs/hw/cpu/debug-registers.md)の翻訳

---
様々なCPUコアがデバッグレジスタの存在を示唆するエントリを[ADT](../../fw/adt.md)にエクスポートしています。『coresight』という文字列が
見られ、オフセット`0xfb0`に `0xc5acce55` を書き込むことでcoresightレジスタファイルのロックが解除されます。これは Corellium の
CPU スタートコードも同じです。ロック状態レジスタはCPU0 では `0x210030fb4` にあります。

CPU0 のPC は `0x210040090`で読むことができますが（他のコアには通常のオフセットが適用されます）、他のレジスタは明示的には出現して
いないようです。
