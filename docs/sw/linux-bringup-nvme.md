---
title: Linux Bringup: NVME
---

2025/6/30時点の[linux-bringup-nvme](https://github.com/AsahiLinux/docs/blob/main/docs/sw/linux-bringup-nvme.md)の翻訳

訳注: debian文書へのリンクは日本語文書へのリンクに変更

---
# USBドライブブート

## 簡単な方法
* [Glanzmann氏のノート](https://tg.st/u/asahi.txt) に従い、MacOS 上で debian bullseye rootfs を取得し、
新規作成した nvme パーティションに直接 dd してください。

## より難しい方法
* これは別のLinuxマシンで実行 - debian bullseyeを使用

### rootfsの構築
* [自分でビルド](https://www.debian.org/releases/stretch/arm64/apds03.html.ja)

```
mkdir debinst
sudo debootstrap --arch arm64 --foreign bullseye debinst http://ftp.au.debian.org/debian/
sudo cp /usr/bin/qemu-aarch64-static debinst/usr/bin
```
    * chroot でログインし bash プロンプトを表示：`sudo LANG=C.UTF-8 chroot debinst qemu-aarch64-static /bin/bash`
    * 次に第2ステージを完了 `/debootstrap/debootstrap --second-stage`
    * その間に望む他のパッケージもインストール： `apt install file screenfetch procps`
    * ssh用に ssh サーバをインストール `apt install openssh-server`
    * sshでrootログインを許可 `vi /etc/ssh/sshd_config` で `PermitRootLogin yes` を設定
    * 最も重要なのはrootのパスワードを設定すること `passwd` 

### rootfsをUSBドライブにインストール
    * USBドライブを接続し、fdiskでパーティションを作成（ドライブは/dev/sdbと仮定） `sudo fdisk /dev/sdb`
    * パーティションをフォーマット（最初のUSBドライブと仮定） `sudo mkfs.ext4 /dev/sdb1`
    * ドライブを /mnt/img 等の場所にマウント `sudo mount /dev/sdb1 /mnt/img`
    * 上記で作成した rootfs をドライブにインストール `sudo cp -a debinst/. /mnt/img`
    * ドライブをアンマウント `sudo umount /mnt/img`

### USBドライブとrootとして起動
    * [USBケーブルでのブート](linux-bringup.md#usb%E3%82%B1%E3%83%BC%E3%83%96%E3%83%AB%E3%81%A7linux%E3%82%92%E5%AE%9F%E8%A1%8C)に戻る
    * 最新の m1n1.macho がロードされていることを確認 `python3 proxyclient/tools/chainload.py build/m1n1.macho`
    * ビルトイン機能でカーネルを構築（.configで=mをチェックし、=yに変更する）
    * 特にCONFIG_EXT4_FS=yが必要
    * [Asahi linux snapshot](https://github.com/amworsley/AsahiLinux/tree/asahi-kbd) と [config](https://raw.githubusercontent.com/amworsley/asahi-wiki/main/images/config-keyboard+nvme) を試用
    * 次に、USBドライブでgzip圧縮イメージを起動。MBA（MacBook Air）の2番目のUSB Type-Cポートに、Type-C to USB-Type A HUBを介して
ドライブを差し込む必要あり
    * 低速のUSBデバイス（キーボード）を差し込むとHUBがリセットされUSBドライブが壊れてしまうので注意が必要。なのでキーボードは使用しない。
Type-AからType-Cへのドングルは問題なく動作
    * USBケーブル経由で新しいカーネルをロードし、USBドライブをルートファイルシステムとして起動
```
python3 proxyclient/tools/linux.py -b 'earlycon console=tty0  console=tty0 debug net.ifnames=0 rw root=/dev/sda1 rootdelay=5 rootfstype=ext4'  Image.gz t8103-j313.dtb
```
    * rootファイルシステムはドライブの最初のパーティション (/dev/sda1) にあり、それはMBA (t8103-j313.dtb) 
    * もし違うものが起動するなら、**arch/arm64/boot/dts/apple/** の .dts ファイルで **model** フィールドの値を確認

### nvmeにrootfsをインストール
 * MacOS上で[Glanzmann氏のノート](https://tg.st/u/asahi.txt)に従って空き領域を確保する必要あり
 * これはあくまで例で実際の数値は異なる場合があるので十分注意
 * 空き領域を作成 - 最後の数字はmacosが占有する領域 `diskutil apfs resizeContainer disk0s2 200GB`
 * パーティションをリストアップし空き領域がどこにあるか確認 `diskutil list`
 * NVME に Linux rootfs 用の FAT32 パーティションを空き領域から割り当て
 * **注意** パーティションは空き領域の**前**に指定する必要あり `diskutil addPartition disk0s3 FAT32 LB 42.6GB`
 * ext4 USBドライブを(上記のように)rootとして起動
 * fdisk を使ってどのパーティションが新しい FAT32 パーティションであるかを確認（上で作成したサイズでないといけない） `fdisk -l /dev/nvme0n1`
 * 確認できたら ext4 にフォーマット `mkfs.ext4 /dev/nvme0n1p4`
 * マウント `mount /dev/nvme0n1p4 /mnt`
 * USBドライブのrootfsをそこにコピー `/mnt`
 * -xがnvmeの新規ファイルシステムへの再帰的な降下を防ぐはず
 * アンマウント `umount /mnt`.
 * 次にUSBケーブル経由でnvme上の新規rootファイルシステムでの起動が行えるか試す

```
python3 proxyclient/tools/linux.py -b 'earlycon console=tty0  console=tty0 debug net.ifnames=0 rw root=/dev/nvme0n1p6 rootfstype=ext4' Image.gz t8103-j313.dtb
```
