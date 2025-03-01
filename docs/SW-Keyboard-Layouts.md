2025/3/1時点の[SW-Keyboard-Layouts](https://github.com/AsahiLinux/docs/blob/main/docs/SW-Keyboard-Layouts.md)の翻訳

訳注: キーボードの問題を報告するときは翻訳元のwikiページに英語で行ってください。

---
つまるところ、Linux上でのMacのキーボードレイアウトは完全にめちゃくちゃです。私たちは直したいと思っています。
*あなたの*レイアウトはどうなっているのか、一緒に考えてください。

注意：これはMacBookの内蔵キーボードのみですが、Appleの外付けキーボードも対象となる可能性があります。サードパーティ製のキーボードの問題は報告しないでください。

# 手伝う方法

最初に、Appleの公式文書に従って、キーボードレイアウトを[特定](https://support.apple.com/en-us/HT201794)してください。2つのことを知る必要があります。

* キーボードの種類（ANSI、ISO、日本語）
* 特定の国のレイアウト

次に、キーボードのモデルが正しく選択されていることを確認します。キーボードの設定（KDE: System Settings → Input Devices → Keyboard → Hardware）で、
正しいキーボードのモデルを選択する必要があります。

* ANSI レイアウトの場合: **Apple | Apple Aluminium (ANSI)**
* ISOレイアウトの場合: **Apple | Apple Aluminium (ISO)**
* 日本語（JIS）レイアウトの場合: **Apple | Apple Aluminium (JIS)**

最後に、言語に適したレイアウトを選択します。これは単なる言語タイプで*あるべき*なので、Mac特有のカスタマイズはキーボードモデルに基づいて適用されるはずです。
しかしながら、複数の選択肢を試してみてください（例えば、ある言語には*Macintosh*のバリアントがあるかもしれず、事態を悪化させるかもしれません）。

それから、以下のテンプレートを使ってレポートを追加してください（テンプレート部分に従い[PRを送信](https://github.com/AsahiLinux/docs)）。
もし、レイアウトがすでにリストアップされているにもかかわらず、異なる経験（たとえば、別のマシンで）をした場合、新しい*システム構成*サブセクションを
追加し、そこにあなたが見た違いを記してください。

動作するものや、キーキャップに印刷されているものと異なるもの、言語や地域特有の癖や注意すべき点、macOSが行う特別なこと、キーキャップに
印刷されていない隠れた組み合わせで動作するもの、Apple以外のキーボードでWindowsや一般のLinuxデスクトップで行った経験との違いなど、
できるだけ詳しく説明してください。将来正しいことをするために、できるだけ多くの情報が必要です。

これは、macOSとLinuxのショートカットキー（例：OptionとCtrl）の*違い*についてではないことに注意してください。キーボードレイアウトの変更に
よってLinuxデスクトップをmacOSに（正しく）エミュレートすることはできないからです。macOSとLinuxの一般的な違いではなく、
地域のキーボードレイアウトの問題に関心があります。

既知の問題：M2 MacBook Airマシンでは、現在、`hid_apple`ドライバの`iso_layout`種類のデフォルト動作が他のマシンと異なる場合があり、
シフトの右と『1』の左にあるキーを入れ替えることができます。しかし、どちらのオプションもすべてのレイアウトに適しているわけではないので、
これは特定のレイアウトにとって良いことかもしれないし悪いことかもしれません（これは私たちが修正したいことの1つです）。ただ、既存の
機器依存の不整合には注意してください。これは次の安定したカーネルリリースで修正され、少なくとも機器間で一貫性を持たせる予定です。

# レポート

## (ANSI|ISO|JIS) - (レイアウト) - (機器)
* XKBキーボードの最適なレイアウト/バリエーション：（あなたのレイアウト）

(あなたのメモをこちらに)
(訳注: レポートは翻訳元のwikiページにお願いします)

### システム構成
```
# Output of running:
cd /sys/module/hid_apple/parameters/; grep . *; pacman -Q xkeyboard-config-asahi; uname -r; cat /proc/device-tree/model; echo; find /sys/devices -name country | xargs cat; dmesg | grep "Keyboard type"
```

## JIS - 日本語

* XKBキーボードのベストレイアウト・バリアント：`Japanese/Japanese`

正しいレイアウトタイプは、*Japanese*（デフォルトバリアント）だけです。*Japanese(Macintosh)*を選択しないでください。これは、
パスワードの入力が不可能になる使い物にならない『かな』レイアウトです。

すべてのキーは、キーキャップに印刷されているように正しくマッピングされています。

IME（fcitx5＋mozc推奨）を使用しているならば、IMEのキーマッピングはおそらくデフォルトでは期待しているようには
なっていないでしょう。IMEの設定で、『英数』（*Eisu toggle*）を『*Deactivate Input Method(入力方式を無効にする)*」に、
『かな』（*Hiragana Katakana*）を『*Activate Input Method(入力方式を有効にする)*』にマッピングするのがよいかと思われます。

日本語Macのキーボードには、＼（バックスラッシュ）キーがありません。*Advanced(詳細設定)* → *Configure keyboard options(キーボードオプションの設定)* → *Compatibility options(互換性オプション)* で、2つのオプションを選択できます。

* *日本語アップルキーボード*はOADG109Aのバックスラッシュをエミュレート。一般的なPCのOADG109Aレイアウトのように、バックスラッシュをシフトしていない『_』キーに配置
* *日本語アップルキーボード*はPC106のバックスラッシュをエミュレート。『¥』キーをバックスラッシュに変換

\* 理論上はともかく、このオプションは今のところ壊れているような気がします。意図した動作なのですが...。

### システム構成
```
fnmode:3
iso_layout:-1
swap_ctrl_cmd:0
swap_fn_leftctrl:0
swap_opt_cmd:0
xkeyboard-config-asahi 2.35.1_3-1
6.2.0-asahi-6-1-edge-ARCH
Apple MacBook Pro (14-inch, M1 Pro, 2021)
0f
00
```

## ANSI - 韓国語 - M2 MBA
* XKBキーボードのベストレイアウト・バリアント：`Korean`

`iso_layout`を`-1`に設定すると、2021-03-15現在、`1`キーの隣にある`〜`/`＼`キー（`₩`の上に`〜`と刻印されている）が代わりに`<`/`>`として動作します。

### システム構成
```
fnmode:3
iso_layout:-1
swap_fn_leftctrl:0
swap_opt_cmd:0
xkeyboard-config-asahi 2.35.1_3-1
6.1.0-asahi-2-2-edge-ARCH
Apple MacBook Air (13-inch, M2, 2022)
0d
00
00
00
00
00
```

## ISO - スイス - M1 Pro MBP
* XKBキーボードのベストレイアウト・バリアント：『German (Switzerland)』

『German (Switzerland,Macintosh）』もありますが、プレビューに失敗します。上記で特に問題は見つからなかったので、Macintoshの
バリアントがあるメリットはわかりません。

なお、スイスでは、4つの国語があります。ドイツ語、フランス語、イタリア語、ルマンシュ語です。ドイツ語とフランス語には、それぞれ
XKBキーボードレイアウトバリアント（『French (Switzerland)』）がありますが、物理レイアウトは同じです。上記はドイツ語配列に適用され、
フランス語配列についてあまりわかりません。

### システム構成
```
fnmode:3
iso_layout:-1
swap_ctrl_cmd:0
swap_fn_leftctrl:0
swap_opt_cmd:0
xkeyboard-config-asahi 2.35.1_3-1
6.2.0-rc3-asahi-7-1-edge-ARCH
Apple MacBook Pro (14-inch, M1 Pro, 2021)
00
00
0d
00
```

## ISO - ドイツ / オーストリア - M1 Air 2020
* XKBキーボードのベストレイアウト/バリアント: "German (Austria)"

何も問題はなく、すべて期待通り動作します。

### システム構成
```
fnmode:2
iso_layout:-1
swap_ctrl_cmd:0
swap_fn_leftctrl:1
swap_opt_cmd:1
xkeyboard-config-asahi 2.35.1_3-1
6.5.0-asahi-15-1-edge-ARCH
Apple MacBook Air (M1, 2020)
00
0d
```

## ISO - イタリア語 - MBP 16インチ M1 Pro
* XKBキーボードのベストレイアウト/バリアント: **Italian**

Gnomeを使っていて、イタリア語（Machinintosh）が存在しますが、完全に間違っていて、奇妙なqzertyレイアウトを使用しています。
イタリア語レイアウトでは、alt gr機能がなく、右コマンドが左altとして動作しているようで、@, #, €, eccが打てないのです。

```
fnmode:3
iso_layout:-1
swap_ctrl_cmd:0
swap_fn_leftctrl:0
swap_opt_cmd:0
xkeyboard-config-asahi 2.35.1_3-1
6.2.0-asahi-11-1-edge-ARCH
Apple MacBook Pro (16-inch, M1 Pro, 2021)
0d
00
```

## ISO - イタリア語 - MBP 14インチ M2 Pro
* ベストハードウェアモデル: **Apple|Apple**
* XKBキーボードのベストレイアウト/バリアント: **Italian**

```
fnmode:3
iso_layout:-1
swap_ctrl_cmd:0
swap_fn_leftctrl:0
swap_opt_cmd:0
```

## ISO - フランス語 - M1 Pro MBP
* XKBキーボードのベストレイアウト/バリアント: **Apple | Apple Aluminium (ISO)** バリアントなし

バリアントを選択すると、間違ったマッピングになります。

このテストは、Apple MacBook Pro (14-inch, M1 Pro, 2021)で、KDE, Wayland, xkeyboard-config-asahi を使って行われました。

## ISO - ギリシャ語 - M1 Max MBP
* XKBキーボードのベストレイアウト/バリアント: **US**及びに**Greek**

§±キーは、`~ として機能します(両レイアウトとも)。

`~キーは «» として機能します(両レイアウトとも)。

```
fmode:3
iso_layout:-1
swap_fn_leftctrl:0
xkeyboard-config-asahi 2.35.1_3-1
6.1.0-asahi-2-2-edge-ARCH
Apple Macbook Pro (14-inch, M1 Max, 2021)
0d
00
```

## ISO - トルコ語 Q - M2 MBP
* XKBキーボードのベストレイアウト/バリアント: Turkish または tr

キーボードは非Macキーボードとほぼ同じように動作します。Alt Gr（右Alt）を必要とする組み合わせは、右Optionでのみ動作します。
これはmacOSでの動作とは異なりますが、正常に動作する方法です。注意点として、『Alt Gr + A』は、通常『æ』を出力するのに『â』を出力してしまいます。

The keyboard mostly works like it should on a non-Mac keyboard. Combinations that require Alt Gr (right Alt) only work with right Option, which is not how it works on macOS but it's the way it works normally. One caveat is that `Alt Gr + A` outputs â, when it normally should output æ.

### システム構成
```
fnmode:3  
iso_layout:-1  
swap_ctrl_cmd:0  
swap_fn_leftctrl:0  
swap_opt_cmd:0  
xkeyboard-config-asahi 2.35.1_3-1  
6.2.0-asahi-11-1-edge-ARCH  
Apple MacBook Pro (13-inch, M2, 2022)  
00  
00  
0d
```

## ANSI US - ポーランド語 - M1 Max MBP 2021

* XKBキーボードのベストレイアウト/バリアント: pl

起動時のキーボードモデルのデフォルトはGenericになります。

発音記号付きの文字（ąćęłóćżź）は、正しいオプションキーで動作します： 正常
特殊文字（数字＋シフト）：正常
特殊文字の右から文字へ：正常
バックチック／チルダ左から数字：正常

### システム構成
```
fnmode:3
iso_layout:-1
swap_fn_leftctrl:0
swap_opt_cmd:0
xkeyboard-config-asahi 2.35.1_3-1
6.1.0-rc6-asahi-4-1-ARCH
Apple MacBook Pro (16-inch, M1 Max, 2021)
21
00
```

## ANSI US - 中国本土の句読点付き - dvorakに物理的にリマップ - M1 Pro MBP 2021

* XKBキーボードのベストレイアウト/バリアント: zh-tw(Zhuyin/Chewing), en-us, ee, fi, ラテンアルファベットを使用する言語用のdvorakバリアント

システムレイアウトが英語（Macintosh）で、英語（Dvorak, Macinstoh）のレイアウトを選択しようとすると、自動的に英語（Macintosh）fcitx5にフォールバックします,。

English(US) - Englsh(Dvorak, Macintosh): 正常に動作

Chewing(Chinese Taiwan): レイアウトをdvorakで選択しても問題なし、句読点もdvorakの対応位置にマッピング

Estonian(Dvorak): äõöüを右opt + aoeuで通常のANSI dvorak句読点

Finnish(Dvorak): öÖ = ;: ANSIでは、å = opt + o、デッドキーを使わないとäが打てない。opt + ; =¨ 超変

### システム構成
```
fnmode:3
iso_layout:-1
swap_ctrl_cmd:0
swap_fn_leftctrl:0
swap_opt_cmd:0
xkeyboard-config-asahi 2.35.1_3-1
6.2.0-asahi-11-1-edge-ARCH
Apple MacBook Pro (14-inch, M1 Pro, 2021)
00
21
00
```

## ISO - 国際版英語 - M1 Air
* ベストハードウェアモデル: **Apple|Apple Aluminum (ISO)**
* XKBキーボードのベストレイアウト/バリアント: **English (US)**

* 自身のオリジナルキーボードはISO Germanで、国際版英語用のシールを貼付。Mac OSでは正常に動作
* <https://www.apple.com/uk/shop/product/MK2A3Z/A/magic-keyboard-international-english> のように見える
* プレビューは 『English (US)』で機能し、国際版英語のバリエーションでは機能せず
* English (US) との違い1： Shift と Z の間のキーは物理キーボードでは ~` だが、Fedora では < > を入力
* English (US) との違い2： 左上の 1 の隣のキーは、物理キーボードでは +- § だが、Fedora では ~` を入力

### システム構成
```
fnmode:3
iso_layout:-1
swap_ctrl_cmd:0
swap_fn_leftctrl:0
swap_opt_cmd:0
warning: database file for 'core' does not exist (use '-Sy' to download)
warning: database file for 'community' does not exist (use '-Sy' to download)
warning: database file for 'extra' does not exist (use '-Sy' to download)
error: package 'xkeyboard-config-asahi' was not found
6.10.6-401.asahi.fc40.aarch64+16k
Apple MacBook Air (M1, 2020)
00
0d
```
