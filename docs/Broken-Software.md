2025/2/21時点の[Broken Software](https://github.com/AsahiLinux/docs/blob/main/docs/Broken-Software.md)の翻訳

---
このページでは、Apple Silicon 機器では正しく動作しないことが知られているソフトウェアをリストアップしています。
これを公開することで、コミュニティのメンバーが影響を受けるパッケージの修正を上流に提供する動機となり、AArch64ソフトウェアの
エコシステムがより良くなることを期待しています。

### ${PACKAGE} が AArch64 に対応しているのに、なぜ動かないのでしょうか?
これはほとんどの場合、16K ページへの対応が正しくないか、不完全であることが原因です。
パッケージが4Kのページサイズを想定して作られていたり、大きなページと互換性がない場合があります。
デスクトップ Linux のソフトウェアではこれは深刻な問題ではありませんでした。
x86/amd64 では 4K ページしか対応しておらず、PowerPC では 4K _または_ 64K ページしか対応していないからです。
AArch64 は、AArch64 機器が 4K、 16K _ または_ 64K ページを使うことができるという点で独自です。

### なぜ4Kページを使わないのですか？
これらの機器は 4K カーネルをブートできますが、そうするためには非常に面倒なパッチが必要です。
IOMMU は 16K-aligned ページ しか対応していないからです。
これは性能上の重大な問題を引き起こすだけでなく、AArch64に不完全な対応しているユーザースペースの
ソフトウェアという実際の問題にも対処していません。
XNU(macOS)はユーザー空間で独立したページサイズに対応することでこの問題を回避していますが、
Linuxにはそのようなメカニズムはありませんし、おそらく今後もないでしょう。

### なぜ固定バージョンの ${PACKAGE} を自分でホストしないのですか?
デスクトップクラスの AArch64 マシンは今後数年間でより一般的になることが予想されます。上流優先のポリシーを持つことで、
これらの修正がディストリビューションのリポジトリを通じてすべての人に伝搬され、AArch64 のエコシステムをすべての人の
ために改善することが確認できます！ [修正済みパッケージ](#修正済みパッケージ) をご覧ください。この結果としてすべての
人のために修正されたソフトウェアのリストがあります。Emacsを私たちだけのものにしたくないでしょう?

### なぜ『動作しない』ことが『即座のsegmentation fault』を意味するのですか？
ELFの実行ファイルやライブラリが16Kページにアライメントされていないセクションを持つ場合、ローダーはバイナリをメモリに
マップすることができず、この失敗をsignalで知らせ、segmentation faultを発生させます。

このことは `readelf -l /path/to/binary` を使って確認することができます。タイプ `LOAD` のすべてのプログラム
ヘッダーセクションは、 Apple Siliconのような16Kのマシンで正常に読み込むためには`ALIGN` 値が少なくとも `0x4000` で
ある必要があります。ここで説明されているライブラリは 4K ページ (`0x1000`) にしかアラインされていないので、ロードできません。

```
$ readelf -l lib64/ld-android.so

Elf file type is DYN (Shared object file)
Entry point 0x0
There are 9 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x00000000000001f8 0x00000000000001f8  R      0x8
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000874 0x0000000000000874  R      0x1000
  LOAD           0x0000000000001000 0x0000000000001000 0x0000000000001000
                 0x0000000000000004 0x0000000000000004  R E    0x1000
  LOAD           0x0000000000002000 0x0000000000002000 0x0000000000002000
                 0x00000000000000a0 0x00000000000000a0  RW     0x1000
  DYNAMIC        0x0000000000002000 0x0000000000002000 0x0000000000002000
                 0x00000000000000a0 0x00000000000000a0  RW     0x8
  GNU_RELRO      0x0000000000002000 0x0000000000002000 0x0000000000002000
                 0x00000000000000a0 0x0000000000001000  R      0x1
  GNU_EH_FRAME   0x000000000000082c 0x000000000000082c 0x000000000000082c
                 0x0000000000000014 0x0000000000000014  R      0x4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x0
  NOTE           0x0000000000000238 0x0000000000000238 0x0000000000000238
                 0x0000000000000020 0x0000000000000020  R      0x4

 Section to Segment mapping:
  Segment Sections...
   00     
   01     .note.gnu.build-id .dynsym .gnu.hash .dynstr .eh_frame_hdr .eh_frame 
   02     .text 
   03     .dynamic 
   04     .dynamic 
   05     .dynamic 
   06     .eh_frame_hdr 
   07     
   08     .note.gnu.build-id 
```

AArch64コンパイラのデフォルトは、すべてのAArch64機器との互換性のために、64KにアラインされたセクションでELFファイルを生成するが、
ツールバグ(例えば古いバージョンの `patchelf` によって操作されたバイナリ) や、カスタマイズされたコンパイラフラグ (例えば
古いバージョンの Chrome (および Electron) や最新の Android プログラムを含む多くの Google プログラム) によって、
セクションが4Kにしかアラインメントされていないバイナリが生成されることがあります。


## 入手可能なワークアラウンドはありますか？

Fedora Linux Asahi Remix の `muvm` パッケージは標準で 4K カーネルを仮想化するよう設定されています
(FEX を x86_64 binfmt ハンドラーとしているならば x86_64 プログラムも動作するでしょう)。
`muvm` 内でソフトウェアを実行してみれば成功の度合いが変わるでしょう。

## 問題があるパッケージ
| パッケージ | 上流への報告 | 注釈 |
| ------- | --------------- | ----- |
| Chromium | <https://issues.chromium.org/issues/378017037> | kGuardPageSizeのハードコートが原因で 16KiB ページの Linux で cppgc がクラッシュ |
| hardened_malloc | <https://github.com/GrapheneOS/hardened_malloc/issues/183> | 16kページ対応する前にhardened_mallocに多くの変更が必要。MTEが必要でkこの時点では高優先度ではない |
| jemalloc | <https://github.com/jemalloc/jemalloc/issues/467> | 上流は修正する気がない、4kページサイズシステムでコンパイルするならビルドオプションが必要。 ArchLinuxARM](https://github.com/archlinuxarm/PKGBUILDs/pull/1914)で対応 |
| jemalloc | <https://github.com/archlinuxarm/PKGBUILDs/pull/1914> | ページサイズ≧システムに対してコンパイルしたときだけ動作 |
| MEGAsync | <https://github.com/meganz/MEGAsync/pull/801> |
| notion-app(-enhancer) | <https://github.com/notion-enhancer/notion-repackaged/issues/107> | electron + 壊れたビルドフラグ |
| Wine | <https://bugs.winehq.org/show_bug.cgi?id=52715> |
| Zig | <https://github.com/ziglang/zig/issues/11308> | 

\* x86-64 ソフトウェアの実行は FEX を実行している 4k ページサイズの microVM 経由で対応します。

## 修正済みパッケージ
| パッケージ | 修正コミット | 注釈 |
| ------- | ------------- | ----- |
| 1Password | _プロプライエタリ_ | 8.8.0-119 betaで修正 |
| Android Cuttlefish | <https://android-review.googlesource.com/c/device/google/cuttlefish/+/2545951> | muslに切り替えたことによりAOSPメインブランチで修正 |
| box64 | <https://github.com/ptitSeb/box64/issues/384> | 0.2.8以降修正 |
| btrfs | <https://lore.kernel.org/lkml/cover.1653327652.git.dsterba@suse.com/> | Linux 5.19以降で修正[注意事項](https://social.treehouse.systems/@marcan/111493984306764821)) |
| Chromium | <https://bugs.chromium.org/p/chromium/issues/detail?id=1301788> | Electronアプリを含む<br>102以降で修正 |
| Emacs | <https://lists.gnu.org/archive/html/bug-gnu-emacs/2021-03/msg01260.html> | 28.0以降で修正 |
| f2fs | <https://github.com/torvalds/linux/commit/5c9b469295fb> | https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=d7e9a9037de27b642d5a3edef7c69e2a2b460287 で修正 ||
| fd | <https://github.com/sharkdp/fd/issues/1085> | 10.1以降で修正 |
| k3s-io | <https://github.com/k3s-io/k3s/issues/7335> | 1.27.2以降で修正 |
| KiCad | <https://gitlab.com/kicad/code/kicad/-/issues/16008> | 7.0.10以降で修正 |
| libglvnd | <https://gitlab.freedesktop.org/glvnd/libglvnd/-/merge_requests/262> | 1.5.0以降で修正 |
| libunwind | <https://github.com/libunwind/libunwind/pull/330> | 1.7.0以降で修正 |
| libunwind | <https://github.com/libunwind/libunwind/issues/260> | 1.8.0以降で修正 |
| libvirt/QEMU/KVM | <https://patchew.org/QEMU/20230727073134.134102-1-akihiko.odaki@daynix.com/> | QEMU 7.2.6 / 8.0.5 / 8.1.1以降で修正 |
| lvm2 | <https://bugzilla.redhat.com/show_bug.cgi?id=2059734> | 2.03.21以降で修正 |
|pdfium| <https://bugs.chromium.org/p/pdfium/issues/detail?id=1853> |chromium 108で修正|
|qt5-webengine| <https://bugreports.qt.io/browse/QTBUG-105145> |chromium 87、上流では修正されていないはず、 [下流のArchLinuxARMで修正](https://github.com/archlinuxarm/PKGBUILDs/pull/1928) |
|qt6-webengine| <https://bugreports.qt.io/browse/QTBUG-105145> |6.3ではchromium 94、 6.5で上流では部分的に修正されるがQtpdfは違う、[下流のArchLinuxARMで修正](https://github.com/archlinuxarm/PKGBUILDs/pull/1928) |
| Redis | <https://bugzilla.redhat.com/show_bug.cgi?id=2240293> <https://bodhi.fedoraproject.org/updates/FEDORA-2023-bdb1515542> | redis-7.0.13-2.fc38 及びに redis-7.2.1-2.fc39以降のfedoraで修正 |
| rr | <https://github.com/rr-debugger/rr/pull/3146> | 5.6.0以降で修正 |
| Rust | <https://github.com/archlinuxarm/PKGBUILDs/commit/19a1393> | ALARM/extraの`rust-1.62.1-1.1`で修正 |  
| Telegram Desktop | <https://github.com/telegramdesktop/tdesktop/issues/26103> | 4.1.1以降で修正|
| Visual Studio Code | <https://aur.archlinux.org/packages/visual-studio-code-bin> | 1.71.0以降で修正(Electron 19を使用) |
| WebKitGTK | <https://github.com/WebKit/WebKit/commit/0a4a03da45f774> | 2.34.6以降で修正 |

## バグ
サードパーティソフトウェアの問題（ページサイズやアーキテクチャ上の問題を除く）で、Asahiのコアチームメンバーによって報告または追跡されているもの：

| パッケージ       | 問題 |注釈 |
| -------------- | ----- |----- |
| abrt           | [ABRTクラッシュレポートを提出できない: 処理が失敗(ABRT can't submit crash report: processing failed)](https://bugzilla.redhat.com/show_bug.cgi?id=2238248) | issueクローズ (訳注:fedora 38がEOL(保守終了)なので)|
| blender        | [blenderが未対応ハードウェアに関するちゃんとしたフィードバックを提供する代わりにコアダンプ(blender core dumps at execution instead of giving sane feedback about unsupported hardware)](https://bugzilla.redhat.com/show_bug.cgi?id=2237821) |問題解決|
| chromium       | [Skiaシェーダコンパイルエラー(Skia shader compilation error)](https://bugs.chromium.org/p/chromium/issues/detail?id=1442633) | Chromium 121.0.6167.85 で修正 |
| dracut       | [Memoize find_kmod_module_from_sysfs_node(find_kmod_module_from_sysfs_nodeをメモ)](https://github.com/dracut-ng/dracut-ng/pull/408) | dracut 103で修正 |
| firefox        | [wayland:起動時の最初のフレームが一瞬初期化されない場合あり(x11でも)(wayland: The first frame on startup is sometimes uninitialized for a moment (also maybe on x11))](https://bugzilla.mozilla.org/show_bug.cgi?id=1831051) |
| firefox        | [Linux aarch64ユーザエージェントでYouTubeの解像度が1080までになる(YouTube is capping resolutions to 1080 on Linux aarch64 user agents)](https://bugzilla.mozilla.org/show_bug.cgi?id=1869521) | Firefox 123で修正 |
| gcc            | [aarch64とx86_64でcephをLTOで正しくコンパイルしない(LTO miscompilation of ceph on aarch64)](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=113359) | gcc 13.3/14.1で修正 (訳注:[ceph](https://ja.wikipedia.org/wiki/Ceph)、[LTO(Link Time Optimization, リンク時最適化](https://qiita.com/kaityo256/items/a822fc462a4de6ddd8e7)) |
| glibc          | [TLS modidの再利用がTLSアクセスを破壊(TLS modid reuse breaks TLS accesses)](https://bugzilla.redhat.com/show_bug.cgi?id=2251557) |glibc 2.39で修正 |
| gnome-bluetooth/bluez | A2DP出力の音声が頻繁に途切れたり接続が切れたりする（bluemanでは問題なし） | まだバグは未記入 |
| gtk              | [GSK が Wayland の小数スケーリング時に load=dont-care と blend=over を使用した不正なレンダリング操作を行い、グラフィックの破損を引き起こす問題(GSK issues illegal render ops with load=dont-care and blend=over with Wayland fractional scaling, causing graphical corruption)](https://gitlab.gnome.org/GNOME/gtk/-/issues/7146) |
| gtk              | [GSK/vulkan がフォーマットをチェックせず`VkImageFormatProperties.maxMipLevels` というチルティングせずにミップマップを使用(GSK/vulkan uses mipmaps without checking the formats/tiling `VkImageFormatProperties.maxMipLevels`)](https://gitlab.gnome.org/GNOME/gtk/-/issues/7229) |
| hyprland         | [hyprLand 0.42.0 使用時に OpenGL アプリケーションがクラッシュする問題(issue with OpenGL applications crashing when using Hyprland 0.42.0)](https://github.com/hyprwm/Hyprland/issues/7364) | hyprland 0.43.0 で修正 |
| hyprland         | [Explicit Sync timelines fail importing, killing the client](https://github.com/hyprwm/Hyprland/issues/8158) |
| kpipewire      | [Spectacleが特定の大きさでh264のウィンドウを録画するのに失敗(Spectacle fails to record a window with h264 in specific dimensions)](https://bugs.kde.org/show_bug.cgi?id=475472) |問題解決|
| kpipewire      | [画面録画品質がひどい(Screen recording quality is terrible)](https://bugs.kde.org/show_bug.cgi?id=476186) |
| kpipewire      | [OpenH264コーディック対応(OpenH264 codec support)](https://bugs.kde.org/show_bug.cgi?id=476187) | Plasma 6.1.4 で修正|
| kwin           | [ハードウェアカーソル未対応時にマルチスクリーン上で出力がフリーズ(Outputs freeze on multi-screen when hardware cursors are not supported)](https://bugs.kde.org/show_bug.cgi?id=477451) | Plasma 6.0で修正 |
| kwin           | [マルチスクリーンでルートバックグラウンドダメージ領域が正しく計算されない(Root background damage regions are calculated incorrectly with multiscreen)](https://bugs.kde.org/show_bug.cgi?id=477454) |
| kwin           | [ソフトウェアカーソルの再描画に不具合あり(Software cursor repaints are glitchy with fractional scaling sometimes)](https://bugs.kde.org/show_bug.cgi?id=477455) | Plasma 6.0で修正 |
| lsp-common-lib   | [AArch64でのアトミック操作を修正(Fix atomic operations for AArch64)](https://github.com/lsp-plugins/lsp-plugins/issues/463) | lsp-common-lib 1.0.40 で修正 |
| lib-dsp-lib    | [aarch64 msmatricコードの修正(Fix aarch64 msmatrix code)](https://github.com/lsp-plugins/lsp-dsp-lib/pull/20) | lsp-dsp-lib 1.0.20 で修正 |
| plasmashell      | [startplasmaがprofile.dとenvironment.d間の変数統合を破壊(startplasma breaks variable merging between profile.d and environment.d)](https://bugs.kde.org/show_bug.cgi?id=491579) |
| qqc2-desktop-style | [小数倍率を使用すると QML ソフトウェア内のテキストグリフの一部が垂直方向にずれたりつぶれたりする(Some text glyphs in QML software are vertically mis-aligned or squished when using a fractional scale factor)](https://bugs.kde.org/show_bug.cgi?id=479891) | KDE Frameworks 6.9.0 で修正 |
| systemsettings | [デフォルトのシステムキーボードモデルがWayland上で正しく設定されていない(default system keyboard model is not correctly set on Wayland)](https://bugs.kde.org/show_bug.cgi?id=475435) |
| wireplumber    | [luaからPWモジュールへ引数を渡せない(Cannot pass args to PW modules from lua)](https://gitlab.freedesktop.org/pipewire/wireplumber/-/issues/538)|
| wireplumber      | [Wireplumber ignores default playback volume(wireplumberがデフォルトプレイバックボリュームを無視](https://gitlab.freedesktop.org/pipewire/wireplumber/-/issues/655#) | wireplumber 0.5.3 で修正|
| wlroots          | [DRM 色変換行列対応を追加（redshift などで有効）(Add support for DRM Color Transformation Matrix (Useful for e.g. redshift))](https://gitlab.freedesktop.org/wlroots/wlroots/-/issues/1078) | [関連 PR](https://gitlab.freedesktop.org/wlroots/wlroots/-/merge_requests/4815) |
| wlroots          | [seatd使用時にマスターとしてレンダーを開けない(cannot open render as master when using seatd)](https://gitlab.freedesktop.org/wlroots/wlroots/-/issues/3911) |
| xkeyboard-config | [xkyboardでのMac Fnキーの扱い(Handling Mac Fn keys in xkeyboard)](https://gitlab.freedesktop.org/xkeyboard-config/xkeyboard-config/-/issues/379) |
