---
title: Linux Bringup: X11
---

2025/3/9時点の[linux-bringup-x11](https://github.com/AsahiLinux/docs/blob/main/docs/sw/linux-bringup-x11.md)の翻訳

訳注:
- 英語版debianのサイトへのリンクは対応する日本語版debianのサイトへのリンクに変更

---
# X11の実行
 * カーネルを次のオプションでビルドするとアクセラレーションなしのX11が実行可能 `CONFIG_DRM_FBDEV_EMULATION=y`
 * そのカーネルを起動した後、関連する xserver fbdev をインストール (debian の場合) `sudo apt install xserver-xorg-video-fbdev`
 * その後Xを起動することが可能 `startx`
 * もし問題があるならXサーバのログからエラーメッセージを検索 `less /var/log/Xorg.0.log`
 * X が起動したとすると、ウィンドウマネージャを設定する必要あり。ディストリビューションごとに設定。このdebianの場合は
[GUIシステム](https://www.debian.org/doc/manuals/debian-reference/ch07.ja.html) 参照
 * 私は非常に軽量なキーボードベースのfluxboxをインストール `apt install fluxbox`
 * シンプルなグラフィカルなログイン画面を使う場合 `apt install xdm`
 * Xターミナル **konsole** をインストール `apt install konsole`
 * ウェブブラウザはChromeが特別なカーネルページング対応を必要とするため（現時点では利用不可）、Firefoxをインストール `sudo apt install firefox-esr`

(![Macbook Air 2020で動くX11](https://github.com/AsahiLinux/docs/blob/main/docs/assets/mba-xorg-fbdev.png))
