2025/3/1時点の[M4-Series-Feature-Support](https://github.com/AsahiLinux/docs/blob/main/docs/M4-Series-Feature-Support.md)の翻訳

---
このページでは、現存するすべてのM4シリーズ(M4, M4 Pro, M4 Max)の Apple Silicon Mac で現在対応してている機能と、
その上流の状況について詳しく説明します。表は次のように解釈されます:

* **kernel release, 例 6.0:** 機能はこのリリースの時点で上流に取り込み済み
* **linux-asahi (kernel release):** 機能は安定しており、`linux-asahi`で使用可能で、指定されたリリースまでに上流に統合される予定
* **linux-asahi:** 機能は (ほぼ) 安定しており、Fedora Asahi Remixと`linux-asahi`で使用可能
* **linux-asahi-edge**: 機能は(ほぼ) 安定しておりFedora Asahi Remixで使用可能。歴史的経緯によりAsahi Linuxディストリビューション内の`linux-asahi-edge`パッケージで使用可能
* **作業中**: この機能の開発は活発に進められているが、まだ広くテスト、使用、配布する準備ができていない
* **未着手**: この機能に関するアクティブな作業は現時点では未着手

機能にまだ対応できていない場合、それがいつ対応できるかの見積もりはありません。[サポートチャンネル(IRCなど)で見積もりを求めないでください。](When-will-Asahi-Linux-be-done.md)

## 目次
- [SoCブロック](#socブロック)
- [M4機器](#m4機器)
- [M4 Pro/Max機器](#m4-promax機器)
- [注釈](#注釈)

注 これらのマシンはまだ一般発売されていないため(訳注:2025/3/1時点で発売済みです)、対応している機能と不足している機能は、Appleが過去にSoCごと、機器ごとに変更した内容に基づいた
予測です。これらの機器への対応に関する作業が始まれば、このページは迅速に変更されるでしょう。


## SoCブロック
これらは、指定されたSoCを搭載したすべてのデバイスに存在する機能/ハードウェアブロックです。

|                  | M4<br>(T8132)        | M4 Pro/Max<br>(T604x)       |
|------------------|:--------------------:|:---------------------------:|
| DCP              | 未着手                 | 未着手                      |
| USB2 (TB ports)  | linux-asahi          | linux-asahi                 |
| USB3 (TB ports)  | linux-asahi          | linux-asahi                 |
| Thunderbolt      | 未着手                 | 未着手                      |
| DP Alt Mode      | 作業中                | 作業中                       |
| GPU              | 未着手                 | 未着手                      |
| Video Decoder    | 未着手                 | 未着手                      |
| NVMe             | 5.19                 | 5.19                        |
| PCIe             | 未着手                 | 未着手                      |
| PCIe (GE)        | -                    | -                           |
| cpufreq          | 6.2                  | 6.2                         |
| cpuidle          | linux-asahi ([注釈参照](#cpuidleの状況)) | linux-asahi ([注釈参照](#cpuidleの状況))       |
| Suspend/sleep    | asahi-edge           | asahi-edge                  |
| Video Encoder    | 未着手                 | 未着手                      |
| ProRes Codec     | 未着手                 | 未着手                      |
| AICv3            | 未着手                 | 未着手                      |
| DART             | 未着手                 | 未着手                      |
| PMU              | 未着手                 | 未着手                      |
| UART             | 5.13                 | 5.13                        |
| Watchdog         | 5.17                 | 5.17                        |
| I<sup>2</sup>C   | 5.16                 | 5.16                        |
| GPIO             | 5.16                 | 5.16                        |
| USB-PD           | 5.16                 | 5.16                        |
| MCA              | 6.1 / 6.4 (dts)      | linux-asahi                 |
| SPI              | linux-asahi          | linux-asahi                 |
| SPI NOR          | linux-asahi          | linux-asahi                 |
| SMC              | linux-asahi          | linux-asahi                 |
| SPMI             | linux-asahi          | linux-asahi                 |
| RTC              | linux-asahi          | linux-asahi                 |
| SEP              | 作業中                | 作業中                       |
| Neural Engine    | ツリー外([注釈参照](#ANEドライバ))     | ツリー外([注釈参照](#ANEドライバ))             |

## M4機器

|                    | MacBook Pro <br> (14インチ, 後期 2023) | MacBook Air <br>(13インチ と 15インチ 2024) |
|--------------------|:-----------------------------------:|:---------------------------------------:|
| インストーラー        | いいえ                              | いいえ                              |
| DeviceTree         | 未着手                               | 未着手                               |
| メインディスプレイ    | 未着手                               | 未着手                               |
| キーボード           |  未着手                             | 未着手                               | 
| キーボードバックライト |  未着手                             | 未着手                               | 
| タッチパッド         |  未着手                             | 未着手                             | 
| 輝度調整            | 未着手                              | 未着手                               | 
| バッテリー情報       |  未着手                              | 未着手                               | 
| WiFi               | 未着手                             | 未着手                               | 
| Bluetooth          | 未着手                             | 未着手                               |  
| HDMI 出力          |  未着手                             | 未着手                               | 
| 3.5mm ジャック      | 未着手                              | 未着手                               |  
| スピーカー　         | 未着手                              | 未着手                               |  
| マイク　　　         | 未着手                              | 未着手                               |
| Webcam             | 未着手                               | 未着手                               | 
| SD card slot       |  未着手                              | 未着手                               | 
| TouchID            |  未着手                              | 未着手                               | 

## M4 Pro/Max機器

|                    | Mac mini<br>(2024) | MacBook Pro<br>(14-inch, Nov 2024)  | MacBook Pro<br>(16-inch, Nov 2024) |
|--------------------|:------------------:|:-----------------------------------:|:----------------------------------:|
| インストーラー       | いいえ              | いいえ                                | いいえ                                 |
| DeviceTree         | 未着手              | 未着手                              | 未着手                               |
| メインディスプレイ    | 未着手              | 未着手                              | 未着手                               |
| キーボード           | -                  | 未着手                              | 未着手                              |
| キーボードバックライト | -                  | 未着手                               | 未着手                              |
| タッチパッド         | -                  | 未着手                              | 未着手                              |
| 輝度調整　　         | 未着手              | 未着手                               | 未着手                             |
| バッテリー情報       | -                  | 未着手                               | 未着手                              |
| WiFi               | 未着手              | 未着手                               | 未着手                              |
| Bluetooth          | 未着手              | 未着手                               | 未着手                              |
| HDMI 出力           | -                  | 未着手                               | -                                  |
| 3.5mm ジャック       | 未着手              | 未着手                               | 未着手                               |
| スピーカー           | 未着手              | 未着手                              | 未着手                               |
| マイク              | 未着手              | 未着手                                | 未着手                                |
| Webcam             | 未着手              | 未着手                                | 未着手                                |
| SD card slot       | -                  | 未着手                               | -                                  |
| 1Gbps Ethernet     | 未着手              | -                                   | -                                  |
| 10Gbps Ethernet    | -                  | -                                   | -                                  |
| TouchID            | -                  | 未着手                                | 未着手                               |

注: 多くのペリフェラルはDARTやPCIe 対応に依存

## 注釈
### cpuidleの状況
ARMマシンの一部の電源管理機能は、PSCIインターフェイスを介して制御されます。kernelは、PSCIとの対話に関するApple Siliconと互換性のない特定の方法があり、
最善の方法を見つけるためには上流のメンテナーとの議論が必要です。2年間の議論が失敗したので、この機能をAsahi Linuxに搭載するために、
WFI/WFE命令を直接呼び出すドライバをハックすることを決めました。これはエネルギーを意識した(enegy-aware)スケジューリングとともにノートパソコンでの
UXを大きく改善させ、マシンが発熱する問題を解決し、バッテリーの持ちを大幅に向上させました。これは決して上流に統合できませんが、
近い将来どこかでこのハックしたドライバが不要になると思っています。

### ANEドライバ
ツリー外の[カーネルモジュール](https://github.com/eiln/ane/tree/main)が利用可能です。 linux-asahiに統合される予定です。
