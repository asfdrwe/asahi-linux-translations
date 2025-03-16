---
title: Linux Bringup: USB キーボード
---

2025/3/9時点の[linux-bringup-usb-keyboard](https://github.com/AsahiLinux/docs/blob/main/docs/sw/linux-bringup-usb-keyboard.md)の翻訳

---
# Linux USBキーボード
* USBキーボードをramdisk rootファイルシステムで動作させることが可能
* USB3 XHCD, DWC3, DART などで [Asahi dart/dev kernel](https://github.com/AsahiLinux/linux/tree/dart/dev)(訳注:リンク先がないです)を起動することにより実現
 * この [M1 MacBook Air with USB 用 Linux config file](https://github.com/amworsley/asahi-wiki/blob/main/images/config-jannau-iso9660-noR.gz) をベースにすることが可能 (ガジェットモードは有効にならない)
* initrd を変更して /bin/sh を実行するのみに (/init を編集) 
* ``python3.9 proxyclient/tools/linux.py -b 'earlycon console=tty0 console=tty0 debug' Image-dwc3.gz t8103-j274.dtb initrd-be2.gz`` で
直接起動させた場合
  * Image-dwc3.gz は Asahi dart/dev カーネル、そのカーネルでビルドした t8103.j274.dtb は **linux/arch/arm64/boot/dts/apple/t8103-j274.dtb** 、 initrd-be2.gz はセットアップ後に **/bin/sh** だけ実行するよう修正した debian Bullseye initrd 
* それから、Type-C to Type-A アダプタを使って、普通の古い USB Dell キーボードを接続し、実行中の /bin/sh にコマンドを入力
![M1 macbook で外付け USB キーボードから入力した Linux が動作している様子](https://github.com/AsahiLinux/docs/blob/main/docs/assets/linuxOnM1.png)

 * さらに一歩進んで、[USBドライブブート](SW-Linux-USB-drive.md)を試すことが可能
