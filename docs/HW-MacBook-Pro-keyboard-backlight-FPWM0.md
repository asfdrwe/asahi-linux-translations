2025/3/11時点の[HW-MacBook-Pro-keyboard-backlight-FPWM0](https://github.com/AsahiLinux/docs/blob/main/docs/HW-MacBook-Pro-keyboard-backlight-FPWM0.md)の翻訳

---
MacBook Proでは、キーボードのバックライトはADT内で次のように表示されます。

```
fpwm {
  [...]
  AAPL.phandle = 59
  clock-gates = 37
  device_type = fpwm
  reg = [889470976, 16384]

  kbd-backlight {
    [...]
  }
}
```

このことから、0x235044000にPWMがあり、クロックゲートは0x23b7001e0で有効になり、キーボードバックライトを制御していることがわかります。
そうなっているように見えます :-)

私が理解した限りレジスタは以下の通りです。

```
+0x00: write 0x4239 to enable or after counter values changed
+0x04: unknown, no effect
+0x08: status bits: bit 0x01 is set when the light comes on, 0x02 is set when the light comes off. Write-to-clear.
+0x0c: unknown, no effect
+0x18: off period, in 24 MHz ticks
+0x1c: on period, in 24 MHz ticks
```

つまり、キーボードのバックライトを煩わしく点滅させて発作を誘発する可能性のある完全なm1n1シーケンスは次のようになります。

```
>>> write32(0x23b7001e0, 0xf)
>>> write32(0x23504401c, 1200000)
>>> write32(0x235044018, 1200000)
>>> write32(0x235044000, 0x4239)
```

デューティサイクルを50%に保ちながら周波数を変化させる場合です。

```
>>> write32(0x235044018, 4000)
>>> write32(0x23504401c, 4000)
>>> write32(0x235044000, 0x4239)
```

PRは https://github.com/AsahiLinux/linux/pull/5 です。
