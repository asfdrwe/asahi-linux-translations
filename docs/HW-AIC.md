2025/3/1時点の[HW-AIC](https://github.com/AsahiLinux/docs/blob/main/docs/HW-AIC.md)の翻訳

---
AICはApple Interrupt Controllerです。これらは色々雑多なリバースエンジニアリングノートです。

Appleは特定のSET/CLRレジスタペアスタイルを好んで使用します:

* SET：現在の状態を読み取り、セットビットに1を書き込む(reads current state, writes set bits set to 1)
* CLR: 現在の状態を読み取り、クリアビットに1を書き込む(reads current state, writes clear bits set to 1)

## レジスタ

```
0000~ グローバルなもの
  0004: NR_IRQ?
  0010: GLOBAL_CFG? (impl bits: f8fffff1)
2000~ 割り込み acks, IPIs等

3000~ IRQ_TGT (各レジスタに1つ, 各レジスタにCPUのビットフィールド)
4000~ SW_GEN_SET (ビットフィールド)
4080~ SW_GEN_CLR (ビットフィールド)
4100~ IRQ_MASK_SET (ビットフィールド)
4180~ IRQ_MASK_CLR
4200~ HW_IRQ_MON (現在の割り込みライン状態?)

8020 MSR CNTPCT_EL0の下位32bits (システムタイマー)
8028 MSR CNTPCT_EL0の上位32bits (システムタイマー)

現在のCPUコアのコア毎の状態のミラーアクセス:
2004 IRQ_REASON
2008 IPI_SEND - IPIを送信、ビット0から31未満は他のIPIをCPUに送信、ビット31は『自分自身』のIPIをこのCPUに送信
200c IPI_ACK - IPIを取得、ビット0は『他』のIPIを取得、ビット31は 『自分自身』のIPIを取得
2024 IPI_MASK_SET - IPI用のマスクビットはIPI_ACK用のビットと同じタイプと位置に対応
2028 IPI_MASK_CLR

TODO コア毎のステートオフセットへの直接アクセスの文書化
```

## 使い方

IPIの流れ:

* IPI_SENDへのビットを書き込む
* ARM IRQをアサート
* IRQ_REASONを読み込む
    * IPIはIPI_MASKでマスク
    * ARM IRQをアサート解除
* IPI_ACKにビットを書き込む
* IPI_MASK_CLRにビットを書き込む
    * IPIをマスク解除
    * IPI_ACKがクリアされなかった場合ARM IRQはここで再アサート

HW irqの流れ:

* IRQ_TGTのターゲットビットフィールドを設定
* IRQ_MASK_CLRにビットを書き込み
* (後で) HW IRQをアサート
* IRQ_REASONを読み込む
    * IRQ_MASK は自動的に設定
    * ARM IRQをアサート解除
* (特定のハードウェアではIRQのクリア)
* IRQ_MASK_CLRにビットを書き込む
    * IRQはマスクされていない
    * ハードウェアラインがクリアされなかった場合、ARM irqはここで再度アサート

ターゲットは11個？CPU 0-7と補助的なもの？
        
SW_GENで設定されたビットはハードウェアIRQラインとORされます。

## タイマー

システムタイマーはARM64の標準的なMSRのものでAICをバイパスします。FIQ に直接配線されています。
