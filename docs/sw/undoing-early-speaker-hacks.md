---
title: 初期のスピーカーハックの復元
---

2025/3/9時点の[undoing-early-speaker-hacks](https://github.com/AsahiLinux/docs/blob/main/docs/sw/undoing-early-speaker-hacks.md)の翻訳

---
### はじめに
ここに来たのは、おそらく、早い段階でスピーカーを有効にしようとし、そのとき設定を変更し、スピーカー対応の一般リリースで何かが壊れたからでしょう。
警告を受けています。

以下は、一般的なアーリーアダプターのハックに対する修正です。バグを報告する前に、これら*すべて*を試してください。
以下の問題のいずれかを修正できていないことが判明した場合、バグは無視されます。以下の問題を修正してください。

### 内蔵スピーカー/ヘッドフォンのPro Audioプロファイルが有効になっている/有効になっていた
この現象は、以下の3つの方法のいずれかで起こります：
* ヘッドホンを接続したままプロファイルを変更
* 非常に、非常に、*非常に*古いバージョンの`asahi-audio`を使用
* Wireplumberのノードパーミッションを回避してデバイスプロファイルの実験を実施

最初のケースでは、プロファイルを`Default`(HiFi)に戻してください。KDEのオーディオ設定で
プロファイルを`Default`に戻してください。ヘッドホンが接続されていない場合は
`Show Inactive Devices`を押します。`pavucontrol`のようなアプリケーションでも同じことができるはずです。
疑わしい場合は、WirePlumberのsstateディレクトリ（`rm -rf ~/.local/state/wireplumber/`）を削除し、再起動してください。

他の2つのケースの修正方法は同じです：
1. `rm -rf ~/.local/state/wireplumber/` を実行
2. `asahi-audio`とPipewire*と*Wireplumberを再インストール
3. マシンを再起動

### /etc/ に asahi-audio のプレリリース版のファイルがあります
`asahi-audio`の非常に古いバージョンは `/etc/pipewire/` と `/etc/wireplumber/` に設定を保存していました。
これらのディレクトリやそのサブディレクトリの*どちら*にもAsahi関連のものは何もないはずです。
これを修正するには 

```sh
rm -rf /etc/wireplumber/wireplumber.conf.d/*asahi*
rm -rf /etc/wireplumber/main.lua.d/*asahi*
rm -rf /etc/wireplumber/policy.lua.d/*asahi*
rm -rf /etc/pipewire/pipewire.conf.d/*asahi*
```

これが終わったら、`asahi-audio`とPipewire*と*Wireplumberを再インストールし、システムを再起動してください。

### /usr/share/ に asahi-audio のプリリリースバージョンのファイルがあります
`asahi-audio` のプリリリースバージョンでは1.0と一緒に出荷されたものとは一致しないファイルが `/usr/share/` にあります。
これらのファイルはリリース版と衝突し、問題を引き起こす可能性があります。すべての `asahi-audio` ファイルを手動で削除してください：

```sh
rm -rf /usr/share/asahi-audio/
rm -rf /usr/share/wireplumber/wireplumber.conf.d/*asahi*
rm -rf /usr/share/wireplumber/main.lua.d/*asahi*
rm -rf /usr/share/wireplumber/policy.lua.d/*asahi*
rm -rf /usr/share/pipewire/pipewire.conf.d/*asahi*
```

これが終わったら、`asahi-audio`とPipewire*と*Wireplumberを再インストールし、システムを再起動してください。

### カーネルレベルの安全制御を手動で回避しようとしています
`snd_soc_macaudio.please_blow_up_my_speakers`を追加した場所から削除してください。これは
デフォルトのカーネルコマンドライン、`modprobe.d` または他のどこかです。これが終わったら再起動してください。

### 必要なスピーカーのコーデック設定が適用されていません
これは古いカーネルを使っているか、 `snd_soc_tas2764.apple_quirks` を標準外の値に手動で設定している場合に起こる可能性があります。
上記のように、このモジュールのパラメータへの参照をすべて削除し、カーネルを更新し、再起動してください。
