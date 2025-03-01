2025/3/1時点の[SW:Ubuntu Asahi Qemu](https://github.com/AsahiLinux/docs/wiki/SW%3AUbuntu-Asahi-Gambas)の翻訳

訳注:wikiへのURLでのリンクは日本語wikiへのリンクに置き換え

---
M1 Air上のUbuntu 22.10 Asahiで検証しています。

他のUbuntu Asahiインストール情報は [SW:代替ディストリビューション](SW-Alternative-Distros.md) 参照

virt-managerもインストールしています。

ユーザネットワーキング用にコンパイル時にslirpを含めています:
https://bugs.launchpad.net/qemu/+bug/1917161

```
sudo apt -y install git libglib2.0-dev libfdt-dev \
libpixman-1-dev zlib1g-dev ninja-build \
git-email libaio-dev libbluetooth-dev \
libcapstone-dev libbrlapi-dev libbz2-dev \
libcap-ng-dev libcurl4-gnutls-dev libgtk-3-dev \
libibverbs-dev libjpeg8-dev libncurses5-dev \
libnuma-dev librbd-dev librdmacm-dev \
libsasl2-dev libsdl2-dev libseccomp-dev \
libsnappy-dev libssh-dev \
libvde-dev libvdeplug-dev libvte-2.91-dev \
libxen-dev liblzo2-dev valgrind xfslibs-dev \
libnfs-dev libiscsi-dev flex bison meson \
qemu-utils virt-manager

git clone https://gitlab.com/qemu-project/qemu.git

cd qemu

git submodule init

git submodule update

git clone https://gitlab.freedesktop.org/slirp/libslirp

cd libslirp

meson build

ninja -C build install

cd ..

mkdir build

cd build

../configure --enable-slirp

make -j$(nproc)

sudo make install
```

## 様々なOS用のネットワーク例

https://wiki.qemu.org/Documentation/Networking

##　ネットワーク対応 ReactOS x86 32bit

https://reactos.org

```
wget reactos-32bit-bootcd-nightly.7z

7z x reactos-32bit-bootcd-nightly.7z

mv reactos*.iso ReactOS.iso
```

20Gが増加可能な最大ディスクサイズです:

```
qemu-img create -f qcow2 ReactOS.qcow2 20G
```

ここの`-m 3G`は3GB RAMを意味します。

`start.sh`をこの内容に編集し`chmod +x start.sh`で実行可能してください:

```
qemu-system-i386 -m 3G -drive if=ide,index=0,media=disk,file=ReactOS.qcow2 \
-drive if=ide,index=2,media=cdrom,file=ReactOS.iso -boot order=d \
-serial stdio -netdev user,id=n0 -device rtl8139,netdev=n0
```

それから`./start.sh`を実行します。

## ネットワーク対応 WinXP x86 32bit

1) 80GBの拡張可能なハードディスクイメージを作成:
```
qemu-img create -f qcow2 winxp.qcow2 80G
```
2) winxp.isoからインストール開始

`start.sh`を編集し`chmod +x start.sh`で実行可能にします。ここの `-m 4G` は 4 GB RAMの意味:
```
qemu-system-i386 -m 4G -drive if=ide,index=0,media=disk,file=winxp.qcow2 \
-drive if=ide,index=2,media=cdrom,file=winxp.iso -boot order=d \
-serial stdio -netdev user,id=n0 -device rtl8139,netdev=n0
```
3) インストール後、isoを外してブート
```
qemu-system-i386 -m 4G -drive if=ide,index=0,media=disk,file=winxp.qcow2 \
-boot order=c \
-serial stdio -netdev user,id=n0 -device rtl8139,netdev=n0
```

## ネットワーク対応　Win10 x86_64

1) 80GBの拡張可能なハードディスクイメージを作成:
```
qemu-img create -f qcow2 win10.qcow2 80G
```
2) win10.isoからインストール開始

`start.sh`を編集し`chmod +x start.sh`で実行可能にします。ここの `-m 4G` は 4 GB RAMの意味:
```
qemu-system-x86_64 -m 4G -drive if=ide,index=0,media=disk,file=win10.qcow2 \
-drive if=ide,index=2,media=cdrom,file=win10.iso -boot order=d \
-serial stdio -netdev user,id=n0 -device rtl8139,netdev=n0
```
3) インストール後、isoを外してブート
```
qemu-system-x86_64 -m 4G -drive if=ide,index=0,media=disk,file=win10.qcow2 \
-boot order=c \
-serial stdio -netdev user,id=n0 -device rtl8139,netdev=n0
```

## ネットワーク対応 Win11 x86_64

1) 80GBの拡張可能なハードディスクイメージを作成:
```
qemu-img create -f qcow2 win11.qcow2 80G
```
2) win11.isoからインストール開始

`start.sh`を編集し`chmod +x start.sh`で実行可能にします。ここの `-m 4G` は 4 GB RAMの意味、`-usbdevice tablet`はマウスをより正確に使えるようにする:
```
qemu-system-x86_64 -hda win11.qcow2 -cdrom win11.iso -boot d \
-smp 2 -m 4G -usbdevice tablet \
-serial stdio -netdev user,id=n0 -device rtl8139,netdev=n0
```

3) Win11インストール時

a) バイパスレジストリキーを使用(訳注:インストール時のTPMチェックの無効化):

https://blogs.oracle.com/virtualization/post/install-microsoft-windows-11-on-virtualbox

b) UEFI と TPMの検討とそれを有効にする方法、未検証:

- <https://getlabsdone.com/how-to-install-windows-11-on-kvm/>
- <https://getlabsdone.com/how-to-enable-tpm-and-secure-boot-on-kvm/>
- <https://github.com/stefanberger/swtpm/wiki>
- <https://www.reddit.com/r/AsahiLinux/comments/y7hplo/virtual_machines_on_asahi_linux/>
- <https://that.guru/blog/uefi-secure-boot-in-libvirt/>
- <https://www.reddit.com/r/AsahiLinux/comments/107m4nb/windows_vm_on_asahi_qemukvm_virtmanager/>

```
sudo apt-get install dh-autoreconf libssl-dev \
     libtasn1-6-dev pkg-config libtpms-dev \
     net-tools iproute2 libjson-glib-dev \
     libgnutls28-dev expect gawk socat \
     libseccomp-dev make -y
git clone https://github.com/stefanberger/swtpm
cd swtpm
./autogen.sh --with-openssl --prefix=/usr
make -j4
make -j4 check
sudo make install
```

4) インストール後、isoを外してブート
```
qemu-system-x86_64 -hda win11.qcow2 -boot c \
-smp 2 -m 8G -usbdevice tablet \
-serial stdio -netdev user,id=n0 -device rtl8139,netdev=n0
```

## ネットワーク対応 Ubuntu x86_64

1) 80GBの拡張可能なハードディスクイメージを作成::
```
qemu-img create -f qcow2 ubuntu.qcow2 80G
```
2) ubuntu.isoからインストール開始

`start.sh`を編集し`chmod +x start.sh`で実行可能にします。ここの `-m 4G` は 4 GB RAMの意味:
```
qemu-system-x86_64 -m 4G -drive if=ide,index=0,media=disk,file=ubuntu.qcow2 \
-drive if=ide,index=2,media=cdrom,file=ubuntu.iso -boot order=d \
-serial stdio -netdev user,id=n0 -device rtl8139,netdev=n0
```
3) インストール後、isoを外してブート
```
qemu-system-x86_64 -m 4G -drive if=ide,index=0,media=disk,file=ubuntu.qcow2 \
-boot order=c \
-serial stdio -netdev user,id=n0 -device rtl8139,netdev=n0
```
