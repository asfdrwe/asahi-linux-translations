---
title: Apple Silicon ブートフロー
summary:
Apple Silicon 機器で使われるブートフロー、Soc 統合 ROM からユーザコードまで
---

2025/10/11時点の[boot](https://github.com/AsahiLinux/docs/blob/main/docs/fw/boot.md)の翻訳

---
Apple Siliconデバイスは現在のiOSデバイスと非常によく似たブートフローをたどるようです。

# Stage 0 (SecureROM)

このステージは、ブート[ROM](../project/glossary#r)内にあります。とりわけ、[NOR](../project/glossary.md#n)から通常のステージ1を検証、ロード、実行します。
失敗した場合は、[DFU](../project/glossary.md#d)にフォールバックし、[iBSS](../project/glossary.md#i)ローダーが送られてくるのを待ってから、ステージ1の[DFU](../project/glossary.md#d)のフローを続行します。

# 通常の流れ

## Stage 1 (LLB/iBoot)

このステージはオンボードの[NOR](../project/glossary.md#n)にあるプライマリアーリーローダー(primary early loader)です。このブートステージでは、非常に大まかに以下のような流れになります:

* [NVRAM](Glossary.md#n)から `boot-volume` 変数を読み込み: そのフォーマットは `<gpt-partition-type-uuid>:<gpt-partition-uuid>:<volume-group-uuid>` 。その他の関連する変数は`update-volume`と`upgrade-boot-volume`のようで、おそらく`boot-info-payload`変数内のメタデータによって選択される
* ローカルポリシーのハッシュを取得:
  - 最初にローカルで提案されているハッシュ（[SEP](../project/glossary.md#s) コマンド11）を試みる
  - それが利用できない場合は、ローカルのブレスドハッシュ(blessed hash)を取得 ([SEP](../progject/glossary.md#s)(コマンド14))
* iSCPreboot パーティションの `/<volume-group-uuid>/LocalPolicy/<policy-hash>.img4` にあるローカルブートポリシーを読む。
このブートポリシーには次のメタデータキー ([4CCs](../project/glossary.md#4)):
  - `vuid`: UUID: Volume group UUID - 上記と同じ
  - `kuid`: UUID: KEK group UUID
  - `lpnh`: SHA384: ローカルポリシーのナンスハッシュ(nonce hash)
  - `rpnh`: SHA384: リモートポリシーのナンスハッシュ
  - `ronh`: SHA384: リカバリ OS ポリシーナンスハッシュ
  - `nsih`: SHA384: 次のステージのIMG4ハッシュ
  - `coih`: SHA384: fuOS（カスタムkernelcache）IMG4ハッシュ
  - `auxp`: SHA384 ユーザーが許可した補助的なカーネル拡張のハッシュ
  - `auxi`: SHA384: 補助的なカーネルキャッシュ IMG4 のハッシュ
  - `auxr`: SHA384：補助的なカーネル拡張機能のレセプターのハッシュ
  - `prot`: SHA384: Paired Recovery manifestのハッシュ
  - `hrlp`: bool: Secure Enclave が署名するリカバリ OS ローカルポリシー
  - `lobo`: bool: ローカルブートポリシー
  - `love`: bool: ローカル OS バージョン
  - `smb0`: bool: Reduced security を有効化
  - `smb1`: bool: Permissive security を有効化
  - `smb2`: bool: サードパーティ製カーネル拡張を有効化
  - `smb3`: bool: 手動でモバイルデバイス管理（MDM）を登録
  - `smb4`: bool?: MDM デバイス登録プログラムを無効化
  - `sip0`: u16: カスタマイズされた SIP
  - `sip1`: bool: 署名付きシステムボリューム（`csrutil authenticated-root`）を無効化
  - `sip2`: bool: CTRR ([Configurable Text Region Read-only Region](https://keith.github.io/xcode-man-pages/bputil.1.html)) を無効化
  - `sip3`: bool: `boot-args` フィルタリングを無効化

  また、オプションとして、以下のリンクされたマニフェストが、それぞれ `/<volume-group-uuid>/LocalPolicy/<policy-hash>.<id>.im4m` に存在:
  - `auxk`: AuxKC (サードパーティのkext) manifest
  - `fuos`: fuOS (カスタムkernelcache) manifest

* 次のステージを読み込む場合:

  - ブートディレクトリはターゲットパーティションのPreboot subvolumeのパス `/<volume-uuid>/boot/<local-policy.metadata.nsih>` に存在
  - <boot-dir>/usr/standalone/firmware/iBoot.img4` を同じディレクトリにあるデバイス・ツリーや他のファームウェア・ファイルと一緒に復号、検証、実行。他のメタデータ記述子についてはまだ根拠なし

* カスタムステージ([fuOS](Glossary.md#f))を読み込む場合:

  - ...

この段階で失敗すると、エラーになるか、[DFU](../project/glossary.md#d)にフォールバックし、iBECローダーの送信を待ってから、[DFU](Glossary.md#d)の流れでステージ2に進みます。

## Stage 2 (iBoot2)

このステージはOSレベルのローダーで、OSパーティションの中にありmacOSの一部として出荷されています。システムの残りの部分をロードします。

# [DFU](../project/glossary.md#d)の流れ

## Stage 1 (iBSS)

このステージは『復元』ホストからデバイスに送られます。このステージでは、2つ目のステージであるiBECのブートストラップ、検証、実行が行われます。

## Stage 2 (iBEC)

# モード

起動すると、[AP](../project/glossary.md#a)は[SEP](../project/glossary.md#s)で確認できるようにブートモードのいずれかになります:

|  ID | Name                                      |
|----:|-------------------------------------------|
|   0 | macOS                                     |
|   1 | 1TR ("one true" recoveryOS)        |
|   2 | recoveryOS ("ordinary" recoveryOS) |
|   3 | kcOS                                      |
|   4 | restoreOS                                 |
| 255 | unknown                                   |

[SEP](../project/glossary.md#s)では、[1TR](../project/glossary.md#1)で特定のコマンド(ブートポリシーの編集など)の実行を許可していないと、エラー11『AP boot mode』で失敗します。
