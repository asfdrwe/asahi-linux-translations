---
title: ディスプレイコントローラー
---

2025/3/9時点の[display-controllers](https://github.com/AsahiLinux/docs/blob/main/docs/hw/soc/display-controllers.md)の翻訳

---
Mシリーズのチップには`dcp`と`dcpext`の2種類のディスプレイコントローラがあります。どちらの種類にも次のように対応しています。
- DP 1.4 (4レーン、DSC付き): MSTはなし！
- HDMI (dp2hdmiコンバーター経由): ルーティングの制限については以下を参照
- USB-Cポート: DP altmodeまたは各ポート最大2コントローラーのUSB4トンネリング。ルーティングの制限については以下を参照

コントローラー固有情報:
| 種類 | モード制限 |
| - | - |
| `dcp` | 5K, 60Hz, 10bpp (Apple提供情報、未テスト) |
| `dcpext` | 6K, 60Hz, 10bpp (Apple提供情報、未テスト) |

M1 のルーティング制限:
| コントローラー | 内部ディスプレイ | HDMI | USB-C |
| - | - | - | - |
| `dcp` | + | + | |
| `dcpext` | | | + |

M2 以降のルーティング制限:
| コントローラー | 内部ディスプレイ | HDMI | USB-C |
| - | - | - | - |
| `dcp` | + | + | + |
| `dcpext` |  | + | + |

SoC固有情報:
| SoC | `dcp`数 | `dcpext`数 | 注 |
| - | - | - | - |
| M1 | 1 | 1 | |
| M1 Pro | 1 | 2 | |
| M1 Max | 1 | 4 |
| M1 Ultra | 1 | 8 | 内部ディスプレイを持つUltra デバイスなし。ダイの一つで `dcp` が無効、もう一つの `dcp` は HDMI にルーティング |
| M2 | 1 | 1
| M2 Pro | 1 | 2
| M2 Max | 1 | 4
| M2 Ultra | 0 | 8 | 内部ディスプレイを持つ Ultra デバイスなし。両方のダイの `dcp` で無効 |
| M3 | 1 | 1 |
| M3 Pro | 1 | 2 |
| M3 Max | 1 | 4 |
| M4 | ? | ? | 未着手 |
| M4 Pro | ? | ? | 未着手 |
| M4 Max | ? | ? | 未着手 |
