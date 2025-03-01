2025/3/1時点の[HW-SPRR-and-GXF](https://github.com/AsahiLinux/docs/blob/main/docs/HW-SPRR-and-GXF.md)の翻訳

---
# Guarded execution

Guarded executionモードはEL1とEL2の隣にある横方向の例外レベルで、同じページテーブルを使用しますが異なるパーミッションを使用します（SPRRを参照）。これらのレベルはGL1およびGL2と呼ばれます。これはS3_6_C15_1_2のビット1で有効になります。

命令`0x00201420`はgenterで、ELからGLに切り替え、PCを`S3_6_C15_C8_1`に設定します。`0x00201420`はgexitで、ELに戻ります。

```
#define SYS_GXF_ENTER_EL1 sys_reg(3, 6, 15, 8, 1)
```

Guarded modeではEL1/2と同様にELR、FAR、ESR、SPSR、VBAR、TPIDRの各レジスタが個別に設定されています。
さらに、ASPSRレジスタは、gexitがGLとELのどちらに戻るべきかを示します。 

```
#define SYS_TPIDR_GL1 sys_reg(3, 6, 15, 10, 1)
#define SYS_VBAR_GL1 sys_reg(3, 6, 15, 10, 2)
#define SYS_SPSR_GL1 sys_reg(3, 6, 15, 10, 3)
#define SYS_ASPSR_GL1 sys_reg(3, 6, 15, 10, 4)
#define SYS_ESR_GL1 sys_reg(3, 6, 15, 10, 5)
#define SYS_ELR_GL1 sys_reg(3, 6, 15, 10, 6)
#define SYS_FAR_GL1 sys_reg(3, 6, 15, 10, 7)
```

# SPRR
SPRRはページテーブルエントリからパーミッションビットを取り出し 属性インデックスに変換します。MAIRが動作する様子と類似:

```
   3      2      1     0
 AP[1]  AP[0]   UXN   PXN
```
UXN と PXN は APRR と比較して反転していることに注意してください。

これはそれぞれのエントリが4ビット持つシステムレジスターへのインデックスとして使われます:
```
    3     2     1     0
  GL[1] GL[0] EL[1] EL[0]
```

GL/ELはほぼ別々に扱うことができますが、2つの例外があり、特定のGLパーミッションによって
2つの例外があり、特定のGLパーミッションが2つのELビットの通常の意味を変更します。

| レジスタ値 | EL page permissions | GL page permissions |
|-|-|-|
| `0000` | `---` | `---` |
| `0001` | `r-x` | `---` |
| `0010` | `r--` | `---` |
| `0011` | `rw-` | `---` |
| `0100` | `---` | `r-x` |
| `0101` | `r-x` | `r-x` |
| `0110` | `r--` | `r-x` |
| `0111` | `---` | `r-x` |
| `1000` | `---` | `r--` |
| `1001` | `--x` | `r--` |
| `1010` | `r--` | `r--` |
| `1011` | `rw-` | `r--` |
| `1100` | `---` | `rw-` |
| `1101` | `r-x` | `rw-` |
| `1110` | `r--` | `rw-` |
| `1111` | `rw-` | `rw-` |

これらの4ビットはELモードやGLモードで動作しているときの実際のパーミッションを示しています。
EL0とEL1はパーミッションが分離されるように別々のレジスタを持っています。

S3_6_C15_C1_0 / SPRR_CONFIG_EL1のビット1は、SPRRと新しいシステムレジスターへのアクセスを有効にします。

S3_6_C15_1_5はEL0用、S3_6_C15_1_6はEL1/GL1用のパーミッション・レジスタです。

```
#define SYS_SPRR_CONFIG_EL1       sys_reg(3, 6, 15, 1, 0)
#define SPRR_CONFIG_EN            BIT(0)
#define SPRR_CONFIG_LOCK_CONFIG   BIT(1)
#define SPRR_CONFIG_LOCK_PERM_EL0 BIT(4)
#define SPRR_CONFIG_LOCK_PERM_EL1 BIT(5)

#define SYS_SPRR_PERM_EL0 sys_reg(3, 6, 15, 1, 5)
#define SYS_SPRR_PERM_EL1 sys_reg(3, 6, 15, 1, 6)
```
