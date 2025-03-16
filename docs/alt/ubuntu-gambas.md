---
title: Ubunt Gambas
---

2025/3/9時点の[ubuntu-gambas](https://github.com/AsahiLinux/docs/blob/main/docs/alt/ubuntu-gambas.md)の翻訳

---
**このページにはディストリビューション固有の情報が含まれています。**
**この文書レポジトリはディストロ固有のドキュメンテーションのための**
**ものではありません。これはディストリビューション独自の文書システムで**
**行うのが適切です。このページは削除される予定です。**

M1 Air上のUbuntu 22.10 Asahiで検証しています。

https://gambas.sourceforge.net

パッケージをインストールする間はrootで実行。
```
sudo su

apt-get update && apt-get install -y build-essential \
g++ automake autoconf libbz2-dev libzstd-dev \
default-libmysqlclient-dev unixodbc-dev libpq-dev \
libsqlite0-dev libsqlite3-dev libglib2.0-dev \
libgtk2.0-dev libcurl4-gnutls-dev libgtkglext1-dev \
libpcre3-dev libsdl-sound1.2-dev libsdl-mixer1.2-dev \
libsdl-image1.2-dev libxml2-dev libxslt1-dev \
librsvg2-dev libpoppler-dev libpoppler-glib-dev \
libpoppler-private-dev libpoppler-cpp-dev libasound2-dev \
libdirectfb-dev libxtst-dev libffi-dev libqt4-dev \
libqtwebkit-dev libqt4-opengl-dev libglew-dev \
libimlib2-dev libv4l-dev libsdl-ttf2.0-dev \
libgdk-pixbuf2.0-dev linux-libc-dev libgstreamer1.0-dev \
libgstreamer-plugins-base1.0-dev libcairo2-dev \
libgsl-dev libncurses5-dev libgmime-2.6-dev \
libalure-dev libgmp-dev libgtk-3-dev libsdl2-dev \
libsdl2-mixer-dev libsdl2-ttf-dev libsdl2-image-dev \
sane-utils libdumb1-dev libqt5opengl5-dev \
libqt5svg5-dev libqt5webkit5-dev libqt5x11extras5-dev \
qtbase5-dev qtwebengine5-dev libwebkit2gtk-4.0-dev \
git libssl-dev

exit
```

一般ユーザで実行。
```
./reconf-all

./configure -C --disable-keyring

make -j$(nproc)

sudo make install
```
