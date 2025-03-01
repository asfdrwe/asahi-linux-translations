2025/3/1時点の[M1-Series-Feature-Support](https://github.com/AsahiLinux/docs/blob/main/docs/M1-Series-Feature-Support.md)の翻訳

訳注: 文書のリンクは対応する日本語訳へのリンクに変更。

---
このページでは、現存するすべてのM1シリーズ(M1, M1 Pro, M1 Max, M1 Ultra)のApple Silicon Macで現在対応してている機能と、
その上流の状況について詳しく説明します。表は次のように解釈されます:

* **kernel release, 例 6.0:** 機能はこのリリースの時点で上流に取り込み済み
* **linux-asahi (kernel release):** 機能は安定しており、Fedora Asahi Remixで使用可能で、指定されたリリースまでに上流に統合される予定
* **linux-asahi:** 機能は (ほぼ) 安定しており、Fedora Asahi Remixで使用可能
* **作業中**: この機能の開発は活発に進められているが、まだ広くテスト、使用、配布する準備ができていない
* **未着手**: この機能に関するアクティブな作業は現時点では未着手

機能にまだ対応できていない場合、それがいつ対応できるかの見積もりはありません。[サポートチャンネル(IRCなど)で見積もりを求めないでください。](When-will-Asahi-Linux-be-done.md)

## 目次
- [SoCブロック](#socブロック)
- [M1機器](#m1機器)
- [M1 Pro/Max/Ultra機器](#m1-promaxultra機器)
- [注釈](#注釈)

## SoCブロック
これらは、指定されたSoCを搭載したすべてのデバイスに存在する機能/ハードウェアブロックです。
|                  | M1<br>(T8103)        | M1 Pro/Max/Ultra<br>(T600x) |
|------------------|:--------------------:|:---------------------------:|
| DCP              | linux-asahi          | linux-asahi                 |
| USB2 (TB ports)  | linux-asahi          | linux-asahi                 |
| USB3 (TB ports)  | linux-asahi          | linux-asahi                 |
| Thunderbolt      |　作業中                | 作業中                      |
| DP Alt Mode      |　作業中                | 作業中                      |
| GPU              |  linux-asahi          | linux-asahi                 |
| Video Decoder    | 作業中         　      | 作業中                      |
| NVMe             | 5.19                 | 5.19                        |
| PCIe             | 5.16                 | 5.16                        |
| PCIe (GE)        | -                    | -                           |
| cpufreq          | 6.2                  | 6.2                         |
| cpuidle          | linux-asahi([注釈](#cpuidleの状況)) | linux-asahi([注釈](#cpuidleの状況))        |
| Suspend/sleep    | linux-asahi          | linux-asahi                 |
| Video Encoder    | 作業中                | 作業中                       |
| ProRes Codec     | -                    | 未着手                       |
| AICv2            | -                    | 5.18                        |
| DART             | 5.15                 | 6.1                         |
| PMU              | 5.18                 | 5.18                        |
| UART             | 5.13                 | 5.13                        |
| Watchdog         | 5.17                 | 5.17                        |
| I<sup>2</sup>C   | 5.16                 | 5.16                        |
| GPIO             | 5.16                 | 5.16                        |
| USB-PD           | 5.16                 | 5.16                        |
| MCA              | 6.1                  | 6.1                         |
| SPI              | linux-asahi          | linux-asahi                 |
| SPI NOR          | linux-asahi          | linux-asahi                 |
| SMC              | linux-asahi          | linux-asahi                 |
| SPMI             | linux-asahi          | linux-asahi                 |
| RTC              | linux-asahi          | linux-asahi                 |
| SEP              | 作業中                | 作業中                       |
| Neural Engine    | ツリー外([注釈](#ANEドライバ))  | [注釈](#ANEドライバ)        　    |

## M1機器
|                    | Mac Mini<br>(2020)   | MacBook Pro<br>(13-inch, 2020) | MacBook Air<br>(2020) | iMac<br>(2021)       |
|--------------------|:--------------------:|:------------------------------:|:---------------------:|:--------------------:|
| Installer          | 対応                 | 対応                            | 対応                   | 対応                  |
| DeviceTree         | 5.13                 | 5.17                           | 5.17                  | 5.17                 | 
| メインディスプレイ    | 5.17                 | 5.17                           | 5.17                  | 5.17                | 
| 輝度調整             | -                    | linux-asahi                    | linux-asahi           | linux-asahi          |
| HDMI 出力           | 5.13                 | -                              | -                     | -                    |
| HDMI Audio         | linux-asahi([注釈](#hdmi-audio)) | -                              | -                     | -                    |
| キーボード           | -                    | linux-asahi                    | linux-asahi           | -                    |
| キーボードバックライト | -                    | 6.4                            | 6.4                   | -                    |
| タッチパッド         | -                    | linux-asahi                    | linux-asahi           | -                    |
| バッテリー情報       | -                    | linux-asahi                    | linux-asahi           | -                    |
| USB-A ports        | 5.16                 | -                              | -                     | -                    |
| WiFi               | 6.1                  | 6.1                            | 6.1                   | 6.1                  |
| Bluetooth          | 6.2                  | 6.2                            | 6.2                   | 6.2                  |
| 3.5mm ジャック       | linux-asahi          | linux-asahi                    | linux-asahi           | linux-asahi         |
| スピーカー           | linux-asahi ([注釈](#スピーカー)) | linux-asahi ([注釈](#スピーカー)) | linux-asahi ([注釈](#スピーカー)) | 未対応|
| SDカードスロット      | -                    | -                              | -                     | -                   |
| 1Gbps Ethernet     | 5.16                 | -                              | -                     | 5.17                 |
| 10Gbps Ethernet    | 5.17                 | -                              | -                     | -                    |
| マイク              | -                    | 作業中                          | 作業中                 | 作業中                |
| Webcam             | -                    | linux-asahi                    | linux-asahi           | linux-asahi          |
| タッチバー           | -                    | linux-asahi                  | -                     | -                    |
| TouchID            | 未着手                | 未着手                          | 未着手                  | 未着手               |

### M1 Pro/Max/Ultra機器
|                    | MacBook Pro<br>(14/16-inch, 2021) | Mac Studio<br>(2022) |
|--------------------|:---------------------------------:|:--------------------:|
| Installer          | 対応                               | 対応                 |
| DeviceTree         | 6.2                               | 6.2                  |
| メインディスプレイ    | 5.17                              | 5.17                 |
| 輝度調整            | linux-asahi                        | -                    |
| HDMI 出力           | linux-asahi (13.5 FWのみ)          | 6.2                  |
| HDMI Audio         | linux-asahi([注釈](#hdmi-audio))   | linux-asahi([注釈](#hdmi-audio)) |
| キーボード           | linux-asahi                       | -                    |
| キーボードバックライト | 6.4                               | -                    |
| タッチパッド         | linux-asahi                       | -                    |
| バッテリー情報        | linux-asahi                       | -                    |
| USB-A ports        | -                                 | linux-asahi          |
| WiFi               | 6.1                               | 6.1                  |
| Bluetooth          | 6.2                               | 6.2                  |
| 3.5mm ジャック      | linux-asahi                       | linux-asahi          |
| スピーカー          |linux-asahi ([注釈](#スピーカー)) | linux-asahi ([注釈](#スピーカー)) |
| SDカードスロット     | 5.17                              | 5.17                 |
| 1Gbps Ethernet     | -                                 | -                    |
| 10Gbps Ethernet    | -                                 | linux-asahi          |
| マイク              | 作業中                             | -                    |
| Webcam             | linux-asahi                       | -                    |
| タッチバー          | -                                 | -                    |
| TouchID            | 未着手                             | 未着手                |

## 注釈
### cpuidleの状況
ARMマシンの一部の電源管理機能は、PSCIインターフェイスを介して制御されます。kernelは、PSCIとの対話に関するApple Siliconと互換性のない特定の方法があり、
最善の方法を見つけるためには上流のメンテナーとの議論が必要です。2年間の議論が失敗したので、この機能をAsahi Linuxに搭載するために、
WFI/WFE命令を直接呼び出すドライバをハックすることを決めました。これはエネルギーを意識した(enegy-aware)スケジューリングとともにノートパソコンでの
UXを大きく改善させ、マシンが発熱する問題を解決し、バッテリーの持ちを大幅に向上させました。これは決して上流に統合できませんが、
近い将来どこかでこのハックしたドライバが不要になると思っています。

### ANEドライバ
ツリー外[kernel module](https://github.com/eiln/ane/tree/main)で提供されています。 将来linux-asahiに統合されます。

### スピーカー
[lsp-pluginsのバグ](https://github.com/lsp-plugins/lsp-dsp-lib/pull/20)のため、スピーカーは個別のパッチで有効になっています。
このバグはスピーカーにダメージを与える可能性があるフルスケールのアーティファクトを引き起こします。この修正は
lsp-pluginsリリース[1.0.20](https://github.com/lsp-plugins/lsp-dsp-lib/releases/tag/1.0.20)に含まれます。


### HDMI Audio
`asahi-6.8.6-3`以降、M1/M2 Ultraを除くHDMIポートを持つ全ての機器でHDMI audio対応のプレビューが利用できるようになりました。
ユーザースペースでの統合が欠けているため、`Analog Output (Built-in Audio Stereo)`などと表示されます。まだいくつかの不具合があります。
音声の出だしが途切れたり、ノイズが入ったりすることがあります。HDMI表示部分は動作していても、audioが利用できない場合があります。
