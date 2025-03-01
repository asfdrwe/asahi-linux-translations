2025/3/1時点の[HW-CPU-debug-registers](https://github.com/AsahiLinux/docs/blob/main/docs/HW-CPU-debug-registers.md)の翻訳

訳注:
- 原文のHW-ADT.mdへのリンクミスを修正

---
様々なCPUコアがデバッグレジスタの存在を示唆するエントリを[ADT](HW-ADT.md)にエクスポートしています。『coresight』という文字列が
見られ、オフセット`0xfb0`に `0xc5acce55` を書き込むことでcoresightレジスタファイルのロックが解除されます。これはCorelliumの
CPUスタートコードも同じです。ロック状態レジスタはCPU0では `0x210030fb4` にあります。

CPU0のPCは `0x210040090`で読むことができますが（他のコアには通常のオフセットが適用されます）、他のレジスタは明示的には出現して
いないようです。
