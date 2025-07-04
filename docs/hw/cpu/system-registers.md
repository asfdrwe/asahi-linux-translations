---
title: システムレジスタ
---

2025/6/30時点の[System Registers](https://github.com/AsahiLinux/docs/blob/main/docs/hw/cpu/system-registers.md)の翻訳

---
網羅的な列挙と調査は[システムレジスタダンプ](system-registers-dumps.md)を参照してください。

## 用語集

以下推測

* ACC: Apple Core Cluster (アップルコアクラスタ)
* HID: Hardware Implementation Defined Register (ハードウェア実装定義レジスタ)
* EHID: Hardware Implementation Defined Register (e-core) (ハードウェア実装定義レジスタ(効率コア))
* IPI: Inter-processor Interrupt (プロセッサ間割り込み)

## レジスタの定義

Linuxのフォーマットを使用:

```c

/* These make sense... */
#define SYS_APL_HID0_EL1            sys_reg(3, 0, 15, 0, 0)
#define SYS_APL_EHID0_EL1           sys_reg(3, 0, 15, 0, 1)
#define SYS_APL_HID1_EL1            sys_reg(3, 0, 15, 1, 0)
#define SYS_APL_EHID1_EL1           sys_reg(3, 0, 15, 1, 1)
#define SYS_APL_HID2_EL1            sys_reg(3, 0, 15, 2, 0)
#define SYS_APL_EHID2_EL1           sys_reg(3, 0, 15, 2, 1)
#define SYS_APL_HID3_EL1            sys_reg(3, 0, 15, 3, 0)
#define SYS_APL_EHID3_EL1           sys_reg(3, 0, 15, 3, 1)
#define SYS_APL_HID4_EL1            sys_reg(3, 0, 15, 4, 0)
#define SYS_APL_EHID4_EL1           sys_reg(3, 0, 15, 4, 1)
#define SYS_APL_HID5_EL1            sys_reg(3, 0, 15, 5, 0)
#define SYS_APL_EHID5_EL1           sys_reg(3, 0, 15, 5, 1)
#define SYS_APL_HID6_EL1            sys_reg(3, 0, 15, 6, 0)
#define SYS_APL_HID7_EL1            sys_reg(3, 0, 15, 7, 0)
#define SYS_APL_EHID7_EL1           sys_reg(3, 0, 15, 7, 1)
#define SYS_APL_HID8_EL1            sys_reg(3, 0, 15, 8, 0)
#define SYS_APL_HID9_EL1            sys_reg(3, 0, 15, 9, 0)
#define SYS_APL_EHID9_EL1           sys_reg(3, 0, 15, 9, 1)
#define SYS_APL_HID10_EL1           sys_reg(3, 0, 15, 10, 0)
#define SYS_APL_EHID10_EL1          sys_reg(3, 0, 15, 10, 1)
#define SYS_APL_HID11_EL1           sys_reg(3, 0, 15, 11, 0)
#define SYS_APL_EHID11_EL1          sys_reg(3, 0, 15, 11, 1)

/* Uh oh */
#define SYS_APL_HID12_EL?           sys_reg(3, 0, 15, 12, 0)
#define SYS_APL_HID13_EL?           sys_reg(3, 0, 15, 14, 0)
#define SYS_APL_HID14_EL?           sys_reg(3, 0, 15, 15, 0)

/* All sanity went out the window here */
#define SYS_APL_HID16_EL?           sys_reg(3, 0, 15, 15, 2)
#define SYS_APL_HID17_EL1           sys_reg(3, 0, 15, 15, 5)
#define SYS_APL_HID18_EL?           sys_reg(3, 0, 15, 11, 2)
#define SYS_APL_EHID20_EL1          sys_reg(3, 0, 15, 1, 2)
#define SYS_APL_HID21_EL?           sys_reg(3, 0, 15, 1, 3)

#define SYS_APL_PMCR0_EL1           sys_reg(3, 1, 15, 0, 0)
#define SYS_APL_PMCR1_EL1           sys_reg(3, 1, 15, 1, 0)
#define SYS_APL_PMCR2_EL1           sys_reg(3, 1, 15, 2, 0)
#define SYS_APL_PMCR3_EL1           sys_reg(3, 1, 15, 3, 0)
#define SYS_APL_PMCR4_EL1           sys_reg(3, 1, 15, 4, 0)
#define SYS_APL_PMESR0_EL1          sys_reg(3, 1, 15, 5, 0)
#define SYS_APL_PMESR1_EL1          sys_reg(3, 1, 15, 6, 0)
#define SYS_APL_PMSR_EL1            sys_reg(3, 1, 15, 13, 0)

#define SYS_APL_PMC0_EL1            sys_reg(3, 2, 15, 0, 0)
#define SYS_APL_PMC1_EL1            sys_reg(3, 2, 15, 1, 0)
#define SYS_APL_PMC2_EL1            sys_reg(3, 2, 15, 2, 0)
#define SYS_APL_PMC3_EL1            sys_reg(3, 2, 15, 3, 0)
#define SYS_APL_PMC4_EL1            sys_reg(3, 2, 15, 4, 0)
#define SYS_APL_PMC5_EL1            sys_reg(3, 2, 15, 5, 0)
#define SYS_APL_PMC6_EL1            sys_reg(3, 2, 15, 6, 0)
#define SYS_APL_PMC7_EL1            sys_reg(3, 2, 15, 7, 0)
#define SYS_APL_PMC8_EL1            sys_reg(3, 2, 15, 9, 0)
#define SYS_APL_PMC9_EL1            sys_reg(3, 2, 15, 10, 0)

#define SYS_APL_LSU_ERR_STS_EL1     sys_reg(3, 3, 15, 0, 0)
#define SYS_APL_E_LSU_ERR_STS_EL1   sys_reg(3, 3, 15, 2, 0)
#define SYS_APL_LSU_ERR_CTL_EL1     sys_reg(3, 3, 15, 1, 0)

#define SYS_APL_L2C_ERR_STS_EL1     sys_reg(3, 3, 15, 8, 0)
#define SYS_APL_L2C_ERR_ADR_EL1     sys_reg(3, 3, 15, 9, 0)
#define SYS_APL_L2C_ERR_INF_EL1     sys_reg(3, 3, 15, 10, 0)

#define SYS_APL_FED_ERR_STS_EL1     sys_reg(3, 4, 15, 0, 0)
#define SYS_APL_E_FED_ERR_STS_EL1   sys_reg(3, 4, 15, 0, 2)

#define SYS_APL_APCTL_EL1           sys_reg(3, 4, 15, 0, 4)
#define SYS_APL_KERNELKEYLO_EL1     sys_reg(3, 4, 15, 1, 0)
#define SYS_APL_KERNELKEYHI_EL1     sys_reg(3, 4, 15, 1, 1)

#define SYS_APL_VMSA_LOCK_EL1       sys_reg(3, 4, 15, 1, 2)

#define SYS_APL_APRR_EL0            sys_reg(3, 4, 15, 2, 0)
#define SYS_APL_APRR_EL1            sys_reg(3, 4, 15, 2, 1)

#define SYS_APL_CTRR_LOCK_EL1       sys_reg(3, 4, 15, 2, 2)
#define SYS_APL_CTRR_A_LWR_EL1      sys_reg(3, 4, 15, 2, 3)
#define SYS_APL_CTRR_A_UPR_EL1      sys_reg(3, 4, 15, 2, 4)
#define SYS_APL_CTRR_CTL_EL1        sys_reg(3, 4, 15, 2, 5)

#define SYS_APL_APRR_JIT_ENABLE_EL2 sys_reg(3, 4, 15, 2, 6)
#define SYS_APL_APRR_JIT_MASK_EL2   sys_reg(3, 4, 15, 2, 7)

#define SYS_APL_s3_4_c15_c5_0_EL1   sys_reg(3, 4, 15, 5, 0)

#define SYS_APL_CTRR_LOCK_EL2       sys_reg(3, 4, 15, 11, 5)
#define SYS_APL_CTRR_A_LWR_EL2      sys_reg(3, 4, 15, 11, 0)
#define SYS_APL_CTRR_A_UPR_EL2      sys_reg(3, 4, 15, 11, 1)
#define SYS_APL_CTRR_CTL_EL2        sys_reg(3, 4, 15, 11, 4)

#define SYS_APL_IPI_RR_LOCAL_EL1    sys_reg(3, 5, 15, 0, 0)
#define SYS_APL_IPI_RR_GLOBAL_EL1   sys_reg(3, 5, 15, 0, 1)

#define SYS_APL_DPC_ERR_STS_EL1     sys_reg(3, 5, 15, 0, 5)

#define SYS_APL_IPI_SR_EL1          sys_reg(3, 5, 15, 1, 1)

#define SYS_APL_VM_TMR_LR_EL2       sys_reg(3, 5, 15, 1, 2)
#define SYS_APL_VM_TMR_FIQ_ENA_EL2  sys_reg(3, 5, 15, 1, 3)

#define SYS_APL_IPI_CR_EL1          sys_reg(3, 5, 15, 3, 1)

#define SYS_APL_ACC_CFG_EL1         sys_reg(3, 5, 15, 4, 0)
#define SYS_APL_CYC_OVRD_EL1        sys_reg(3, 5, 15, 5, 0)
#define SYS_APL_ACC_OVRD_EL1        sys_reg(3, 5, 15, 6, 0)
#define SYS_APL_ACC_EBLK_OVRD_EL?   sys_reg(3, 5, 15, 6, 1)

#define SYS_APL_MMU_ERR_STS_EL1     sys_reg(3, 6, 15, 0, 0)

#define SYS_APL_E_MMU_ERR_STS_EL1   sys_reg(3, 6, 15, 2, 0)

#define SYS_APL_AFPCR_EL0           sys_reg(3, 6, 15, 2, 5)

#define SYS_APL_APSTS_EL1           sys_reg(3, 6, 15, 12, 4)

#define SYS_APL_UPMCR0_EL1          sys_reg(3, 7, 15, 0, 4)
#define SYS_APL_UPMESR0_EL1         sys_reg(3, 7, 15, 1, 4)
#define SYS_APL_UPMECM0_EL1         sys_reg(3, 7, 15, 3, 4)
#define SYS_APL_UPMECM1_EL1         sys_reg(3, 7, 15, 4, 4)
#define SYS_APL_UPMPCM_EL1          sys_reg(3, 7, 15, 5, 4)
#define SYS_APL_UPMSR_EL1           sys_reg(3, 7, 15, 6, 4)
#define SYS_APL_UPMECM2_EL1         sys_reg(3, 7, 15, 8, 5)
#define SYS_APL_UPMECM3_EL1         sys_reg(3, 7, 15, 9, 5)
#define SYS_APL_UPMESR1_EL1         sys_reg(3, 7, 15, 11, 5)

/* Note: out of order wrt above */
#define SYS_APL_UPMC0_EL1           sys_reg(3, 7, 15, 7, 4)
#define SYS_APL_UPMC1_EL1           sys_reg(3, 7, 15, 8, 4)
#define SYS_APL_UPMC2_EL1           sys_reg(3, 7, 15, 9, 4)
#define SYS_APL_UPMC3_EL1           sys_reg(3, 7, 15, 10, 4)
#define SYS_APL_UPMC4_EL1           sys_reg(3, 7, 15, 11, 4)
#define SYS_APL_UPMC5_EL1           sys_reg(3, 7, 15, 12, 4)
#define SYS_APL_UPMC6_EL1           sys_reg(3, 7, 15, 13, 4)
#define SYS_APL_UPMC7_EL1           sys_reg(3, 7, 15, 14, 4)
#define SYS_APL_UPMC8_EL1           sys_reg(3, 7, 15, 0, 5)
#define SYS_APL_UPMC9_EL1           sys_reg(3, 7, 15, 1, 5)
#define SYS_APL_UPMC10_EL1          sys_reg(3, 7, 15, 2, 5)
#define SYS_APL_UPMC11_EL1          sys_reg(3, 7, 15, 3, 5)
#define SYS_APL_UPMC12_EL1          sys_reg(3, 7, 15, 4, 5)
#define SYS_APL_UPMC13_EL1          sys_reg(3, 7, 15, 5, 5)
#define SYS_APL_UPMC14_EL1          sys_reg(3, 7, 15, 6, 5)
#define SYS_APL_UPMC15_EL1          sys_reg(3, 7, 15, 7, 5)
```

### HIDレジスター

この命名規則はPowerPCに由来するものと思われます。多くのchiken bitsがここに配置されているようです。

これらは主にCPUの機能を無効にするためのchiken bitsで、特定のCPU世代にしか適用されないものが多いようです。しかしその定義はグローバルです。

#### SYS_APL_HID0_EL1

* [20] ループバッファを無効化
* [21] AMX Cache Fusionを無効化
* [25] ICプリフェッチ制限1 『Brn』
* [28] フェッチ幅を無効化
* [33] PMULL Fuseを無効化
* [36] Cache Fusionを無効化
* [45] いくつかのPg (ページ？) 電力最適化
* [62:60] 命令キャッシュのプリフェッチ深度

#### SYS_APL_EHID0_EL1

* [45] nfpRetFwdDisb

#### SYS_APL_HID1_EL1

* [14] CMP-Branch Fusionを無効化
* [15] ForceMextL3ClkOn
* [23] rccForceAllIexL3ClksOn
* [24] rccDisStallInactiveIexCtl
* [25] disLspFlushWithContextSwitch
* [44] グループ間AES Fusionを無効化
* [49] MSR Speculation DAIFを無効化
* [54] SMCをトラップ
* [58] enMDSBStallPipeLineECO
* [60] Branch Kill Limitを有効化 / SpareBit6

#### SYS_APL_EHID1_EL1

* [30] MSR Speculation DAIF を無効化

#### SYS_APL_HID2_EL1

* [13] MMUのMTLBプリフェッチを無効化
* [17] MTBを強制パージ

#### SYS_APL_EHID2_EL1

* [17] MTBを強制パージ

#### SYS_APL_HID3_EL1

* [2] 色最適化を無効化
* [25] DC ZVAコマンドのみ無効化
* [44] Arbiter Fix BIF CRD を無効化
* [54] Xmon Snp Evict Trigger L2 Starvation Mode を無効化
* [63] Pcie Throttleを有効化

#### SYS_APL_EHID3_EL1

* [2] 色最適化を無効化
* [25] DC ZVAコマンドのみ無効化

#### SYS_APL_HID4_EL1

* [1] STNT Widgetを無効化
* [9] Speculative LS Redirectを無効化
* [11] DC MVA Opsを無効化
* [33] Speculative Lnch Readを無効化
* [39] Ns Ord Ld Req No Older Ldを強制 (非投機的な順番通りのLoadはその命令以前のLoadを必要としない？)
* [41:40] Cnfカウンタ閾値
* [44] DC SW L2 Opsを無効化
* [49] Lfsr Stall Load Pipe 2 Issue を有効化
* [53] Lfsr Stall Stq Replay を有効化

#### SYS_APL_HID5_EL1

* [15:14] Crd Edb Snp Rsvd
* [44] HWP Loadを無効化
* [45] HWP Storeを無効化
* [54] Dn FIFO Read Stallを有効化
* [57] Full Line Writeを無効化
* [61] Fill 2C Mergeを無効化

#### SYS_APL_EHID5_EL1

* [35] Fill Bypassを無効化

#### SYS_APL_HID6_EL1

* [9:5] Up Crd Tkn Init C2
* [55] ClkDiv Gatingを無効化

#### SYS_APL_HID7_EL1

* [7] Cross Pick2を無効化
* [10] Nex Fast FMULを無効化
* [16] Spec Flush PtrがInvalidかつMPがValid時に非投機実行(Non Speculative)を強制
* [20] Stepping時非投機実行(Non Speculative)を強制
* [25:24] Non Speculative Target Timer Selを強制

#### SYS_APL_HID8_EL1

* [7:4] DataSetID0
* [11:8] DataSetID1
* [35] Wkeが厳密順番を強制(Wke Force Strict Order)
* [59:56] DataSetID2
* [63:60] DataSetID3

#### SYS_APL_HID9_EL1

* [16] TSO を有効化
* [26] TSO がDC ZVA WCを許可
* [29] TSO がVLD Microopsをシリアライズ
* [48] EnableFixBug51667805
* [48] EnableFixBug51667805 
* [49] EnableFixBug51667717
* [50] EnableFixBug57817908
* [52] 非アラインのSTNT (Store Non-Temporal?) Widgetを無効化
* [53] EnableFixBug58566122
* [54] EnableFixBug47221499
* [55] HidEnFix55719865

#### SYS_APL_EHID9_EL1

* [5] Dev Throttle 2を有効化

#### SYS_APL_HID10_EL1
* [0] Hwp Gups を無効化

#### SYS_APL_EHID10_EL1

* [19] RCC Disable Power Save Prf (performance?) Clock Off
* [32] Drain UC状態を強制待機(Force Wait State Drain UC)
* [49] ZVAの一時的なTSOの無効化

#### SYS_APL_HID11_EL1

* [1] X64 NT Lnch Optimization を無効化
* [7] Fill C1 Bub(ble?)の最適化を無効化
* [15] HIDを有効化しUC 55719865を修正
* [23] Fast Drain 最適化を無効化
* [59] LDNT(Load Non-Temporal?) Widgetを無効化

#### SYS_APL_EHID11_EL1

* [41:40] SMB Drain閾値

#### SYS_APL_HID13_EL1

* [17:14] PreCyc
* [63:60] Cycleカウントをリセット

#### SYS_APL_HID14_EL1

* [?:0] Nex Sleep Timeout Cyclone
* [32] Nex Power Gatingを有効化

#### SYS_APL_HID16_EL1

* [18] LEQ Throttle Aggr
* [56] SpareBit0
* [57] RS4 Secを有効化
* [59] SpareBit3
* [60] xPick RS 45 を無効化
* [61] MPx Pick 45 を有効化
* [62] MP Cyclone 7を有効化
* [63] SpareBit7

#### SYS_APL_HID17_EL1

* [2:0] Crd Edb Snp Rsvd

#### SYS_APL_HID18_EL1

* [14] HVC Speculationを無効化
* [49] SpareBit17

#### SYS_APL_EHID20_EL1

* [8] SMCをトラップ
* [15] 最古のRedirとそれ以降がValid有効の場合に非投機実行(Nonspeculation)を強制
* [16] Spec Flush Pointer != Blk Rtr Pointer の場合に非投機実行(Nonspeculation)を強制
* [22:21] Nonspeculation Targeted Timerを強制

#### SYS_APL_HID21_EL1

* [19] LDREX Fill Replyを有効化
* [33] LDQ RTRが古いLST Rel Cmplを待機
* [34] Cdp Reply Purged Trans を無効化
* [52] すべての SPR SYNC に対して MMU をパージ 

### ACC/CYC レジスタ

これらはコアコンプレックスと電源管理の設定に関連しているようです。

#### SYS_APL_ACC_OVRD_EL1

* [14:13] Power Down SRMが可能 (3=deepsleep)
* [16:15] ACCスリープ時のL2フラッシュを無効化 (2=deepsleep)
* [18:17] Train Down Linkが可能 (3=deepsleep)
* [26:25] Power Down CPMが可能 (2=deny 3=deepsleep)
* [28:27] CPM Wakeup (3=force)
* [29] Clock DTRを無効化
* [32] WFI CPUのPIOを無効化
* [34] deepsleepを有効化

#### SYS_APL_ACC_CFG_EL1

ACCスリープ時に分岐予測器の状態保持

* [3:2] BP Sleep (2=BDP, 3=BTP)

#### SYS_APL_CYC_OVRD_EL1

* [0] WFI Returnを無効化
* [25:24] Power Downが可能 (2=force up、3=force down)
* [21:20] FIQ mode (2=disable)
* [23:22] IRQ mode (2=disable)

### メモリサブシステムレジスタ

主にエラー制御？

#### SYS_APL_LSU_ERR_STS_EL1

* [54] L1 DTLBマルチヒットを有効化

#### SYS_APL_LSU_ERR_CTL_EL1

* [3] L1 DTLB マルチヒットを有効化

#### SYS_APL_L2C_ERR_STS_EL1

L2サブシステムのフォールトコントロールと情報。このレジスタはクラスタレベルでクラスタ内の全コアで共有。

* [1] Recursive fault(再帰的フォールト) (別のフォールト保留時にフォール発生？)
* [7] Access fault(アクセスフォルト) (アンマップされた物理アドレスなど)
* [38..34] フラグを有効化? (iBootからの入力はすべて1)
* [39] SError interruptsを有効化（非同期エラー）
* [43..40] フラグを有効化? (iBootからの入力はすべて1)
* [56] フォールトフラグのwrite-1-to-clear動作を有効化
* [60] 何かを有効化? (エントリー時1)

#### SYS_APL_L2C_ERR_ADR_EL1

L2サブシステムのフォールトに対するフォールトアドレス。

* [?:0] フォールトの物理アドレス
* [42] ? 再帰的な命令フェッチフォールト後に時々1
* [57:55] 5=データ書き込み 6=データ読み込み 7=命令フェッチ?
* [62..61] フォールトの原因となったクラスタ内のコア

#### SYS_APL_L2C_ERR_INF_EL1

L2サブシステムのエラー情報。

下位ビットの値:

書込み:

* 1: マップされていない領域または保護された領域への書き込みまたはPCIe BAR領域へのnGnRnE書き込み
* 2: 32ビットのみのペリフェラルへの8ビットまたは16ビットの書き込み
* 3：SoC I/O空間へのnGnRE書き込み

読み込み:

* 1: マップされていない空間または保護された空間からの読み込み

上位ビット:

* [26] アドレスアラインメントに関係する何か (addr 4 mod 8に対する16ビットおよび32ビットの読み込み/書き込みで見られる)

### CTRR レジスタ

設定可能なテキスト読み取り専用領域。

#### SYS_APL_CTRR_CTL_EL1

* [0] A MMU ライトプロテクトをオフ
* [1] A MMU ライトプロテクトをオン
* [2] B MMU ライトプロテクトをオフ
* [3] B MMU ライトプロテクトをオン
* [4] A PXN
* [5] B PXN
* [6] A UXN
* [7] B UXN

### APRR レジスター

#### SYS_APL_APRR_EL0 / SYS_APL_APRR_EL1

これはテーブル。値は4ビットのフィールド:

* [0] X
* [1] W
* [2] R

このインデックスはPTEのアクセス保護と実行保護の設定。

* [0] XN
* [1] PXN
* [2] AP[0]
* [3] AP[1]

レジスタ値は16個の4ビットフィールドで自然な順((_rwx) << (4*prot))。

### IPI レジスタ

AICを使用しない『高速』なIPIに使用。

#### SYS_APL_IPI_RR_LOCAL_EL1

* [3:0] 対象CPU
* [29:28] RR Type(0=immediate、1=retract、2=deferred、3=nowake)

#### SYS_APL_IPI_RR_GLOBAL_EL1

* [3:0] 対象CPU
* [20:16] 対象クラスタ
* [29:28] RRタイプ (0=immediate, 1=retract, 2=deferred, 3=nowake)

#### SYS_APL_IPI_CR_EL1

グローバルレジスタ。

* [15:0] Deferred IPI countdown値(REFCLK ticks単位)

#### SYS_APL_VM_TMR_LR_EL2

(名称は非公式)

GICのICH_LR<n>_EL2に類似。ゲストのCNTVが起動すると状態がpending(63:62 == 1)、SYS_APL_HV_TMR_MASKではマスクされず、HACR_EL2ではマスク。

#### SYS_APL_VM_TMR_FIQ_ENA_EL2

(名称非公式)

* [0] CNTV guest timer mask bit (1=enable FIQ, 0=disable FIQ)
* [1] CNTP guest timer mask bit (1=enable FIQ, 0=disable FIQ)

#### SYS_APL_IPI_SR_EL1

ステータスレジスタ

* [0] IPI pending (1を書き込むとクリア)

IPI処理との競合を避けるためクリア後にバリアー（ISB SY）が必要。

### 仮想メモリシステムアーキテクチャのロック

#### SYS_APL_VMSA_LOCK_EL1

* [0] VBARをロック
* [1] SCTLRをロック
* [2] TCRをロック
* [3] TTBR0をロック
* [4] TTBR1をロック
* [63] SCTLRをロック M bit

起動時にセキュリティ上の理由で一部のArmレジスタへの書き込みをロックするために使用されます。

### ポインタ認証関連レジスタ

#### SYS_APL_APCTL_EL1

* [0] Apple Mode
* [1] Kernel Keyを有効化
* [2] AP Key 0を有効化
* [3] AP Key 1を有効化
* [4] User Keyを有効化

#### SYS_APL_APSTS_EL1

* [0] M Key Valid

### パフォーマンスカウンタレジスタ

コントロールレジスタへの書き込みが有効になるには`isb`が必要。

#### SYS_APL_PMC0-9_EL1

パフォーマンスカウンタ。

M1: 48ビット、 ビット47がPMIのトリガー。 M2: 64ビット、ビット63がPMIのトリガー。

* PMC #0: 固定CPUサイクルカウント (有効時)
* PMC #1: 固定命令カウント(有効時)

#### SYS_APL_PMCR0_EL1

* [7:0] PMC #7-0のカウンタを有効化
* [10:8] Interrupt mode (0=off 1=PMI 2=AIC 3=HALT 4=FIQ)
* [11] PMI interrupt 有効 (0を書き込むとクリア)
* [19:12] PMC#7-0のPMIを有効化
* [20] PMIのカウントを無効化
* [22] eret終了するまでPMIをブロック
* [23] グローバル（コアのみでない）L2Cイベントをカウント
* [30] ユーザーモードでのレジスタへのアクセスを許可
* [33:32] PMC#9-8のPMIを有効化
* [45:44] PMC#9-8のPMIの有効化

#### SYS_APL_PMCR1_EL1

どのELxモードがイベントをカウントするかを制御。

* [7:0] EL0 A32 が PMC #0-7 を有効化　(最近のチップでは未実装)
* [15:8] EL0 A64 が PMC #0-7　を有効化
* [23:16] EL1 A64 が PMC #0-7 を有効化
* [31:24] EL3 A64 が PMC #0-7 を有効化 (EL3を搭載した古いチップを除き未実装)
* [33:32] EL0 A32 が PMC #9-8を有効化 (最新のチップでは未実装)
* [41:40] EL0 A64 が PMC #9-8を有効化
* [49:48] EL1 A64 が PMC #9-8を有効化
* [57:56] EL3 A64 が PMC #9-8 を有効化 (最新のチップでは未実装)

#### SYS_APL_PMCR2_EL1

watchpoint registersを制御。

#### SYS_APL_PMCR4_EL1

breakpointsとaddress matchingを制御。

#### SYS_APL_PMCR4_EL1

opcode matchingを制御。

#### SYS_APL_PMSR_EL1

* [9:0] PMC #9-0でオーバーフローを検出
#### SYS_APL_PMESR0_EL1

PMC #2-5のイベント選択レジスタ

* [7:0] PMC #2用イベント
* [15:8] PMC #3用イベント
* [23:16] PMC #4用イベント
* [31:24] PMC #5用イベント

#### SYS_APL_PMESR1_EL1

PMC #6-9のイベント選択レジスタ

* [7:0] PMC #6 用イベント
* [15:8] PMC #7 用イベント
* [23:16] PMC #8 用イベント
* [31:24] PMC #9 用のイベント

#### SYS_APL_UPMCx_EL1

PMCsをUncore。48ビット、ビット47はオーバーフロービットで，PMIをトリガ。

#### SYS_APL_UPMCR0_EL1

* [15:0] カウンタ#15-0のカウンタを有効化
* [18:16] Interrupt mode (0=off 2=AIC 3=HALT 4=FIQ)
* [35:20] カウンタ#15-0のPMIを有効化

#### SYS_APL_UPMSR_EL1

* [0] PMIをUncore
* [1] CTI
* [17:2] uncoreカウンタ#15-0がオーバーフロー

#### SYS_APL_UPMPCM_EL1

* [7:0] uncore PMIsのためのPMI core mask - どのコアにPMIが配信されるか

#### SYS_APL_UPMESR0_EL1

イベント選択レジスタ。

#### SYS_APL_UPMESR1_EL1

イベント選択レジスタ。

#### SYS_APL_UPMECM[0-3]_EL1

クラスタ内の各イベントにコアマスクを設定。つまりそれらのコアからのイベントのみがuncore PMCにカウント。

### 一般設定レジスタ

#### ACTLR_EL1 (ARM標準または非標準)

* [1] TSOの有効化
* [3] HWPを無効化
* [4] APFLGを有効化
* [5] Apple FP拡張を有効化。これにより、FPCR.FZはdon't careとなり、AFPCR.DAZとAFPCR.FTZに置換。
* [6] PRSVを有効化
* [12] IC IVAU ASIDを有効化

#### HACR_EL2 (ARM標準または非標準)

* [20] guest CNTV timerをマスク (1=masked)

SYS_APL_GTIMER_MASKとは動作が異なります。あちらは先にタイマーをマスクするが、こちらはFIQをSYS_APL_HV_TMR_LRで『保留』します。

### 浮動小数点とAMXのレジスタ

#### SYS_APL_AFPCR_EL0

Apple固有の浮動小数点関連ビット。

* [0] DAZ (Denormals as Zero)
* [1] FTZ (Flush to Zero)

SSEと等価のモードビットを実装しています。意味があるようにするためにはACTLR_EL1.AFPとともに有効化する必要があります。

AArch64のFEAT_AFT機能が同様の対応を実装しているが、標準化前にAppleが実装しました。標準版では、FPSCR[1] (AH)を1にして、FPCR[0] (FIZ)はDAZ、FPCR[24] (FZ)はFTZのようにする必要があります。

### IDレジスター

#### MIDR_EL1 (ARM標準)

* [15:4] PartNum
    * 1: Alcatraz Cyclone (A7 / H6P)
    * 2: Fiji Typhoon (A8 / H7P)
    * 3: Capri Typhoon (A8X / H7G)
    * 4: Malta / Elba Twister (TSMC A9 / A9X / H8P / H8G)
    * 5: Maui Twister (Samsung A9 / H8P)
    * 6: Cayman / Gibraltar Hurricane-Zephyr (A10 / T2 / H9P / H9M Fusion core)
    * 7: Myst Hurricane-Zephyr (A10X / H9G Fusion core)
    * 8: Skye Monsoon (A11 / H10 p-core)
    * 9: Skye Mistral (A11 / H10 e-core)
    * 11: Cyprus Vortex (A12 / H11P p-core)
    * 12: Cyprus Tempest (A12 / H11P e-core)
    * 15: M9 (S4/S5)
    * 16: Aruba Vortex (A12X/Z / H11G p-core)
    * 17: Aruba Tempest (A12X/Z / H11G e-core)
    * 18: Cebu Lightning (A13 / H12 p-core)
    * 19: Cebu Thunder (A13 / H12 e-core)
    * 34: M1 Icestorm (H13G e-core)
    * 35: M1 Firestorm (H13G p-core)
    * 38: Turks (S6 / M10)

* [31:24] Implementer (0x61 = 'a' = Apple)

#### MPIDR_EL1 (ARM標準)

* [23:16] Aff2: 0:e-core, 1:p-core
* [15:8] Aff1: Cluster ID
* [7:0] Aff0: CPU ID

#### AIDR_EL1 (ARM標準または非標準)

* [0] MUL53
* [1] WKDM
* [2] ARCHRETENTION
* [4] AMX
* [9] TSO
* [19] APFLG
* [20] PSRV

### 未知のレジスタ

#### s3_6_c15_c1_0_EL1 / s3_6_c15_c1_5_EL1 / s3_6_c15_c1_6_EL1

これらはAPRRの新しいバージョンらしい。

#### s3_4_c15_c5_0_EL1

これはinit時にcore ID(クラスタ内)が書き込まれる。

#### AHCR_EL2

エンコードが不明。ACTLR_EL1[12]と関連。

#### s3_4_c15_c10_4 (m1n1内ではSIQ_CFG_EL1とラベル付け)

コアがそのレジスタのコピーに0x3を書き込むとAICv2はそのコアにIRQを送りません。レジスタはコア複合体自身の一部であるので、FIQsは影響を受けません
(0x0と0x2はコアのIRQを有効にする既知の値だが、0x0はEL1に奇妙な問題を引き起こすらしい)。
ハードウェアIRQ選択ヒューリスティックに対するある種の『親和性(affinity)』なのかもしれません？
