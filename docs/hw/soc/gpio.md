---
title: GPIO コントローラ
---

2025/6/2時点の[gpio](https://github.com/AsahiLinux/docs/blob/main/docs/hw/soc/gpio.md)の翻訳

---
## DT binding

ADTの 『gpio, t8101』ノードはpinmux機能を持つGPIOコントローラを表しています。
Corelliumのコードベースから判断すると、ピンは 『gpio』機能と『periph』機能間で切り替え可能です。
これらの2つのモードの切り替えを制御するビットのすぐ隣に未知のビットがあるので、もっと多くのオプションがあるかもしれません。
コントローラがpinmux機能を実装しているので、このハードウェアをFDTのpinctrlノードとしてモデル化する必要があります。 これは汎用のpinctrl/pinmux/gpioバインディングプロパティを使って完全に行うことができます:

```
#define APPLE_PINMUX(pin, func) ((pin) | ((func) << 16))

                pinctrl: pinctrl@23c100000 {
                        compatible = "apple,t8103-pinctrl";
                        reg = <0x2 0x3c100000 0x0 0x100000>;
                        clocks = <&gpio_clk>;

                        gpio-controller;
                        #gpio-cells = <2>;
                        gpio-ranges = <&pinctrl 0 0 212>;

                        pcie_pins: pcie-pins {
                                pinmux = <APPLE_PINMUX(150, 1)>,
                                         <APPLE_PINMUX(151, 1)>,
                                         <APPLE_PINMUX(32, 1)>;
                        };
                };
```

`pinmux`を使用することで、ピンや関数の名前を考えてドライバにハードコードする必要がないという利点があります。
ピンの目的はSoCごとに異なり、またSoC内でもコントローラごとに異なると思われます。 M1 SoCには4つのコントローラがあります。
この例では、32ビットのpinmuxセルを単純に分割して使用しています。
下位16ビットはピン番号を、上位16ビットはピン機能を表します。
ピン機能をエンコードするために16ビット全部が必要になることはほとんどないので、将来必要になったときにこれらのビットの一部を再利用することになります。

いくつかの未解決の問題:
* 互換性のある文字列(compatible string)は、ADT内のノード名から 『apple, t8101-gpio』とすべきでしょうか？ それとも、両方を記載すべきでしょうか？
* コントローラは割り込み機能も提供しているようです。 標準的なバインディングでは`interrupt-controller`プロパティが使えるので、これも処理することができます。コントローラーごとに(最大7つの)AIC割り込みがあり、それぞれがGPIOピンのグループを扱います。 GPIOピンは自由にグループに割り当てることができるようですが、ADTにはプロパティが含まれており、一部のコントローラではすべてのグループが機能してしているわけではないことを示唆しています。

gpioコントローラはそれを `interrupt-parent`として使用するデバイスに割り込み機能を提供します。これらのデバイスは2つの`#interrupt-cell`を持っています。1つ目のセルはGPIOピンを指定します。2つ目のピンの意味は不明です。`audio-tas5770L-speaker`, `audio-codec-output`, `hpmBusManager` は 0x1, `wlan` は 0x2, `bluetooth` は 0x2000002 を使用しています。2つ目のセルの値はピンのコンフィグレーションレジスタには対応していないようです。


デバイス                 | ピン | 第2セル  | ibootによる設定 (mac mini)
---------------------- | --- | --------- | --------------------------
hpmBusManager          | 106 | 0x1       | 0x76b80
bluetooth              | 136 | 0x2000002 | 0x76a80
audio-tas5770L-speaker | 182 | 0x1       | 0x76b81
audio-codec-output     | 183 | 0x1       | 0x76b81
wlan                   | 196 | 0x2       | 0x76ac0

アドレス`0x23c100000`のデバイス`/arm-io/gpio`についてのMac OSの動作の観測:
1. オフセット `0x0000` から `0x34c` までのピン コンフィグ (4 バイト) を読み込む (212 ピン)
2. 7つのグループ？すべての割り込みを解除するには、7つのグループの7×4バイトをオフセット `0x800`, `0x840`, `0x880`, `0x8C0`, `0x900`, `0x940`, `0x980` に書き込む
