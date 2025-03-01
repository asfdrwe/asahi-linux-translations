2025/3/1時点の[HW-Clocks](https://github.com/AsahiLinux/docs/blob/main/docs/HW-Clocks.md)の翻訳

---
ADTには（少なくとも）2種類のクロックが定義されています。

* `clock-gates` または `power-gates`: `pmgr` ADT ノードの `devices` 配列をインデックス化
* `clock-ids`： クロックの周波数と`arm-io` ノードの `clock-frequencies` 配列のインデックスを表示

## clock-gates/power-gates
Gated clockは、特定のペリフェラルへのクロック信号が使用されていないときに、そのクロック信号を取り除くために使用されます。
特定のADTノードのIDは通常そのデバイスのMMIO領域にアクセスできるようになる前にオンにする必要があります。

おそらくこれらのクロックに関連するトポロジーが存在していると思われます。例えば、UART ノードが `UART0` しか要求していないのに
 UART MMIO 領域にアクセスするには `SIO`, `UART_P` と `UART0` が必要となるようだからです。

## clock-ids

これらのクロックはおそらくiBootによって事前に設定され、XNU自身は決して触れません。その周波数はADTのノードとして渡されます。
ローインデックス(0x100未満、おそらく6個程度)は`cpu0`ノードで指定されたクロックで、今のところ全て24MHzに設定されているようです
(つまり `bus-frequency` )。0x100 以上のインデックスは `arm-io` ノードの `clock-frequencies` 配列にマップされます。
これらは通常UARTやI2Cバスなどのリファレンスクロックとして使用されます。

`clock-frequencies-regs` プロパティもありますが、その中のレジスタが上記のクロックに正確にどのようにマッピングされるかは不明です。
このプロパティもXNUでは全く使用されていないようです。
