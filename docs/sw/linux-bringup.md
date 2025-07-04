---
title: Linux Bringup
---

2025/6/30時点の[linux-bringup](https://github.com/AsahiLinux/docs/blob/main/docs/sw/linux-bringup.md)の翻訳

訳注:
- ブログへのリンクは対応する日本語訳へのリンクに変更

---
# Linuxのビルド
* M1対応のためにパッチを当てた最新のAsahiカーネルはgithub linux [Asahi Kernel](https://github.com/AsahiLinux/linux)を参照
* 特に [2021年12月の進捗報告](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS20211011.md) によると、
[asahi brache](https://github.com/AsahiLinux/linux/tree/asahi) は「...最先端のカーネルであり...十分に使える状態なので...ぜひ試していただきたい」
* AsahiLinuxのrepoで開発が行われているが、M1対応のための変更はすでにlinuxのカーネルでメインライン化（
詳細は[進捗報告](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS20211011.md)の表を参照）されているので
公式のLinuxカーネルミラーをcloneすることが可能

```
git clone https://github.com/torvalds/linux.git
cd ./linux
```

* デフォルトのarm64設定を生成しsimple framebuffer対応を追加

```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
./scripts/config --set-val CONFIG_FB_SIMPLE y
```

* Linux を arm64 用にビルド

```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j8 Image dtbs
```

* m1n1のproxyclient linux.pyスクリプトはgzipで圧縮されたimageが必要

```
gzip < ./arch/arm64/boot/Image > Image.gz
```

* **注意** m1n1 の dtb: `build/dtb/t8103-j274.dtb` をUSBにより良く対応する [Asahi dart/dev kernel](https://github.com/AsahiLinux/linux/tree/dart/dev) (訳注:リンク切れ)などの別の Linux カーネルで『使用しない』。この場合**linux/arch/arm64/boot/dts/apple**の*そのカーネル*で生成されたものを使用。そうでないとUSBポートが動作しないだろう

```
cp .../linux/arch/arm64/boot/dts/apple/t8103-j274.dtb t8103-j274.dtb
```

### キーボード＋nvmeの動作
* [rev a2281d64fdbc](https://github.com/amworsley/AsahiLinux/tree/asahi-kbd)のスナップショットと
[これ](https://raw.githubusercontent.com/amworsley/asahi-wiki/main/images/config-keyboard+nvme)のような設定

## USBケーブルを挿したままの起動
* m1n1/linux は USB の低レベル PHY のセットアップをまだ行っていないので、Mac を起動する**前**に USB ケーブル/ハブ/アダプタを接続。 
[m1n1のブートのセットアップ](../platform/dev-quickstart.md#セットアップ)
でインストールしたm1n1を起動する際にiBootに実行させる
* セットアップ後に m1n1 の C コードが更新された場合は、新しい .macho イメージをチェーンロードする必要あり

```
python3.9 proxyclient/tools/chainload.py build/m1n1.macho
```

# USBケーブルでLinuxを実行
* M1 Macに
[USB Type-C to Type A/Cケーブル](../platform/dev-quickstart.md#標準的なUSBケーブルを使ったUSBガジェットモード)
を接続すると、もう一方のコンピュータに2つのUSBシリアルインターフェースが提供される

![M1 MacBookAirと2012 MacBootAir Proを接続するUSB Type-C to Type Aケーブル](https://github.com/AsahiLinux/docs/blob/main/docs/assets/usb-setup.png)

* pythonのプロキシツールで接続し、Linuxを直接起動したり、m1n1のアップデート版などのmachoなバイナリをロードしたり、Linuxイメージと組み合わせたりすることが可能
* debian arm64 インストーラから 27Mb の initrd を取得

```
wget https://deb.debian.org/debian/dists/buster/main/installer-arm64/current/images/netboot/debian-installer/arm64/initrd.gz
```

## 直接起動
* M1N1DEVICE をセットアップ
```
M1N1DEVICE=/dev/ttyACM0
export M1N1DEVICE
```

* linux.pyツールでlinuxを起動

```
python3.9 proxyclient/tools/linux.py -b 'earlycon console=ttySAC0,1500000 console=tty0 debug' \
Image.gz t8103-j274.dtb initrd.gz
```

debian arm64 installer(initrd.gz)でインストーラが起動します。もし、USBキーボードで使えるシェルが欲しい場合は、このインストーラのinitrd.gz(ramdisk)のスクリプトを変更することで、シェルを起動することができます。

ramdiskを展開:

```
gzip -cd < /pathto/initrd.gz | cpio --extract
```

initスクリプトの最後の 『exec $init』を 『exec /bin/busybox sh』に置き換えるなどinitスクリプトを修正します。
新しいramdiskを再作成: (ramdiskディレクトリより) 

```
find . | cpio -o -H newc | gzip -c -9 >| /pathto/initrd-new.gz
```

# ハイパーバイザー下での実行
* m1n1 ハイパーバイザで Linux を実行すると、メモリの検査、停止、起動のほかスタックトレースを行うことも可能
* .macho を組み合わせたイメージを作成(run_guest.py は .macho のみ受け付ける) 
```
cat build/m1n1.macho Image.gz build/dtb/apple-j274.dtb initramfs.cpio.gz > m1n1-payload.macho
```

* run_guest でロード
```
python3.9 proxyclient/tools/run_guest.py -S m1n1-payload.machao
```

<details>
<summary>フルログを見る</summary>

```
% python3.9 proxyclient/tools/run_guest.py -S m1n1-payload.macho
Fetching ADT (0x00058000 bytes)...
Disable iodev IODEV.USB1
Initializing hypervisor over iodev IODEV.USB0
TTY> Starting secondary CPUs...
TTY> Starting CPU 1 (0:1)...   Started.
TTY> Starting CPU 2 (0:2)...   Started.
TTY> Starting CPU 3 (0:3)...   Started.
TTY> Starting CPU 4 (1:0)...   Started.
TTY> Starting CPU 5 (1:1)...   Started.
TTY> Starting CPU 6 (1:2)...   Started.
TTY> Starting CPU 7 (1:3)...   Started.
Removing ADT node /arm-io/dart-usb0
Removing ADT node /arm-io/atc-phy0
Removing ADT node /arm-io/usb-drd0
Removing ADT node /arm-io/acio0
Removing ADT node /arm-io/acio-cpu0
Removing ADT node /arm-io/dart-acio0
Removing ADT node /arm-io/apciec0
Removing ADT node /arm-io/dart-apciec0
Removing ADT node /arm-io/apciec0-piodma
Removing ADT node /arm-io/i2c0/hpmBusManager/hpm0
Removing ADT node /arm-io/atc0-dpxbar
Removing ADT node /arm-io/atc0-dpphy
Removing ADT node /arm-io/atc0-dpin0
Removing ADT node /arm-io/atc0-dpin1
Removing ADT node /arm-io/atc-phy0
Removing ADT node /cpus/cpu1
Removing ADT node /cpus/cpu2
Removing ADT node /cpus/cpu3
Removing ADT node /cpus/cpu4
Removing ADT node /cpus/cpu5
Removing ADT node /cpus/cpu6
Removing ADT node /cpus/cpu7
LOAD: _HDR 16384 bytes from 0 to 0
LOAD: TEXT 131072 bytes from 4000 to 4000
LOAD: RODA 32768 bytes from 24000 to 24000
LOAD: DATA 393216 bytes from 2c000 to 2c000
ZERO: 442368 bytes from 0x8c000 to 0xf8000
LOAD: PYLD 9851973 bytes from 8c000 to f8000
SKIP: 57256891 bytes from 0xa5d445 to 0x40f8000
Total region size: 0x11b4000 bytes
Physical memory: 0x819d30000 .. 0xbccbe8000
Guest region start: 0x81ade8000
Mapping guest physical memory...
Loading kernel image (0xa5d449 bytes)...
.........................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................
Copying SEPFW (0x750000 bytes)...
Copying TrustCache (0x60000 bytes)...
Adjusting addresses in ADT...
Uploading ADT (0x4f088 bytes)...
Setting up bootargs at 0x81bf98000...
Entering hypervisor shell. Type `start` to start the guest.
>>>
```


</details>

* デバッグシェルでプロンプトが表示され、startコマンドでロードポイントから実行を開始:
<details>
<summary>ブートアップログを見る</summary>

```
Entering hypervisor shell. Type `start` to start the guest.
>>> start
Disabling other iodevs...
 - IODEV.UART
 - IODEV.FB
 - IODEV.USB1
 - IODEV.USB0_SEC
 - IODEV.USB1_SEC
Doing essential MMIO remaps...
Updating page tables...
PT[200000000:235200000] -> HW
PT[235200000:235204000] -> RESERVED VUART
PT[235204000:23b700420] -> HW
PT[23b700420:23b700424] -> RESERVED PMU HACK
PT[23b700424:23d280088] -> HW
PT[23d280088:23d28008c] -> RESERVED PMU HACK
PT[23d28008c:23d280098] -> HW
PT[23d280098:23d28009c] -> RESERVED PMU HACK
PT[23d28009c:700000000] -> HW
Improving logo...
Shutting down framebuffer...
Enabling SPRR...
Enabling GXF...
Jumping to entrypoint at 0x81b118800
Pass: mrs x0, HID5_EL1 = 2082df50e700df14 (HID5_EL1)
Pass: msr HID5_EL1, x0 = 2082df50e700df14 (OK) (HID5_EL1)
Pass: mrs x0, EHID9_EL1 = 600000811 (EHID9_EL1)
Pass: msr EHID9_EL1, x0 = 600000811 (OK) (EHID9_EL1)
Pass: mrs x0, EHID10_EL1 = 3000528002788 (EHID10_EL1)
Pass: msr EHID10_EL1, x0 = 3000528002788 (OK) (EHID10_EL1)
Pass: mrs x0, EHID20_EL1 = 618100 (EHID20_EL1)
Pass: msr EHID20_EL1, x0 = 618100 (OK) (EHID20_EL1)
Pass: mrs x0, EHID20_EL1 = 618100 (EHID20_EL1)
Pass: msr EHID20_EL1, x0 = 618100 (OK) (EHID20_EL1)
Pass: mrs x0, EHID20_EL1 = 618100 (EHID20_EL1)
Pass: msr EHID20_EL1, x0 = 618100 (OK) (EHID20_EL1)
Pass: mrs x1, CYC_OVRD_EL1 = 2000000 (CYC_OVRD_EL1)
Skip: msr CYC_OVRD_EL1, x1 = 2000000
Pass: mrs x1, ACC_CFG_EL1 = d (ACC_CFG_EL1)
Skip: msr ACC_CFG_EL1, x1 = d
```

</details>

*  実行中に **^C** を使用して、デバッグシェルを取得

```
...
Skip: msr CYC_OVRD_EL1, x1 = 2000000
Pass: mrs x1, ACC_CFG_EL1 = d (ACC_CFG_EL1)
Skip: msr ACC_CFG_EL1, x1 = d
^CUser interrupt
Entering debug shell
>>> 
```
*  カーネルのSystem.mapファイルを読み込み、PAC_MASK（ポインタ保護マスク）を設定した後のスタックトレース（シンボル付き）を取得

```
>>> load_system_map('../linux/System.map')
>>> hv.pac_mask = 0xfffff00000000000
>>> bt
Stack trace:
 - 0xffff8000102fdc3c (cpu_do_idle+0xc)
 - 0xffff8000102fdc5c (arch_cpu_idle+0xc)
 - 0xffff800010305420 (default_idle_call+0x20)
 - 0xffff80001005fcf0 (do_idle+0x210)
 - 0xffff80001005fec4 (cpu_startup_entry+0x24)
 - 0xffff8000102fe278 (rest_init+0xd0)
 - 0xffff8000103d0c7c (arch_call_rest_init+0xc)
 - 0xffff8000103d11f0 (start_kernel+0x528)
```

* プログラムカウンタより前のアドレスをディスアセンブル

```
>>> disassemble_at(p.hv_translate(hv.ctx.elr, True) - 32, 64)
    81f6fdc04:  d53cd042        mrs     x2, tpidr_el2
    81f6fdc08:  d53cd041        mrs     x1, tpidr_el2
    81f6fdc0c:  d53cd041        mrs     x1, tpidr_el2
    81f6fdc10:  d53cd040        mrs     x0, tpidr_el2
    81f6fdc14:  d53cd040        mrs     x0, tpidr_el2
    81f6fdc18:  d503233f        paciasp
    81f6fdc1c:  d5033f9f        dsb     sy
    81f6fdc20:  d503207f        wfi
    81f6fdc24:  d50323bf        autiasp
    81f6fdc28:  d65f03c0        ret
    81f6fdc2c:  d503201f        nop
    81f6fdc30:  d503233f        paciasp
    81f6fdc34:  a9bf7bfd        stp     x29, x30, [sp, #-16]!
    81f6fdc38:  910003fd        mov     x29, sp
    81f6fdc3c:  97fffff7        bl      81f6fdc18 <_start+0x14>
    81f6fdc40:  a8c17bfd        ldp     x29, x30, [sp], #16
```

* ハイパーバイザーへの最後の例外からレジスタ %0 をダンプアウト

```
>>> hv.ctx.regs[0]
0xffff0003ccaf21e0
```

### Linux コンソール出力
* ハイパーバイザー下では、m1n1 HV で生成された 2 番目の ttyACM1 デバイスを介して Linux のコンソール出力を取得可能
* **注意** 出力にはたくさんの先頭の空白が含まれるので、```sed '/^ */s///' ```で除去して次のようにきれいに表示:

<details>
<summary>コンソール出力を見る</summary>

```
% picocom /dev/ttyACM1
picocom v3.1
port is        : /dev/ttyACM1
flowcontrol    : none
baudrate is    : 9600
parity is      : none
databits are   : 8
stopbits are   : 1
escape is      : C-a
local echo is  : no
noinit is      : no
noreset is     : no
hangup is      : no
nolock is      : no
send_cmd is    : sz -vv
receive_cmd is : rz -vv -E
imap is        : 
omap is        : 
emap is        : crcrlf,delbs,
logfile is     : none
initstring     : none
exit_after is  : not set
exit is        : no
Type [C-a] [C-h] to see available commands
Terminal ready
m1n1
sc: Initializing
CPU init... CPU: M1 Icestorm
boot_args at 0x81bd9c000
revision:     2
version:      2
virt_base:    0xfffffe0011b34000
phys_base:    0x819b34000
mem_size:     0x3b30b0000
top_of_kdata: 0x81bda0000
video:
base:       0xbd2ce4000
display:    0x0
stride:     0x2800
width:      2560
height:     1600
depth:      0x1001e
machine_type: 0
devtree:      0xfffffe0012b34000
devtree_size: 0x58000
cmdline:      -v
boot_flags:   0x0
mem_size_act: 0x400000000
m1n1 vda44067
Copyright (C) 2021 The Asahi Linux Contributors
Licensed under the MIT license
Running in EL1
Heap base: 0x81ece4000
MMU: Initializing...
MMU: SCTLR_EL1: 100030d50980 -> 30901085
MMU: running with MMU and caches enabled!
fb init: 2560x1600 (30) [s=2560] @0xbd2ce4000
fb console: max rows 46, max cols 64
fb: display logo
Device info:
Model: MacBookAir10,1
Target: J313
Board-ID: 0x26
Chip-ID: 0x8103
WDT registers @ 0x23d2b0000
WDT disabled
pmgr: initialized, 272 devices found.
Initialization complete.
Checking for payloads...
Found a gzip compressed payload at 0x81ace4000
Uncompressing... 2029114 bytes uncompressed to 5693448 bytes
Found a kernel at 0x81ee00000
Found a devicetree at 0x81aed363a
Found a gzip compressed payload at 0x81aed446f
Uncompressing... 7819222 bytes uncompressed to 16741888 bytes
Found a cpio initramfs at 0x81f400000
No more payloads at 0x81b649445
Starting secondary CPUs...
FDT: initrd at 0x81f400000 size 0xff7600
FDT: framebuffer@bd2ce4000 base 0xbd2ce4000 size 0xfa0000
ADT: 64 bytes of random seed available
FDT: KASLR seed initialized
FDT: Passing 64 bytes of random seed
FDT: DRAM at 0x800000000 size 0x400000000
FDT: Usable memory is 0x819b34000..0xbccbe4000 (0x3b30b0000)
FDT: CPU 1 is not alive, disabling...
FDT: CPU 2 is not alive, disabling...
FDT: CPU 3 is not alive, disabling...
FDT: CPU 4 is not alive, disabling...
FDT: CPU 5 is not alive, disabling...
FDT: CPU 6 is not alive, disabling...
FDT: CPU 7 is not alive, disabling...
FDT prepared at 0x8203f8000
tps6598x: Error getting /arm-io/i2c0/hpmBusManager/hpm0 node
usb: tps6598x_init failed for /arm-io/i2c0/hpmBusManager/hpm0.
usb: failed to init hpm0
pmgr: Error getting node /arm-io/atc-phy0
usb: unable to bringup the phy with index 0
tunable: unable to find ADT node /arm-io/apcie/pci-bridge1.
pcie: Error applying apcie-config-tunables for /arm-io/apcie/pci-bridge1
Preparing to boot kernel at 0x81ee00000 with fdt at 0x8203f8000
Valid payload found
Preparing to run next stage at 0x81ee00000...
MMU: shutting down...
MMU: shutdown successful, clearing caches
Vectoring to next stage...
M1 Linux
Starting Shell
Still running 1
Still running 2
Still running 3
Still running 4
Still running 5
Still running 6
Still running 7
Still running 8
Still running 9
Still running 10
Still running 11
Still running 12
Still running 13
Still running 14
Still running 15
Still running 16
Still running 17
Still running 18
Still running 19
Still running 20
Still running 21
....
```

</details>

# rootファイルシステムオプション
* [initrd + USBキーボード](linux-bringup-usb-keyboard.md#linux-usb-keyboard)
* [USBドライブブート](linux-bringup-usb.md)
* [USBドライブをNVMEパーティションに](linux-bringup-nvme.md)

# その他の機能
* [WiFi対応](linux-bringup-wifi.md)
* [X11対応](linux-bringup-x11.md)

# 欠けているもの
* サウンド
* 電源管理 - 蓋を『閉めないで』
