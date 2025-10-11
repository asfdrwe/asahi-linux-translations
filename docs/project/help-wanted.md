---
title: 助っ人募集！
---

2025/10/11時点の[help-wanted](https://github.com/AsahiLinux/docs/blob/main/docs/project/help-wanted.md)の翻訳

---
このページは、やらなければならないけれども、より大きな魚を揚げることに集中するために優先順位が下げられているような
雑多なタスクのリストです。これらのタスクのほとんどは、低い費用と低い労力で済むので、カーネル開発や一般的な
フリーソフトウェアの初心者が始めるには良い場所です。

これらのタスクを引き受けるならば、重複作業を避けるためにタスクの状態を更新してください。
質問や支援が必要な場合は、 `#asahi-dev` に連絡してください。

| タスク | 状況 | 説明 | 連絡先 |
| ---- | ------ | ----------- | ------- |
| libgnome-volume-control 修正 | **手付かず** | GNOME の音量ミキサーは`libgnome-volume-control`プラグインで実装。残念ながら、これはWirePlumber/Pipewireとのやりとりが悪く、ノードグラフのパーミッションを尊重しない。このため、デフォルトのsinkが GNOME の『生の(raw)』ハードウェアデバイスになり、DSP をバイパスしてしまう。`libgnome-volume-control` を修正して、生のハードウェアsinkを非表示にし、正しいデフォルトのsinkを選択するようにする必要あり | chadmed氏 |
| tuxvdmtool をユーザースペース i2c に書き換え | **手つかず** | [tuxvdmtool](https://github.com/AsahiLinux/tuxvdmtool) は [sysfs API](https://github.com/AsahiLinux/linux/commit/786523ac62f0aeec37bf9c6b991e8bf2fadc590d) を使用。sysfs API はあまり使いやすくなく、上流取り込まれる可能性は低い。tuxvdmtool 公開後に、Linux ユーザースペース I2C クライアントドライバ API を使用する方が良い選択であり動作するはずだと結論付け。詳細は https://github.com/AsahiLinux/tuxvdmtool/issues/1 を参照。| janne |
| シリアルポートリセットを修正 | **手つかず** | 2つの Apple Silicon 機器を接続して Apple Silicon デバッグ UART を使用すると、ホストデバイスのシリアルポート（`/dev/ttySAC0`）が`tuxvdmtool reboot serial`を実行してから数秒以内にリセットされる。この原因を特定し、問題を修正。このタスクは、 m1n1 のハイパーバイザーがハードウェア UART0 を自身の用途で上書きしてしまうため、少し厄介 | janne |
| キーボードレイアウトのクリーンアップ (XKB/hid_apple) | 手付かず | LinuxデスクトップのAppleキーボード対応は、レイアウトやハードウェアキーボードを問わず、まちまちです。キーボードドライバはまだ上流ではないので、 大きなクリーンアップをするチャンスがあります。特に、これらの機器のキーボードにはソフトな*Fn*キーがあり、これは完全にソフトウェアで処理されます。現在、`hid_apple`ドライバはこれをカーネルで行っていますが、これは間違ったアプローチです。このキーはXKB/Waylandのユーザースペースで扱われるべきです（Xorgではできませんが、それは非推奨）。そうすれば、ユーザーが他の修飾キーと同じようにキーバインドをカスタマイズしたり、macOSのように特別なシンボルを提供したりするなど、より包括的なFnキーマッピングを行うことができます。これはおそらく、ユーザー空間でこのマッピングを行う新しいXKBキーボードモデルを導入することで実現されるでしょう。これをテストするには、`hid_apple` の `fnmode=0` モジュールパラメータを使用して、すべての Fn キー処理を無効にしてください。あとで、編集キー(insert/delete/home/end/pgup/pgdown)のFnキーの組み合わせエミュレーションだけを行う新しいfnmodeを導入したいと思うかもしれませんが、これは使えるTTYとXorgにとって必要最低限なものであって、それ以外はXKBに任せ、Apple Siliconマシンのデフォルトをこのモードにします。Fnの話以外にも、XKBの設定で修正する必要がある地域限定のMacレイアウトはたくさんあり、英語以外のキーボードを持っている人は、この作業に協力することを歓迎します。[関連するxkeyboard-configの問題](https://gitlab.freedesktop.org/xkeyboard-config/xkeyboard-config/-/issues/379) | janne |
