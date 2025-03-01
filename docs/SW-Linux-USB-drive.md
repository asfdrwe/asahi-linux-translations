2025/3/1時点の[SW-Linux-USB-drive](https://github.com/AsahiLinux/docs/blob/main/docs/SW-Linux-USB-drive.md)の翻訳

訳注: debian rootfsへのリンクは日本語ページヘ

---
## USB ドライブをrootに
* [Alyssa氏のtomshardwareの記事](https://www.tomshardware.com/news/apple-m1-debian-linux)から発想
* [debian rootfs](https://www.debian.org/releases/stretch/arm64/apds03.html.ja) に従って **arm64** のルートファイルシステムを作成
* 注意：以下はdebianシステムでrootとして実行

```
mkdir debinst-buster
debootstrap --arch arm64 --foreign buster debinst-buster http://ftp.au.debian.org/debian/
cp /usr/bin/qemu-aarch64-static debinst-buster/usr/bin
LANG=C.UTF-8 chroot debinst-buster qemu-aarch64-static /bin/bash
```

* chroot内でセットアップを完了

```
/debootstrap/debootstrap --second-stage
```

* その他必要なパッケージは apt でインストール

```
apt install file screenfetch procps openssh-server
```

* sshdをroot権限で使用できるように設定し、パスワードを設定

```
# Add PermitRootLogin yes
vi /etc/ssh/sshd_config
# Set root's password
passwd
```

* USB デバイスに ext4 パーティションを作成（例：/dev/sdb）

```
fdisk /dev/sdb
# n => new partition
# w => write it out
```

* ext4ファイルシステムをフォーマット ``mkfs.ext4 /dev/sdb1``
* 新しい rootfs をインストール (パーティションに rootfs を作成するだけでもよかったのですが・・・ :-)

```
mount /dev/sdb1 /mnt/img
cp -a debinst-buster/. /mnt/img
umount /mnt/img
```

* USBドライブを接続した状態でLinuxを起動（現在は/dev/sda1になっているはず）。2ポート目にシンプルなUSBハブを使い、
USBキーボードとUSBドライブを差し込んでからMacの電源を入れた

```
python3.9 proxyclient/tools/linux.py -b 'earlycon console=tty0  console=tty0 debug net.ifnames=0 rw root=/dev/sda1 rootdelay=5 rootfstype=ext4'  Image-dart-dev.gz t8103-j274-dart-dev.dtb
```

* **Image-dart-dev.gz** - 上記と同様の Asahi linux の dart/dev ブランチ
* **t8103-j274-dart-dev.dtb** - 上記Linuxのdtb
* initrdは渡されない
