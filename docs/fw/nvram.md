---
title: NVRAM
summary:
Apple Silicon Mac で使われる NVRAM 変数
---

2025/3/9時点の[nvram](https://github.com/AsahiLinux/docs/blob/main/docs/fw/nvram.md)の翻訳

---
# 種類

* `string`: 標準的な文字列
* `binary`: バイナリデータを含むURLエンコードされた文字列
* `boolean`: 真偽を表す文字列。`true` または `false` のいずれかの値を持つ文字列
* `int`: 10進の整数
* `bin-int(n)`: n バイトの `binary` エンコードされた整数（リトルエンディアン）
* `hex-int`: 16進の整数
* `volume`: `<gpt-partition-type-uuid>:<gpt-partition-uuid>:<volume-group-uuid>` 形式の文字列。GPTパーティションのUUIDは最初の3つのコンポーネントがバイトスワップされていて少し変

# 値

## ブート

* `auto-boot`: `boolean`: 自動的に起動するかどうかを指定。少なくとも Mac M1 mini ではこれを `false` にすると起動に失敗
* `boot-args`: `string`: カーネルに渡すブート引数。ブートポリシーによってフィルタリングされる可能性あり
* `boot-command`: `string`: 例: `fsboot`
* `boot-info-payload`: `binary`: 不透明で高エントロピーのペイロード(Some kind of opaque, high-entropy payload)
* `boot-note`: `binary`: 不明。例: `%00%00%00%00%00%00%00%bb%0ez%e5%00%00%00%00%a0q%d4%07%08%00%00%00`
* `boot-volume`: `volume`: デフォルトのブートボリューム
* `failboot-breadcrumbs`: `string`: ブートプログレスの様々な部分で生成されるスペースで区切られたコード。例: `3000c(706d7066) 3000d 30010 f0200 f0007(706d7066) 3000c(0) 3000d 40038(958000) 40039(1530000) 4003a(0) 3000f(64747265) 3000c(64747265) 40029 3000d 30010 3000f(69626474) 3000c(69626474) 40029 3000d 30010 3000f(69737973) 3000c(69737973) 3000d 30010 3000f(63737973) 3000c(63737973) 3000d 30010 3000f(62737463) 3000c(62737463) 3000d 30010 3000f(74727374) 3000c(74727374) 3000d 30010 3000f(66756f73) 40060004 30011 30007 <COMMIT> 401d000c <COMMIT> <BOOT> 1c002b(2006300) 3000f(0) 3000c(0) 3000d 30010 3000f(69626f74) 3000c(69626f74) 40040204 40040023 4003000e 30011 30007 401d000f(ffffffff) <COMMIT> `
* `nonce-seeds`: `binary`
* `panicmedic-timestamps`: `hex-int`: ナノ秒精度の UNIX タイムスタンプ。おそらく最後のパニック発生時
* `policy-nonce-digests`: `binary`
* `upgrade-boot-volume`: `volume`

## アップデート

* `IDInstallerDataV1`: `binary:lzvn:bin-plist`: 最新のインストーラの動作に関する情報を含む圧縮されたバイナリのplist。macOS 10.12 と 11.0間のどこかから欠落
* `IDInstallerDataV2`: `binary:lzvn:bin-plist`: `IDInstallerDataV1` と同形式の情報項目の配列を含む圧縮されたバイナリ plist
* `ota-updateType`: `string`: 適用するover-the-air アップデートの種類。例: `incremental`
* `update-volume`: `volume`

## ハードウェア

* `bluetoothActiveControllerInfo`: `binary`.
* `bluetoothInternalControllerInfo`: `binary`.
* `ota-controllerVersion`: `string`: over-the-air アップデートコントローラの識別子。例: `SUMacController-1.10` (Mac Mini M1), `SUS-1.0` (iPhone, iPad)
* `usbcfwflasherResult`: `string`: 例: `No errors`

## 設定

* `backlight-nits`: `hex-int`: 画面のバックライトの強さを指定。Mac Mini M1の例: `0x008c0000`
* `current-network`: `binary`: 直近で接続したWi-Fiネットワーク
* `fmm-computer-name`: `string`: コンピュータ名
* `good-samaritan-message`: `string`: デバイスを紛失したときに起動/パスワード画面に表示するメッセージ
* `preferred-networks`: `binary`: 保存されているWi-Fiネットワークのリスト
* `preferred-count`: `int`: `preferred-networks` に含まれるネットワークの数（1ではない場合）
* `prev-lang:kbd`: `string`: キーボードレイアウト。フォーマット: `<lang>:<locale-id>`, [参照](https://github.com/acidanthera/OpenCorePkg/blob/master/Utilities/AppleKeyboardLayouts/AppleKeyboardLayouts.txt)。例: `en-GB:26`
* `prev-lang-diags:kbd`: `string`: 診断時のキーボードレイアウト。例: `en-GB`
* `SystemAudioVolume`: `bin-int(8)`: ボリューム。例: `%80` (128)
* `SystemAudioVolumeExtension`: `bin-int(16)`: 音量。例: `%ff%7f` (32767)

## Misc

* `_kdp_ipstr`: `string`: 現在割り当てられているIPv4
* `lts-persistance`: `binary`

# 例

## `IDInstallerDataV2`

<details>
<summary>Big Sur 11.2 beta 1 (20D5029f) にアップグレードが成功した例</summary>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<array>
	<dict>
		<key>505</key>
		<string>auth not needed</string>
		<key>6</key>
		<string>key recovery assistant</string>
	</dict>
	<dict>
		<key>505</key>
		<string>auth not needed</string>
		<key>6</key>
		<string>key recovery assistant</string>
	</dict>
	<dict>
		<key>0</key>
		<string>20D5029f</string>
		<key>100</key>
		<string>passed</string>
		<key>6</key>
		<string>upgrade</string>
	</dict>
	<dict>
		<key>505</key>
		<string>auth not needed</string>
		<key>6</key>
		<string>key recovery assistant</string>
	</dict>
	<dict>
		<key>505</key>
		<string>auth not needed</string>
		<key>6</key>
		<string>key recovery assistant</string>
	</dict>
	<dict>
		<key>505</key>
		<string>auth not needed</string>
		<key>6</key>
		<string>key recovery assistant</string>
	</dict>
	<dict>
		<key>6</key>
		<string>key recovery assistant</string>
	</dict>
	<dict>
		<key>6</key>
		<string>key recovery assistant</string>
	</dict>
</array>
</plist>
```

</details>

<details>
  <summary>アップグレードがクラッシュした例</summary>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<array>
	<dict>
		<key>100</key>
		<string>crashed</string>
		<key>102</key>
		<string>initializer</string>
		<key>103</key>
		<string>1</string>
		<key>7</key>
		<string>NO</string>
	</dict>
</array>
</plist>
```

</details>

<details>
  <summary>Sierra 10.12.2 (16C67)にアップグレードが成功した例</summary>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<array>
	<dict>
		<key>0</key>
		<string>16C67</string>
		<key>100</key>
		<string>passed</string>
		<key>103</key>
		<string>1</string>
		<key>202</key>
		<string>832.499040</string>
		<key>203</key>
		<string>41.700535</string>
		<key>205</key>
		<string>30.318743</string>
		<key>206</key>
		<string>0.003648</string>
		<key>207</key>
		<string>0.156793</string>
		<key>208</key>
		<string>2.215885</string>
		<key>209</key>
		<string>8.130921</string>
		<key>299</key>
		<string>0.212016</string>
		<key>3</key>
		<string>solid state</string>
		<key>4</key>
		<string>unencrypted</string>
		<key>5</key>
		<string>case sensitive</string>
		<key>6</key>
		<string>clean</string>
		<key>7</key>
		<string>NO</string>
	</dict>
	<dict>
		<key>0</key>
		<string>16C67</string>
		<key>100</key>
		<string>passed</string>
		<key>103</key>
		<string>2</string>
		<key>202</key>
		<string>802.017327</string>
		<key>203</key>
		<string>29.902674</string>
		<key>205</key>
		<string>4.379149</string>
		<key>206</key>
		<string>0.003310</string>
		<key>207</key>
		<string>0.156726</string>
		<key>208</key>
		<string>2.214545</string>
		<key>209</key>
		<string>10.050913</string>
		<key>299</key>
		<string>0.184676</string>
		<key>3</key>
		<string>solid state</string>
		<key>4</key>
		<string>unencrypted</string>
		<key>5</key>
		<string>case insensitive</string>
		<key>6</key>
		<string>clean</string>
		<key>7</key>
		<string>NO</string>
	</dict>
	<dict>
		<key>0</key>
		<string>16C67</string>
		<key>100</key>
		<string>passed</string>
		<key>103</key>
		<string>3</string>
		<key>6</key>
		<string>software update</string>
	</dict>
	<dict>
		<key>0</key>
		<string>16C67</string>
		<key>100</key>
		<string>passed</string>
		<key>103</key>
		<string>4</string>
		<key>202</key>
		<string>582.532387</string>
		<key>203</key>
		<string>11.511343</string>
		<key>205</key>
		<string>1.900536</string>
		<key>206</key>
		<string>0.005585</string>
		<key>207</key>
		<string>0.101757</string>
		<key>208</key>
		<string>2.142859</string>
		<key>209</key>
		<string>3.942741</string>
		<key>299</key>
		<string>0.122528</string>
		<key>3</key>
		<string>solid state</string>
		<key>4</key>
		<string>unencrypted</string>
		<key>5</key>
		<string>case insensitive</string>
		<key>6</key>
		<string>clean</string>
		<key>7</key>
		<string>YES</string>
	</dict>
</array>
</plist>
```

</details>
