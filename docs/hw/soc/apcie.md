---
title: Apple PCIe コントローラー
---

2025/3/9時点の[apcie](https://github.com/AsahiLinux/docs/blob/main/docs/hw/soc/apcie.md)の翻訳

---
PCIeホスト・ブリッジには少なくとも一部のSynopsys DesignWare由来のロジックが含まれています。 
PCIEコンフィグスペースのオフセット0x8f8/0x8fcにエンコードされているリリースバージョンはバージョン530*-ea15（5.30a-ea15）を示しています。

## ADT バインディング

| プロパティ | 値 | 意味 |
|-------------------|------------------|-------------------|
| compatible        | apcie,t8103      | 互換性のある文字列 |
| #address-cells    | 3                | 通常のPCI DT: `<BAR type><addr>len>` |
| #size-cells       | 2                | 通常のPCI DT     |
| interrupt-parent  | -                | AICの親ハンドル    |
| interrupts        | (0x2b7, 0x2ba, 0x2bd) | 管理用の割り込み。AERかも？ |
| msi-parent-controller | -            | AICの親ハンドル   |
| #msi-vectors      | 32               | MSI用のAICのIRQの数 |
| msi-address       | 0xfffff000       | MSI-X address BARにこれをプログラム |
| msi-vector-offset | 0x2c0            | (これ + msi_id)をMSI-X device BAR内のMSI-X value fieldへ |
| #ports            | 3                | DARTバインディングの数 |
| apcie-common-tunables | 0x2c, 0x4, 0xff, 0x0, 0x1, 0x0, 0x54, 0x4, 0xffffffff, 0x0, 0x140, 0x0 | ?
| apcie-axi2af-tunables | todo | ? |
| apcie-phy-tunables | todo | ? |
| apcie-phy-ip-pll-tunables | todo | ? |
| apcie-phy-ip-auspma-tunables | todo | ? |

### レジスタ

| アドレス | 長さ | 意味 |
|-------------|-------------|----------------------------|
| 0x690000000 | 0x10000000  | ECAM space
| 0x680000000 | 0x40000     | Ctrl
| 0x680080000 | 0x90000     | Phy config
| 0x6800c0000 | 0x20000     | ?
| 0x68c000000 | 0x4000      | ?
| 0x3d2bc000  | 0x1000      | ?
| 0x681000000 | 0x8000      | port0 link / control registers
| 0x681010000 | 0x1000      | port0
| 0x680084000 | 0x4000      | port0 phy
| 0x6800c8000 | 0x16610     | port0
| 0x682000000 | 0x8000      | port1 link / control registers
| 0x682010000 | 0x1000      | port1
| 0x680088000 | 0x4000      | port1 phy
| 0x6800d0000 | 0x6000      | port1
| 0x683000000 | 0x8000      | port2 link / control registers
| 0x683010000 | 0x1000      | port2
| 0x68008c000 | 0x4000      | port2 phy
| 0x6800d8000 | 0x6000      | port2

## 既知のレジスタの意味

|    空間   |    オフセット   |      名前      | 意味または値       |
|-------------|--------------|----------------|------------------------|
| Ctrl        | 0x28         | Refclk         | 1 << 4 は良好
| Ctrl        | 0x50         | ?              | PCIeを有効にするために1を書き込む
| Ctrl        | 0x58         | ?              |  0x50の書き込み後に1を読み出す
| Phy config  | 0x0          | ?              | ビット0x1と0x2を書き込むとそれぞれ0x4と0x8が切り替わる
| portX link  | 0x100        | pcielint　
| portX link  | 0x208        | linksts        |ビット0x1はリンクが有効であることを意味。0x804への書き込みが必要
| portX link  | 0x210        | linkcdmsts
| portX link  | 0x800        | ?              | initializeRootComplex()で読み込む
| portX link  | 0x804        | ?              | enablePortHardware()で読み込む
| portX phy   | 0x0          | PhyGlueLaneReg / RefClockBuffer | ビット0x1と0x2を書き込むと、ビット0x4と0x2がそれぞれトグルされる

## 調整項目
以下の調整項目はポートごとにPCIeブリッジデバイスのコンフィグ空間で動作します。

### pcie-rc-tunables
2020 M1 miniでは、この一連のレジスタ書き込みはその他のレジスタと同様に標準化されたケイパビリティ構造のいくつかのビットを変更します。
| レジスタ | ケイパビリティ | 効果 |
|----------|------------|--------|
| 0x194    | L1 PM Substates | Port Common_Mode_Restore_Timeをクリア |
|          |                 | Port T_POWER_ON Scaleをクリア |
|          |                 | Port T_POWER_ON Valueをクリア|
| 0x2a4    | Data Link Feature | データリンク機能エクスチェンジイネーブルをクリア|
| 0xb80    |             | （拡張）ケイパビリティ構造の一部ではない |
| 0xb84    |             | （拡張）ケイパビリティ構造の一部ではない |
| 0x78     | PCI Express |  Max_Read_Request_Sizeをクリア|

### pcie-rc-gen3-shadow-tunables
| レジスタ | ケイパビリティ | 効果 |
|----------|------------|--------|
| 0x154    | Secondary PCI Express | ダウンストリームポート8.0GT/sトランスミッタープリセットの設定 |
|          |                       | アップストリームポート8.0GT/sトランスミッタープリセットの設定 |
| 0x890    |            | （拡張）ケイパビリティ構造の一部ではない |
|          |            | Synopsys DesignWareのPCIe GEN3_RELATEDレジスタと思われる |
| 0x8a8    |            | （拡張）ケイパビリティ構造の一部ではない |
|          |            | Synopsys DesignWareのPCIe GEN3_EQ_CONTROLレジスターのようだ |

### pcie-rc-gen4-shadow-tunables
| レジスタ | ケイパビリティ | 効果 |
|----------|------------|--------|
| 0x178    | Physical Layer 16.0 GT/s | ダウンストリームポート 16.0 GT/s トランスミッタープリセットの設定 |
|          |                          | アップストリームポート 16.0 GT/s トランスミッタープリセットの設定 |
| 0x890    |            |  |
|          |            | （上記参照) |
| 0x8a8    |            |  拡張）ケイパビリティ構造の一部ではない|
|          |            | （上記参照) |

つまり、文書化されたレジスタの変更はいくつかの(バグのある？)機能を無効にすると同時に、レーンのイコライジングの調整も行うようです。xnuドライバで特定のシリコンrevを指定しなくても、未来のシリコンのrespinでこの機能を再び有効にしたいとAppleは考えているのではないでしょうか？

## DT バインディング
DeviceTreeのバインディングは上流で受け入れられています。

いくつかの未解決の問題が残っています。

- WiFi/BT PCIe デバイスを有効にするにはどうすればよいですか？このデバイスは、PCIe デバイスとして表示される前に、
SMC を通して明示的に有効にする必要があります。これはAppleがどのように『機内モード』を実装しているかを
示唆し、このためにADTに別の『amfm』ノードが存在することを示唆しています。したがって、これを処理するある種の
rfkillデバイス/ノードを持つことは理にかなっているのかもしれません。うまくいけば、APCIeデバイスがオンに
なったときに割り込みを取得し、PCIeリンクを（再）トレーニングできるようになります。

この提案されたバインディングは、u-boot と OpenBSD でうまく実装/テストされています。しかし、これをすべて
動作させるには、クロック、pinctrl/gpio、DARTのバインディングがまだ必要です。
