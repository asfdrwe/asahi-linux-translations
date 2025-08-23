---
title: MachO ブートプロトコル
summary:
m1n1 を MachO バイナリとして起動するときに Apple Silicon 機器で使われるブートプロトコル
---

2025/8/22時点の[macho-boot-protocol](https://github.com/AsahiLinux/docs/blob/main/docs/fw/macho-boot-protocol.md)の翻訳

---
## ブートプロトコル

### メモリ

メモリは0x8_0000_0000 から始まります。

iBootから呼び出されたときのメモリは以下のようになります:


```
+==========================+ <-- RAMの最下位 (0x8_0000_0000)
| コプロセッサのカーブアウト   |
| iBoot用のものなど　        |
+==========================+ <-- boot_args->phys_base, VM = boot_args->virt_base
| kASLR slide gap (<32MiB) |
+==========================+
| Device Tree (ADT)        | /chosen/memory-map.DeviceTree
+--------------------------+
| Trust Cache              | /chosen/memory-map.TrustCache
+==========================+ <-- Mach-O lowest vmaddr mapped to here (+ slide!)
| Mach-O base (header)     | /chosen/memory-map.Kernel-mach_header
+--                      --+
| Mach-O segments...       | /chosen/memory-map.Kernel-(segment ID)...
+--                      --+
| m1n1: Payload region     | /chosen/memory-map.Kernel-PYLD (64MB currently)
+==========================+
| SEP Firmware             | /chosen/memory-map.SEPFW
+--------------------------+ <-- boot_args
| BootArgs                 | /chosen/memory-map.BootArgs
+==========================+ <-- boot_args->top_of_kdata
|                          |
|      (フリーメモリ)       |
| (incl. iBoot trampoline) |
|                          |
+==========================+ <-- boot_args->top_of_kdata + boot_args->mem_size
| Video memory, SEP        |
| carveout, and more       |
+==========================+ <-- 0x8_0000_0000 + boot_args.mem_size_actual
```

### ポインタについて

たどる可能性があるアドレスには4つの種類があります。

* 物理アドレス(physical address)
* m1n1 未再配置オフセット (0からの相対値)
* Mach-O 仮想アドレス(virtual address)
* kASLR-slid 仮想アドレス

気にすべきものは物理アドレスだけです。

m1n1 未再配置オフセットは再配置を実行する前の m1n1 起動コードと関連するリンカースクリプト情報によってのみ使用されます。
C環境はこれらの後に適切に位置調整されるので、これらを見ることはありません。
しかし、m1n1をデバッグしてポインタを表示し生のELFにマッピングしたい場合は、
未再配置のオフセットを得るためにm1n1のロードオフセットを引く必要があります。

仮想アドレスは重要ではありません。
これはMach-Oが物理アドレスの概念を持たないために使用されているだけで、全体のセットアップはDarwinが特定の方法で自分自身をマッピングすることを想定しています。
我々にとっては、vaddrが`paddr + ba.virt_base - ba.phys_base`になっているだけです。
m1n1は上半分の仮想アドレスを使用せず、LinuxはDarwinが行っていることとは関係のない独自のことをしています。

さらに、仮想アドレスマップは2つあります：Mach-Oの中にあるものと、iBootが実際に渡してくるポインタです。後者は kASLR-slidによってオフセットされ、これは vaddrs にも影響します。これがより混乱をもたらします。

よって、iBoot から受け取ったDarwinのkASLR-slidの仮想ポインタに対して、vaddr - ba.virt_base + ba.phys_base` を計算します。それがすべてです。
逆に、リンカースクリプト(とその中のMach-Oヘッダ生成)だけがMach-Oのunslid仮想アドレスを気にします。本当にあまり深く考えようとしないでください。
混乱するだけです。

### エントリー

iBoot は、Mach-O データ構造で定義されているエントリポイントで、unslid の
vaddrです。エントリーはMMUがオフの状態です。`x0`は[boot_args structure](https://github.com/AsahiLinux/m1n1/blob/main/src/xnuboot.h)を指します。

さらに、iBootは、ブートCPUのRVBARを、エントリポイントがエントリポイントが存在するページの先頭に設定してロックします。
これは起動後に変更することができないため、このアドレスは常に特別な意味を持ち、常駐のブートローダコードとして扱われる必要があります。
今のところ実用的な意味は不明ですが、おそらくディープスリープからの復帰後、ブートCPUはここでコードの実行を開始すると思われます。
ここでコードを実行します。なお、これは実際のCPUベクター(`VBAR_EL2`内で自由に変更可能)をロックするものではなく、
また、セカンダリCPUのRVBAR（スタートコマンド発行前に自由に設定可能）にも影響を与えません。

## m1n1 メモリレイアウト

m1n1を初期起動すると関連するメモリは以下のようになります:

```
+==========================+
| Device Tree (ADT)        | /chosen/memory-map.DeviceTree
+--------------------------+
| Trust Cache              | /chosen/memory-map.TrustCache
+==========================+ <-- _base
| Mach-O header            | /chosen/memory-map.Kernel-_HDR
+--                      --+ <-- _text_start, _vectors_start
| m1n1 .text               | /chosen/memory-map.Kernel-TEXT
+--                      --+
| m1n1 .rodata             | /chosen/memory-map.Kernel-RODA
+--                      --+ <-- _data_start
| m1n1 .data & .bss        | /chosen/memory-map.Kernel-DATA
+--                      --+ <-- _payload_start
| m1n1 Payload region      | /chosen/memory-map.Kernel-PYLD (64MB currently)
+==========================+ <-- _payload_end
| SEP Firmware             | /chosen/memory-map.SEPFW
+--------------------------+ <-- boot_args
| BootArgs                 | /chosen/memory-map.BootArgs
+==========================+ <-- boot_args->top_of_kdata, heap_base
| m1n1 heapblock           | (>=128MB)
+--                      --+ <-- ProxyUtils.heap_base (m1n1 heapblock in use end + 128MB)
| Python heap              | (1 GiB)
+--                      --+
|      (Unused memory)     |
+==========================+ <-- boot_args->top_of_kdata + boot_args->mem_size
```

m1n1のヒープブロック領域(mallocのバックエンドやペイロードのロードに使用)は`boot_args.top_of_kdata`から始まり、現時点では割り付けられていません。
proxyclient を使用している場合、ProxyUtils は現在のヒープブロックトップより 128MiB 上の Python ヒープベースを設定し、m1n1 は Python 側の構造にぶつかる前に最大 128MiB の追加メモリを使用できることになります。Python 側の新しい実行は、現在の m1n1側が何であれヒープを再初期化することに注意してください。つまり、 Python の各実行時に m1n1 側のメモリリークがあっても、RAM の総量が足りなくなるまではすぐには問題になりません。

別の Mach-O ペイロードをチェーンロードする場合、次のステージでは m1n1 をその場で上書きします。chainload.py の Mach-O ロードコードは、m1n1 ペイロードセクションのパディングエンドをスキップします(マーカーとしての4つのゼロバイトを除く)。そのため、SEP ファームウェアと BootArgs は、他の方法では m1n1 ペイロードエリアになるはずだった場所に直接続くので、RAM を節約することになります。SEP ファームウェアの再配置はオプションで、有効になっていない場合はそのままでtop_of_kdata は変更されません。m1n1がペイロード領域のサイズよりも大きくならない限り安全なはずです。
