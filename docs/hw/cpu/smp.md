---
title: 対称型マルチプロセッサ (SMP)
summary:
  Apple Silicon SoC 用のセカンダリアプリケーションプロセッサの初期化ルーチン
---

2025/3/9時点の[smp](https://github.com/AsahiLinux/docs/blob/main/docs/hw/cpu/smp.md)の翻訳

---
## SMP スピンアップ

ADTより:

* `/arm-io/pmgr[reg]` 電源管理レジスタ
    * CPUスタートブロック + 0x54000
    * CPU スタートブロックはこのレジスタに対して機器依存のオフセットが存在
        * 0x30000 A7-A8(X)
        * 0xd4000 for A9(X)-A11, T2
        * 0x54000 M1 シリーズ
        * 0x34000 M2 及びに M3
        * 0x28000 M2 Pro/Max
        * 0x88000 M3 Pro/Max
    * For multi-die systems, each die has its own power manager registers.
      The power manager registers for each die is at offset 
      `die * 0x2000000000` from the registers of die 0.
    * マルチダイシステムの場合、各ダイは独自のパワー・マネージャ・レジスタを保持
      各ダイのパワー・マネージャー・レジスタは、ダイ 0 のレジスタから `die * 0x2000000000` オフセットに存在
* `/cpus/cpu<n>[cpu-impl-reg]` CPU 実装レジスタ
* `/cpus/cpu<n>[reg]` CPU スタートアップ情報
     * ビット [0:7] は コアid
     * ビット [8:10] は クラスタid
     * ビット [11:14] は ダイid

A11 はクラスタを適切に取り扱っておらず、 高性能コアと高効率コアの CPU 両方とも 0 としています。高効率コアは 0-3 で、高性能コア は 4-5です。

古いファームウェアでは、`/cpus/cpu<n>[cpu-impl-reg]`は存在しない可能性があり、
その場合、`arm-io/reg[2*n+2]`を使用して開始アドレスを書き込む場所を見つけることができます。

PMGRのCPU スタートレジスタ:

```
offset + 0x4: システム全体の CPU コア起動/アクティブビットマスク
offset + 0x8: クラスタ0(e)の CPU コア起動
offset + 0xc: クラスタ1(p)の CPU コア起動
```

### 起動シーケンス

* スタートアドレスを RVBAR の `cpu-impl-reg + 0x00` に書き込む
    * iBootによりcpu0はロックされるが他のCPUは自由に変更可能
* `pmgr[offset + 0x4]`に (1 << cpu) を設定
    * これはシステム全体の『core alive』信号らしい。この信号はコアがスピンアップするのには不要だが、これがないとAICの割り込みやおそらくそれ以外の何かも動作しない
* `pmgr[(offset + 0x8) + 4*cluster]` に (1 << core) を設定 (0-3 のコア、0-1 のクラスタ)
    * これはコア自体を起動

コアはRVBARで起動します。chiken bits などは通常通り適用する必要があります。
