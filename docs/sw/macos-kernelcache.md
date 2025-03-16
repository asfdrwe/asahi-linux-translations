---
title: macOS カーネルキャッシュ
---

2025/3/9時点の[macos-kernelcache](https://github.com/AsahiLinux/docs/blob/main/docs/macos-kernelcache.md)の翻訳

訳注: プロジェクトページへのリンクは対応する日本語訳へのリンクに置き換え

---
# 先に進む前にこれをお読みください

**Asahi Linuxは非常に厳しい[リバースエンジニアリング方針](https://github.com/asfdrwe/asahi-linux-translations/blob/main/copyright.md)を
定めています。この方針を十分に読み、理解しない限り、Darwinカーネルを含むmacOSのコードの逆アセンブルを開始しないでください。**

私たちは、バイナリリバースエンジニアリングをプロジェクトへの貢献の一部として使用したい貢献者は、事前に私たちと具体的な内容を議論することを期待します。
これは通常クリーンルームの環境を手配することを意味し、そこでは彼らの仕事は関連するサブシステムについてソースコードではなく仕様を書くことのみとなります。

これを守らないと私たちのプロジェクトに直接コードを提供することが禁止されるかもしれません。あなたは警告されています。

**macOSのバイナリを配布することは全体でも一部でも著作権違反です**。あなたは、Appleのコンピュータ上でmacOSのあなた自身のインストールから
ファイルを抽出する必要があります。

繰り返しになりますが、**この件に関して私たちに最初に相談した場合のみ作業を進めてください。**

## Darwinカーネルキャッシュの抽出

* インストールしたOSのPrebootパーティションにあるkernelcacheを探す
* [img4tool](https://github.com/tihmstar/img4tool)を入手
* `img4tool -a kernelcache -e -p kernelcache.im4p -m kernelcache.im4m`
* `img4tool kernelcache.im4p -e -o kernelcache.macho`
* 結果は標準的なMach-Oファイル。Mach-O toolchain が手元にない場合は
[machodump.py](https://gist.github.com/marcan/e1808a2f4a5e1fc562357550a770afb1) を使ってヘッダを見ることが可能

## kernel.release.* ファイルを使った代替案

* **davidrysk氏** の提案により、いくつかの MacOS カーネルイメージが **/System/Library/Kernels/kernel.release.t8020** で既に利用可能
* 下記は、コードを逆アセンブルするためのオフセットを得るために、Marcan氏のスクリプト 
[machodump.py](https://gist.github.com/marcan/e1808a2f4a5e1fc562357550a770afb1) でmachoヘッダーをダンプしたもの
 * 注：これは **construct** python パッケージを必要とするが、debian buster パッケージは動作しなかった（python 3 または 2）、
あるいは github バージョンも動作しない
 * pip3経由のpypiインストールを使用する必要あり

```
 apt install python3-pip
 pip3 install construct
```

 * 次にヘッダーをダンプしてコードのオフセットを抽出

```
python3 machodump.py kernel.release.t8020
...
            cmd = (enum) SEGMENT_64 25
            args = Container: 
                segname = u'__TEXT' (total 6)
                vmaddr = 0xFFFFFE0007004000
                vmsize = 0x00000000000C4000
                fileoff = 0x0000000000000000
                filesize = 0x00000000000C4000
...
        Container: 
            cmd = (enum) UNIXTHREAD 5
            args = ListContainer: 
                Container: 
                    flavor = (enum) THREAD64 6
                    data = Container: 
                        x = ListContainer: 
                            0x0000000000000000
...
                            0x0000000000000000
                        fp = 0x0000000000000000
                        lr = 0x0000000000000000
                        sp = 0x0000000000000000
                        pc = 0xFFFFFE00071F4580
                        cpsr = 0x00000000
                        flags = 0x00000000
....
```

* 開始命令 pc=0xFFFFFE00071F4580 から VM の開始点 (vmaddr=0xFFFFFE0007004000) までのオフセットを計算

```
calc "base(16); 0xFFFFFE00071F4580 - 0xFFFFFE0007004000"
        0x1f0580
```

* 最初の 0x1f0000 = 0x1f0 x 0x1000 (4k) ブロックはスキップして、ここから 64K を分割

```
dd if=/home/amw/doc/share/kernel.release.t8020 of=init.bin bs=4k skip=$((0x1f0)) count=16
```

* 生のバイナリブロブを逆アセンブル

```
aarch64-linux-gnu-objdump -D -b binary -m aarch64 init.bin

init.bin:     file format binary

Disassembly of section .data:

0000000000000000 <.data>:
       0:       14000100        b       0x400
       4:       d503201f        nop
       8:       d503201f        nop
       c:       d503201f        nop
      10:       d503201f        nop
      14:       d503201f        nop
      18:       d503201f        nop
      1c:       d503201f        nop
      20:       d503201f        nop
...
     3f4:       d503201f        nop
     3f8:       d503201f        nop
     3fc:       d503201f        nop
     400:       d510109f        msr     oslar_el1, xzr
     404:       d5034fdf        msr     daifset, #0xf
     408:       f2e88aa0        movk    x0, #0x4455, lsl #48
     40c:       f2c80a80        movk    x0, #0x4054, lsl #32
     410:       f2ac8cc0        movk    x0, #0x6466, lsl #16
     414:       f28c8ee0        movk    x0, #0x6477
     418:       90003fe4        adrp    x4, 0x7fc000
     41c:       3944c085        ldrb    w5, [x4, #304]
     420:       710000bf        cmp     w5, #0x0
...
```
* **davidrysk氏** の指摘のようにXCodeをインストールした実際のMacではより簡単に可能

```
otool -xv /System/Library/Kernels/kernel.release.t8020

...
/System/Library/Kernels/kernel.release.t8020:
(__TEXT_EXEC,__text) section
fffffe00071ec000        sub     x13, sp, #0x60
fffffe00071ec004        sub     sp, sp, #0x60
fffffe00071ec008        st1.4s  { v0, v1, v2 }, [x13], #48 ; Latency: 4
...
fffffe00071f43f8        nop
fffffe00071f43fc        nop
fffffe00071f4400        msr     OSLAR_EL1, xzr
fffffe00071f4404        msr     DAIFSet, #0xf
fffffe00071f4408        movk    x0, #0x4455, lsl #48
fffffe00071f440c        movk    x0, #0x4054, lsl #32
fffffe00071f4410        movk    x0, #0x6466, lsl #16
fffffe00071f4414        movk    x0, #0x6477
fffffe00071f4418        adrp    x4, 2044 ; 0xfffffe00079f0000
fffffe00071f441c        ldrb    w5, [x4, #0x130]        ; Latency: 4
fffffe00071f4420        cmp     w5, #0x0
fffffe00071f4424        b.ne    0xfffffe00071f4438
....
```
