---
title: USB-PD
---

2025/3/9時点の[usb-pd](https://github.com/AsahiLinux/docs/blob/main/docs/hw/soc/usb-pd.md)の翻訳

---
## はじめに

AppleはカスタムUSB-PDメッセージを使用してデバッグなどの目的でType-Cポートのピンの多重化を制御しています。USB-PDの通信はポートのCCxライン（ポート方向によってCC1またはCC2）で行われます。

情報を提供してくれたt8012devの皆さんに感謝します。 https://web.archive.org/web/20211023034503/https://blog.t8012.dev/ace-part-1/ を参照してください。Apple M1 Mac(2020)のコントローラはCD3217 『Ace2』です。

予備知識として[USB-PD spec](https://www.usb.org/document-library/usb-power-delivery)を参照すると良いでしょう。

AppleはUSB ID(0x5AC)を持つベンダー固有の構造化されたVDM(Vendor Define Message、ベンダ定義メッセージ)メッセージを使用していますが、メッセージは
SOP'DEBUG(UFPから発信する場合)またはSOP"DEBUG(DFPから発信する場合)のパケットスタートトークンを使用する必要があります(規格では未使用のため
コントローラによっては送信できない場合あり)。VDMのヘッダーは0x5ac8000｜（コマンド）形式です。

MacはOSが起動した後にのみDFPとして動作するため、このプロトコルはDFP（すなわち電源）として動作させることが推奨されます。

以下コマンドは、[vdmtool](https://github.com/AsahiLinux/vdmtool)のシリアルコンソールに簡単に貼り付けられるように、16進数カンマ区切りフォーマットになっています。このプロトコルでは、32ビットのVDMワードの上位から下位までをパックした16ビットのデータユニットを多用しており、ゼロ終端となっています。

コマンドの返信には『リクエストコマンド ID｜0x40』を使用します。コマンド0x10への返答は『コマンド 0x50』などです。

## Chips

* CD3215C00 『ACE1』 - これは ROM/OTP コードが異なる TPS65983 らしい
* CD3217B12 『ACE2』 - 不確かだがいくつかの違いを持つ実際に新しいシリコンだろう。他のTI製部品と同じかもしれない。初期のM1デバイスはすべてこの部品を使用。ファームウェアの構成は多少異なる

## ポート

Macの各ポートでVDM対応が異なる場合があります。デバッグ機能は通常1つのポートでのみ対応します。

### 2020 MacBook Air (M1)

端に最も近いポートに全てのデバッグ機能があります。

### 2020年のMac Mini (M1)

一番左のポート（電源入力に一番近い）に全てのデバッグ機能があります。

### 2019年 16インチMacBook Pro (MacBookPro16,1 - Titan Ridge)

前方と後方の左ポートはそれぞれ7つのアクションを報告します。 後方右ポートは4つのアクションを報告します。 前方右ポートは3つのアクションを報告します。

### 2019年モデルの13インチMacBook Pro (MacBookPro15,2 - Titan Ridge)

前方左ポートは8つのアクションを報告しています。 後方左ポートは5つのアクションを報告します。 後方右ポートは、4つのアクションを報告します。 前方右ポートは3つのアクションを報告します。

### 2017年の13インチMacBook Pro (MacBookPro14,2 - Alpine Ridge)

後方左ポートは4つのアクションを報告しています。 前方左ポートと前方右ポートはそれぞれ3つのアクションを報告します。後方右ポートは2つのアクションを報告します。

## コマンド

### 0x10 アクションリストの取得

``5ac8010``

それぞれの『アクション』は行うべきことまたはミックスすべき信号のいずれかです。

M1 Mac Mini (2020) 左ポートからの応答例:

```
5ac8010
>VDM 5AC8010
<VDM RX SOP'DEBUG (7) [704F] 5AC8050 46060606 2060301 3060106 1050303 8030809 1030000
                             ^^^^^^^ ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                             vdm hdr action list
```

これはアクション0x4606、0x606、0x206、0x301、0x306、0x106、0x105、0x303、0x803、0x809、0x103の対応を示しています。

MacBookPro16,1は、前方左ポートの動作0x602、0x606、0x601、0x403、0x302、0x501、0x301、後方左ポートの動作0x205、0x206、0x103、0x602、0x302、0x501、0x301、後方右ポートの動作0xE04、0x501、0x301、0x302、前方右ポートの動作0x302、0x501、0x301に対応しています。

MacBookPro15,2は、前方左ポートのアクション0x207、0x205、0x602、0x606、0x501、0x601、0x301、0x302、後方左ポートのアクション0x403、0x602、0x302、0x501、0x103、0x301、0x302、前方右ポートのアクション0x302、0x501、0x301に対応しています。

MacBookPro14,2では、後方左ポートのアクション0x403、0x602、0x301、0x302、前方左ポートのアクション0x302、0x205、0x301、前方右ポートのアクション0x302、0x802、0x301、後方右ポートのアクション0x301、0x302に対応しています。

### 0x11 アクション情報の取得

```5ac8011,<actionid>```

特定のアクションに関する情報を16ビットのショートユニットで返します（ゼロ終端）。

M1 Mac Miniの場合アクションごとに以下のような情報が返されます:

```
Action  Info reply
4606    0183
0606    0183
0206    0187 020C 0318 8001
0301    0187 020C 0303
0306    0187 020C 800C
0106    8001
0105    8000
0303    0187 0221 0303 809E 0030 6030 000C
0803    0187 0221 8001
0809    0187 0221 8001
0103    8000
```

### 0x12: アクションの実行

```5ac8012,<actionid>[,args]```

指定されたアクションを実行するかピンのセットにマッピングします。

`actionid`は下位16ビットにアクションIDを保持し、フィールドは次のようになります:
```
Bits  Description
25    If 1 exits the mode, instead of entering it
24    Persist through soft reset. Seems to do something in DFP mode.
23    If 1 attempts to exit conflicting modes before entering this one
22-16 Bit mask of lines to map to this action
15-0  Action ID
```

レスポンスの例

```
5ac8012,40306
>VDM 5AC8012 40306
<VDM RX SOP'DEBUG (5) [504F] 5AC8052 44740000 306 0 0
                             ^^^^^^^ ^^^^\------------ pin states
                             vdm hdr connection/line state 
```


* 接続/回線状態: 16ビットのヘッダショート(`(ConnectionState << 14) | (LineState[i] << (2 * i))`,iは0から7、排他的)に続いて、各ピンセットからどのアクションがmuxされるかを示す7つのショートが続く。ConnectionStateは、切断されている場合は0、方向に応じて標準的な接続デバイスの場合は1または2、オーディオおよびデバッグ接続の場合は3。LineStateは2ビットの値でその意味は現在不明
* ピンの状態：1つのピンペアに1つのアクションID、16ビット長

この場合アクション306はピンセット2（3番目のピンセット）にマップされています。

## ピンセット

2020年Mac Mini (M1)より:

* 0: Secondary D+,D- (VCONN側コネクタのUSB2データペア)
* 1: Primary D+,D- (CC側コネクタのUSB2データペア)。これらはホスト側にブリッジされておらず異なる信号を出すことができるのがデバッグポートならではの特徴。ケーブルはCC側に1ペアのみ！
* 2: SBU1,SBU2
* 3-6: 不明(SSTX/SSRXペア?)

Mac側でコネクタの向きを自動調整しているので、ケーブル側から見るとピンは常に同じになります。向きの調整は相手側の機器が行います。

### アクション

### 103: PDリセット

これには0x8000の引数が必要です（『アクション情報の取得』から取得）。

```
5AC8012,0103,80000000
>VDM(D) 5AC8012 103 80000000
Disconnect: cc1=0 cc2=0
VBUS OFF
Disconnected
(PD renegotiation occurs)
```

### 105: 再起動

これには0x8000の引数が必要です(『アクション情報の取得』から抜粋)。

```
5AC8012,0105,80000000
>VDM(D) 5AC8012 105 80000000
<VDM RX SOP"DEBUG (5) [524F] 5AC8052 44740000 306 0 0
Disconnect: cc1=0 cc2=0
VBUS OFF
Disconnected
S: DISCONNECTED
IRQ: VBUSOK (VBUS=OFF)
(device reboots and PD renegotiates)
```

#### rebootコマンドについて
Arduino IDEのシリアルモニタから『5AC8012,0105,80000000』というコマンドが送られてきますが、もっと手軽にMacをリブートしたい場合は以下のコマンドを試してみてください:
```
Option 1:
echo "5AC8012,0105,80000000" | picocom -c -b 500000 --imap lfcrlf -qrx 1000 /dev/<あなたのArduinoシリアルデバイス>

Option 2:
stty 500000 </dev/<あなたのArduinoシリアルデバイス> 
echo > /dev/<あなたのArduinoシリアルデバイス> 
echo 5AC8012,0105,80000000 > /dev/<あなたのArduinoシリアルデバイス> 
```

しかし、[default Arduino operations on Serial port](https://forum.arduino.cc/t/solved-problem-with-serial-communication-on-leonardo/139614)のために、上記のコマンドは多くの場合失敗しランダムに成功します。
しかし、以下のPythonコードは、コマンドデータを送信する前にArduinoを手動でリセットすることで、正常に動作することがわかりました:
```
import serial
import time
ser = serial.Serial("/dev/<your Arduino Serial device>", 500000, dsrdtr=True)
ser.dtr = True
ser.dtr = False
time.sleep(0.5)
ser.dtr = True
time.sleep(2)
ser.write(b'5AC8012,0105,80000000\n')
ser.close() 
```

### 106: DFU / ホールドモード

これには0x8001の引数が必要です(『アクション情報の取得』から取得)。これはDFPモード(MacがUFPとして動作する)でのみ正しく動作します。


```
5AC8012,0106,80010000
>VDM(D) 5AC8012 106 80010000
<VDM RX SOP"DEBUG (5) [544F] 5AC8052 44740000 306 0 0
(device reboots in DFU mode, no PD renegotiation occurs)
```

このモードは特別です。Mac Miniではハードシャットダウンすると通常PD通信とUFPモード（Rd open）が無効になります。しかし、このモードからのハードシャットダウン（例：電源ボタンの長押し）では、PD通信が有効なままマシンがパワーダウンします。また、105を介して通常モードで再起動することも可能で、この場合もPDはリセットされず、既存のモードが有効なままとなります。これは、マシンのリセットを通じてデバッグ接続を維持するために使用することができます。

FIXME: あるいはヘッダの persist ビットのせいかもしれません。もっとテストが必要です。

### 306: デバッグ用 UART

Pin order: TX, RX

これはピンセット0～2（D+/D- B、D+/D- A、またはSBU1/2）にマッピングできます。UARTは1.2Vの電圧レベルを使用しています。

```
5AC8012,840306
>VDM 5AC8012 840306
<VDM RX SOP'DEBUG (5) [584F] 5AC8052 44740000 306 0 0
(UART is now mapped to SBU1/2)
```

ピン1がTX、ピン2がRXで、方向が意味を持ちます（CC=CC2の場合は反転します）。つまりCCピンと同じコネクタ側（AまたはB）のSBUピンがTXになります。

### 606: DFU USB

ピンの順番です。D+, D- (当然)

DFUモードでは自動的にピンセット1（D+/D-プライマリ）にマッピングされますが、移動することも可能です。

DFUをもう一方のD+/D-セットに移動させるには:
```
5AC8012,2020606
>VDM(D) 5AC8012 2020606
<VDM RX SOP"DEBUG (5) [5E4F] 5AC8052 44400000 0 0 0
5AC8012,810606
>VDM(D) 5AC8012 810606
<VDM RX SOP"DEBUG (5) [504F] 5AC8052 44430606 0 0 0
(DFU is now on secondary D+/D- pair (pin set 0))
```

### 4606: USBのデバッグ

面白いです。これはメインCPU駆動ではなくシステムがオフの時にも列挙されます（パーシステントモード）。電源の移行時に再列挙されます。


```
[277048.498917] usb 1-4.4.3: New USB device found, idVendor=05ac, idProduct=1881, bcdDevice= 1.20
[277048.498920] usb 1-4.4.3: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[277048.498921] usb 1-4.4.3: Product: Debug USB
[277048.498921] usb 1-4.4.3: Manufacturer: Apple Inc.
```

<details>
<summary>lsusb</summary>

```
Bus 001 Device 097: ID 05ac:1881 Apple, Inc. Debug USB
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass          255 Vendor Specific Class
  bDeviceSubClass       255 Vendor Specific Subclass
  bDeviceProtocol       255 Vendor Specific Protocol
  bMaxPacketSize0        64
  idVendor           0x05ac Apple, Inc.
  idProduct          0x1881 
  bcdDevice            1.20
  iManufacturer           1 Apple Inc.
  iProduct                2 Debug USB
  iSerial                 0 
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength       0x0027
    bNumInterfaces          1
    bConfigurationValue     1
    iConfiguration          0 
    bmAttributes         0xc0
      Self Powered
    MaxPower              100mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           3
      bInterfaceClass       255 Vendor Specific Class
      bInterfaceSubClass    255 Vendor Specific Subclass
      bInterfaceProtocol    255 Vendor Specific Protocol
      iInterface              0 
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x01  EP 1 OUT
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0200  1x 512 bytes
        bInterval               4
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0200  1x 512 bytes
        bInterval               4
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x82  EP 2 IN
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0200  1x 512 bytes
        bInterval               4
Device Qualifier (for other device speed):
  bLength                10
  bDescriptorType         6
  bcdUSB               2.00
  bDeviceClass          255 Vendor Specific Class
  bDeviceSubClass       255 Vendor Specific Subclass
  bDeviceProtocol       255 Vendor Specific Protocol
  bMaxPacketSize0        64
  bNumConfigurations      1
can't get debug descriptor: Resource temporarily unavailable
Device Status:     0x0001
  Self Powered
```
</details>

### 0803: I²Cバス(3.3V)

ピンの順序: SCL, SDA

デバイスアドレス（シフトなし）: 0x38, 0x3f

通常のブート時では興味深いことをしませんが、macOSでは何かします。

また、時々、START 00 STOP (ACKサイクルなし)を低速で送信するだけの場合もあります(?)。

### 0809: もう一つのI²Cバス(3.3V)

ピンの順序: SCL, SDA

デバイスアドレス（シフトなし）: 0x6b, 0x38, 0x3f, 

### TBD

* 0206 1.2Vへの弱い(30kΩ)プル、GNDへの反応なし、トランジションなし。SWDである可能性
* 0301 1.2V、1つのピンがハイにドライブ、もう1つはドライブしない。もう1つのUART？ DFUモードでHigh-z、Highピンが電源/ブートを追跡する以外の何も動きなし
* 0303 ピンセット1-2にのみマップ？GND？遷移なし。未使用のUARTモード？

