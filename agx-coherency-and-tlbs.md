Asahi Lina氏の[AGX coherency, caching, and TLBs](https://gist.github.com/asahilina/1a70f1f26b9cd3c52d61d2b8e8b27673)の2022年10月15日時点での非公式日本語訳。

Apple SiliconのGPUの解析とLinuxドライバの実装に関するメモです。
ライセンスが不明ですがおそらくCC BY-SA 4.0として扱っていいと思うのでAsahi Linux関係の資料として勝手に翻訳しました。
まずければ消します。

略語について適宜解説を追記したり日本語訳wikiにある場合は日本語wikiへのリンクを追加しています。

参考文献:
- [ARMのMMUの解説](https://logmi.jp/tech/articles/323864)
- [AMDのhUMAの解説](https://pc.watch.impress.co.jp/docs/column/kaigai/598132.html)

---
## AGXのコヒーレンシー、キャッシング、TLBについて

以下は、[AGX](https://github.com/asfdrwe/asahi-linux-translations/wiki/%E7%94%A8%E8%AA%9E%E9%9B%86#a)のメモリモデルの微妙な部分と、
観察したTLB(訳注: Translation Lookup Buffer)/キャッシュの問題についての、現在の理解についてのメモです。

### GPU側

#### MMU (UAT)

AGX MMUには64個のコンテキストスロットがあります（63個はユーザーコンテキストに使用可能）。ページテーブルのベースアドレスは、メインメモリ内のページ
（Appleは『TTBAT』と呼んでいます）に格納されています。ユーザーハーフとカーネルハーフ（ローとハイ）がありますが、ハイハーフは
非常に特殊なケースでしか使われないので、GPUの観点からは無視できます。パーミッションビットがおかしく、同じページテーブルを
[ASC](https://github.com/asfdrwe/asahi-linux-translations/wiki/%E7%94%A8%E8%AA%9E%E9%9B%86#a)で共有している（解釈も異なる）が、
基本的にはARMのページテーブルです。各コンテキストスロットには、ARMのTTBR(訳注:Translate Table Base Register)のようなASID(何ビット?)
(訳注:Address Space IDentifier)があります。

GPUユーザオブジェクトは、macOSでは常にGPU=R-かGPU=RWのどちらかにマップされます:

* `GPU=RW, EL1=RW, GL1=RW, perm=10011, Shared, Local, Owner=OS, AF=1, SH=0` 
* `GPU=R-, EL1=RW, GL1=RW, perm=10111, Shared, Local, Owner=OS, AF=1, SH=0`

ページテーブル自体は、macOSから`ff:OS` Normal-Cached Outer Sharable(ノーマルキャッシュ外部共有)としてアクセスされます。TTBAT と
ハンドオフエリアは Normal-Uncached Outer Sharable(ノーマル非キャッシュ外部共有) (`44:OS`) を使用し、これは
[DCP](https://github.com/asfdrwe/asahi-linux-translations/wiki/%E7%94%A8%E8%AA%9E%E9%9B%86#d) ブートフレームバッファと同じです 
(Apple はこれを 『real time』 と呼びます)。

#### TLB

GPU MMUのTLBは、Outer Sharable(外部共有)ドメインで[AP](https://github.com/asfdrwe/asahi-linux-translations/wiki/%E7%94%A8%E8%AA%9E%E9%9B%86#a)側の
TLBI命令（！）で管理されています。つまり、CPUはGPUの仮想アドレス(訳注: VA, Virtual Address)をCPUの仮想アドレスとみなして（！）、TLBI命令を直接発行しているだけなのです。そのため、GPU TLBを
撃ち落とすと無関係なCPU TLBも誤って撃ち落としてしまう可能性があります。macOSでは、CPUのユーザ空間の仮想空間アロケータにGPUの仮想空間レンジをグローバルに予約することで
これを解決しています（痛い！T_T）。

TLBI命令はTTBATで設定されたASIDを使用します(うまくいけば？macOSは常にこれらを1:1に設定するので、これはもっと実験が必要です...)

#### キャッシュ

GPU には明らかにキャッシュがありますが、そのコヒーレンシは現在のところ不明です。レンダーコマンドは 『attachments』 のリストを取りますが、
これには通常フレームバッファと z バッファが含まれますが、このリストにそれらを含めないことは、何も壊さないようです。このリストは x86 の 
MTRR(訳注: Memory Type Range Register、[解説](http://mcn.oops.jp/wiki/index.php?CPU%2FCPUID%2FMTRR))
に似ていて、これらのメモリ範囲を write-combine (キャッシュをバイパスする) として設定するのだと思います。これは、メインメモリに向かう途中で
有用なキャッシュラインを退避させたくないので、フレームバッファにとっては理にかなっています。

AP 側では、すべての GPU アクセス可能バッファは Normal-Cached Outer Sharable としてマップされているようで、これはキャッシュが（少なくとも一部の時間では）
コヒーレントであることを示唆しています。PTE(訳注: Page Table Entry)をまだダンプしていませんが、書き込まれた仮想アドレスのアドレス変換(訳注: AT, Address Translation)
命令出力には`ff:OS` という属性が表示されます。

### ASC(コプロセッサ)側

### AXI2AFブリッジ

メインのSoCファブリックとAXI(訳注: Advanced eXtensible Interface,[解説](https://ja.wikipedia.org/wiki/Advanced_eXtensible_Interface))コプロセッサ
インターフェースを橋渡しするAXI2AF (AXI to Apple Fabric) ブリッジが存在します。このブリッジは、TLBIを適切に渡すために、いくつかのハードコードされたpoke
(訳注: メモリ書き込み)で設定する必要がありますが、これがどのように機能するか、また信頼性があるかはまだ明らかではありません。これらのpokeは現在 m1n1 にあります。

#### MMU

コプロセッサは GPU カーネルの半分とページテーブルを共有します。これにはファームウェアのコード/データも含まれます。許可ビットは ASC のために異なって解釈されるので、
ASC にのみアクセス可能なものを持つことができます (これにはすべての ファームウェア(FW) コード/データビットが含まれます)。

ASCはGPUのようにCPUのTLBI命令を聞いているようですが、これが信頼できるかどうかは不明です。 macOSはファームウェア側のTLBIをASID 0x40（64）で発行しますが、
実際にファームウェアがそのASIDをTTBR1に設定したのを見たことがないにも関わらず...これがバスのマジックなのかバグなのかは不明です。

macOSはファームウェアのアドレス空間ではほとんどアンマップ/リマップしないので、そのフラッシュ/その他のメカニズムが信頼できるかどうかは不明です
(そうでなくても動作する可能性は十分にあるのですが)。しかし、たとえ何度も使わなくても、ASC で TLB の無効化を確実に行う方法を理解することは
良いことだと思います。

#### キャッシュ

ASC のキャッシュは AP のキャッシュとコヒーレント*でない*ようです。しかし、APキャッシュはSoCファブリック全体とコヒーレントです。つまり、AP は全てが
コヒーレントであるかのように装うことができ、ASC はキャッシュ管理をしなければならないという非対称なキャッシュ状況になっています。

構造体は、ASCの2つのキャッシュモード（キャッシュなしとキャッシュあり）を使って（OSによって）マップされます。アンキャッシュはグローバルフラグ、
統計タイプ、AP→ASCカウンタ、リングバッファポインタ、ASC→APリングバッファに使用されます。両側から見ると、これらのマッピングは
首尾一貫しているように見えます（ASC側にキャッシュがないだけ）。キャッシュは他のもののために使用されます。コマンドバッファやAPがASCのために割当/提供した
ファームウェアオブジェクトのようなものに対してです。ASCはこれらの範囲にアクセスする前に無効化します。

ASCのPTEにこれらのマッピングモードを見たことがあります:

* `GPU=--, EL1=RW, GL1=RW, perm=10110, Shared, Global, Owner=OS, AF=1, SH=0`
* `GPU=RW, EL1=RW, GL1=RW, perm=10011, Shared, Global, Owner=OS, AF=1, SH=0`
* `GPU=--, EL1=RW, GL1=RW, perm=10110, Normal, Global, Owner=OS, AF=1, SH=0`
* `GPU=RW, EL1=RW, GL1=RW, perm=10011, Normal, Global, Owner=OS, AF=1, SH=0`

ASCのみかGPUと共有され、Shared（キャッシュなし）またはNormal（キャッシュあり）アトリビュートを使用することができます。
SHは常に0（Non Shared）であり、これは共有性／キャッシュコヒーレンシがASC側からは実際には重要でない／機能しないという考えと一致します。

ASCキャッシュバッファのマッピングを解除するには、キャッシュがフラッシュされるようにするための特別なdanceが必要です。
ドライバは現在、変数を排除するためにすべての共有/非キャッシュをマップしており、アンマップdanceがまだ実装されていないためです。

#### UAT PPL ハンドオフ

macOSのPPL(訳注: Page Protection Layer, [解説](https://support.apple.com/ja-jp/guide/security/sec8b776536b/web))とASCのuPPL
(訳注:u? Page Protection Layer,何の訳語かわかりません)の間の通信を
調整するために使用される共有ページがあります。これらはページテーブルを管理する特権的なコード片です(そのため、OS/ファームウェアの残りの部分が侵害されても、
メモリマップの制御を確実に行うことができます)。ハンドオフには以下のものが含まれます。

* マジックナンバー
* UAT TTB修正用の[Dekker lock](https://ja.wikipedia.org/wiki/%E3%83%87%E3%83%83%E3%82%AB%E3%83%BC%E3%81%AE%E3%82%A2%E3%83%AB%E3%82%B4%E3%83%AA%E3%82%BA%E3%83%A0)
* いくつかの状態フラグ
* 各コンテキスト0〜63とASC用の64のフラッシュ構造体

TTBの書き込みは事実上アトミックな64ビットストアであり、ASCが書き込むことはないので、Dekker lockが何のためにあるかは明らかではありません。
念のためこれを実装しています。

面白い逸話：ハンドオフ領域をキャッシュなしに切り替えた後、IMPDEF（！！）(訳注: IMPlemenation Defined, 実装依存)同期フォルトが発生しました。
VMを解放するときにTTBでcompare-exchangeしていたからで、それらアトミックRMW命令はキャッシュなしモードではどうやら動作しないらしいです...
IMPDEFフォールトは想定外だったんですが！

フラッシュ構造体は、TLB無効時に（目的不明、おそらくダミー？）使用されますが、キャッシュマッピングのアンマップdanceにも（既知の目的で）使用されています。

GPU仮想アドレスのmacOS TLB無効はこんな感じです:

```
# [cpu3] [HandoffTracer] MMIO: R.8   MAGIC_FW = 0x4b1d000000000002 ()
## This is the same PTE that was already present (no change)
# [cpu3] [AGXTracer@/arm-io/gfx-asc] UAT write L0 at 1:0x1500000000 (#0x354) -> 0x00E0000961DF4C0B
# [cpu3] [AGXTracer@/arm-io/gfx-asc] UAT map 1:0x1500d50000 -> 0x961df4000 (0xe0000961df4c0b (OS=1, UXN=1, PXN=1, OFFSET=0x25877d, nG=1, AF=1, SH=0, AP=0, AttrIndex=2, TYPE=1, VALID=1))
# [cpu3] [HandoffTracer] MMIO: R.4   FLUSH_STATE[1] = 0x0 ()
# [cpu3] [HandoffTracer] MMIO: W.8   FLUSH_ADDR[1] = 0x1500d50000 ()
# [cpu3] [HandoffTracer] MMIO: W.8   FLUSH_SIZE[1] = 0x4000 ()
# [cpu3] [HandoffTracer] MMIO: R.1   UNK2 = 0x0 ()
# [cpu3] [HandoffTracer] MMIO: R.4   UNK = 0x0 ()
# [cpu3] [HandoffTracer] MMIO: R.8   FLUSH_ADDR[1] = 0x1500d50000 ()
# [cpu3] [HandoffTracer] MMIO: W.8   FLUSH_ADDR[1] = 0x1500d50000 ()
# [cpu3] [HandoffTracer] MMIO: R.8   FLUSH_ADDR[1] = 0x1500d50000 ()
# [cpu3] [HandoffTracer] MMIO: W.8   FLUSH_ADDR[1] = 0xdead001500d50000 ()
# [cpu3] [HandoffTracer] MMIO: W.4   FLUSH_STATE[1] = 0x2 ()
## There seems to sometimes be a long delay here with activity from other threads, possibly waiting for something? Unclear what...
# [cpu3] [HandoffTracer] MMIO: R.8   MAGIC_FW = 0x4b1d000000000002 ()
# [cpu3] [AGXTracer@/arm-io/gfx-asc] UAT write L0 at 1:0x1500000000 (#0x354) -> 0x0000000000000000
# [cpu3] [AGXTracer@/arm-io/gfx-asc] UAT unmap 1:0x1500d50000 (0x0 (OS=0, UXN=0, PXN=0, OFFSET=0x0, nG=0, AF=0, SH=0, AP=0, AttrIndex=0, TYPE=0, VALID=0))
# [cpu3] Pass: msr TLBI VAE1OS, x8 = 1000001500d50 (OK) (TLBI VAE1OS)
# [cpu3] [HandoffTracer] MMIO: R.1   UNK2 = 0x0 ()
# [cpu3] [HandoffTracer] MMIO: R.4   FLUSH_STATE[1] = 0x2 ()
# [cpu3] [HandoffTracer] MMIO: W.4   FLUSH_STATE[1] = 0x0 ()
```

これは何のためにあるのか謎です。ファームウェアを起動して実際にこれを処理するようなものは何も見当たりません。ハードウェアの何かがハンドオフ領域への書き込みをスヌープして、
それを何らかの形で解釈しているのでなければ、どうでしょう!

macOSのASCキャッシュページのアンマップは、実際にコプロセッサを起動してuPPLを呼び出す特別なdanceを伴います。これは、PPLがそれらのページがキャッシュからフラッシュさ
れたことを安全に確認する必要があるためです。

```
# [cpu3] [HandoffTracer] MMIO: R.8   MAGIC_FW = 0x4b1d000000000002 ()
# [cpu3] [0xfffffe00135b46d0] MMIO: R.8   0x9fff78010 (gfx_shared_region, offset 0x10) = 0x823350003
## This first remaps the pages as uncached (AttrIndex=2).
# [cpu3] [AGXTracer@/arm-io/gfx-asc] UAT write L0 at 0:0xfa00c000000 (#0x10a) -> 0x00C00009109BC44B
# [cpu3] [AGXTracer@/arm-io/gfx-asc] UAT map 0:0xfa00c428000 -> 0x9109bc000 (0xc00009109bc44b (OS=1, UXN=1, PXN=0, OFFSET=0x24426f, nG=0, AF=1, SH=0, AP=1, AttrIndex=2, TYPE=1, VAL
ID=1))
# [cpu3] [AGXTracer@/arm-io/gfx-asc] UAT write L0 at 0:0xfa00c000000 (#0x10b) -> 0x00C000090FD8044B
# [cpu3] [AGXTracer@/arm-io/gfx-asc] UAT map 0:0xfa00c42c000 -> 0x90fd80000 (0xc000090fd8044b (OS=1, UXN=1, PXN=0, OFFSET=0x243f60, nG=0, AF=1, SH=0, AP=1, AttrIndex=2, TYPE=1, VAL
ID=1))
## Then there's a TLB invalidate... but there's a bug here! The address is 0xfa00c430000 (the *end* of the range) while it should be the start!
# [cpu3] Pass: msr TLBI RVAE1OS, x14 = 40801ffe80310c (OK) (TLBI RVAE1OS)
## Then the PPL puts the range into the handoff area and sets the state to 1 (pending cache inval)
# [cpu3] [HandoffTracer] MMIO: R.4   FLUSH_STATE[64] = 0x0 ()
# [cpu3] [HandoffTracer] MMIO: W.8   FLUSH_ADDR[64] = 0xffffffa00c428000 ()
# [cpu3] [HandoffTracer] MMIO: W.8   FLUSH_SIZE[64] = 0x8000 ()
# [cpu3] [HandoffTracer] MMIO: R.1   UNK2 = 0x0 ()
# [cpu3] [HandoffTracer] MMIO: W.4   FLUSH_STATE[64] = 0x1 ()
## And then issues an op via a special ring buffer to wake up the ASC and tell it to call into the uPPL
## The uPPL them issues a cache flush/inval for this range
# [cpu3] [AGXTracer@/arm-io/gfx-asc] [kickep]   FWRing Kick 0x84000000000000 (TYPE=0x8, KICK=0x0)
# [cpu3] [AGXTracer@/arm-io/gfx-asc] FW Kick~! 0x0
# [cpu3] [AGXTracer@/arm-io/gfx-asc] [17:FWCtl] Message @0.16:
FWCtlMsg @ 0xffffffa0000c0200:
 FWCM.[  0.  8] addr = 0xffffffa00c428000
 FWCM.[  8.  4] unk_8 = 0x0
 FWCM.[  c.  4] context_id = 0x40
 FWCM.[ 10.  2] unk_10 = 0x1
 FWCM.[ 12.  2] unk_12 = 0x2
## Once this completes (how does PPL know? There's no visible polling, could just be too fast?) it does the unmaps...
# [cpu3] [HandoffTracer] MMIO: R.8   MAGIC_FW = 0x4b1d000000000002 ()
# [cpu3] [0xfffffe00135b51c8] MMIO: R.8   0x9fff78010 (gfx_shared_region, offset 0x10) = 0x823350003
# [cpu3] [AGXTracer@/arm-io/gfx-asc] UAT write L0 at 0:0xfa00c000000 (#0x10a) -> 0x0000000000000000
# [cpu3] [AGXTracer@/arm-io/gfx-asc] UAT unmap 0:0xfa00c428000 (0x0 (OS=0, UXN=0, PXN=0, OFFSET=0x0, nG=0, AF=0, SH=0, AP=0, AttrIndex=0, TYPE=0, VALID=0))
# [cpu3] [AGXTracer@/arm-io/gfx-asc] UAT write L0 at 0:0xfa00c000000 (#0x10b) -> 0x0000000000000000
# [cpu3] [AGXTracer@/arm-io/gfx-asc] UAT unmap 0:0xfa00c42c000 (0x0 (OS=0, UXN=0, PXN=0, OFFSET=0x0, nG=0, AF=0, SH=0, AP=0, AttrIndex=0, TYPE=0, VALID=0))
## Flushes the TLB again (this time with the right address!)
# [cpu3] Pass: msr TLBI RVAE1OS, x14 = 40801ffe80310a (OK) (TLBI RVAE1OS)
## And checks and clears the flush state flag, which the uPPL set to 2 to indicate it flushed the cache.
# [cpu3] [HandoffTracer] MMIO: R.1   UNK2 = 0x0 ()
# [cpu3] [HandoffTracer] MMIO: R.4   FLUSH_STATE[64] = 0x2 ()
# [cpu3] [HandoffTracer] MMIO: W.4   FLUSH_STATE[64] = 0x0 ()
```

最初のTLBIのバグに注目してください...これが、この全体が実際にmacOSできちんと信頼性を持って動作しているのか疑問に思う理由の1つです。GPUコンテキストを破壊するときだけ、
しかも少数の小さな構造体に対してのみアンマップを行うようです。したがって、信頼性が低く、偶然に動作している可能性も十分にあります... 
ASC共有構造体の大部分は、前もってマッピングされたプールから割り当てられ、決してマッピング解除されることはありません。ドライバでは、
(キャッシュモードとキャッシュモードの)grow-onlyヒープを持ち、必要なときにページを追加するだけで、一度使われたページを解放/マッピング解除することはなく、
そこから共有構造体を割り当てる予定です。これにより、PTEチャーンを最小限に抑え、TLB無効化問題全体を排除し、小さなファームウェア構造体の分野では、
ページを解放することなく済むと思います。

### 問題点

#### カーネル側の問題

時々、Linux カーネルはメモリ破壊のためにクラッシュします。これが起こるたびに、関係するメモリは以前 GPU マップされていたようで、その後アンマップされ、
通常はその場所に他のページがマップされます。不良ページにはタイル配列ポインタ（レンダリングごとにマップ/アンマップされるバッファの1つ）が頻繁に含まれます。
これはGPUが古いTLBにタイルポインタを書き戻しているのだと思っていましたが、TLB無効がタイル配列とm1n1で動作し、TLBIが起こると
すぐにGPUフォルトを引き起こすように見えるので、もう確信がありません（GPUはマイクロ秒以内にフォールトし、私はAPの観点からTLBI後にタイル配列書き込みが
起こったのを見たことがないのです）。これはASCがレンダリング前にタイル配列をクリアし、TLBが古くなっているのではないかと疑っています（次節を参照）。

#### ASC側の問題

ASC 側は頻繁におかしな方法でクラッシュします。見たことあるもの:

* バリアコマンドの終端を越えてキャッシュを無効にしようとすると、ASCがコマンドタグを何か別のものとして読み取ったことを示唆（おそらくゼロ、これはTAコマンドでもっと大きなもの）
* 非連続的なスタンプ更新のアサート。これがいつ起こるかははっきりしないが、通常はASCが更新しようとしたスタンプ値に矛盾があったことを意味（多分、古いコマンドを読んだ）
* 解放されたGEMバッファに0x42を埋め始めた後、ファームウェアが未知のSKUコマンド0x2でアサートするのを見たことがあり。SKUコマンドIDは0x3fでマスクされているので、解放されたGEMバッファを本物のMicroSeqとしてパースしようとしたのだと思われる
* *Linuxカーネルの仮想ポインタ*を参照解除しようとしていますが、これはカーネルページをランダムに読み込んでいることを示唆
* さらにランダムなNULL参照など

これらのすべてのケースで、ハイパーバイザー側からMMU PTEを見ると、すべてが適切にマッピングされていることがわかりますが、少なくともいくつかのケースでは、
以前のマッピングを使用すると、観察した効果と一致します（ハイパーバイザーのAGXトレーサー UAT ページテーブル キャッシュは、TLBIをリッスンしないので、
実際にはこの問題自体を再現することができます）。つまり、ASCに対するTLB invalsは信頼できないようです（AXI2AFポークを発見する前は完全に壊れていましたが、
まだ多少壊れているようです・・・）。

#### GPU側の問題

ASC側の問題を何も解放しないことで緩和した後、GPUをオンにし続ける（長いシャットダウンタイムアウトを設定する）と、GPU側の問題が発生します。これは通常、
GPUのタイムアウトで、明示的なフォールトなしです（フォールトレジスタがフォールトを示さない）。何が起こっているのかよくわからないのですが...
確かに過去に小さすぎるタイル状の頂点バッファでタイムアウトやおかしなことになったのを見た覚えがあるので、その部分について疑っています。
もっと再現性を高める必要があり、Python側で再現できれば...

特に、`kmscube`は通常動作しますが、2フレーム目でクラッシュすることも確認済みです。GNOMEセッションを開始すると、かなり確実にクラッシュします。
また、ある時点でビジュアルが壊れるのを見たことがあります。良いニュースは、GPU シャットダウンのタイムアウトを 1秒以上 に設定した後、完全なロギングと
ハイパーバイザのトレースを行ってもこれらすべてがまだ頻繁に起こるので、これはもう微妙なタイミングの問題ではなく、Python で再現可能なもののはずです...

また、完全にハングすることもあります(ファームウェアからのタイムアウトメッセージもなく、完了もしません)。

#### GPU シャットダウンハック
レンダーバッチごとにGPUがパワーダウンするのを待つと、安定性の問題がほぼ解決されるようです。最初これはGPU TLBをクリアするためだと思ったのですが、
それ以上のことがあると思うのです。GPUがパワーダウンした後、ASCもその直後にパワーダウンするのだと今は思っています（おそらくWFI
(訳注:Wait For Interrupt,割り込みまち命令)かdeep WFI?）。
だから、GPUがシャットダウンするのを（ASCが維持するグローバルをポーリングすることによって）*事実上*ほとんどASCがシャットダウンするのを待つことになります。
これはASC側の問題がなくなった理由（ASC構造体を解放して再マップしたときでも）と、GPUパワーダウンのタイムアウトに依存して不安定になることがある理由を説明しています:
ASCアイドルモードはおそらくキャッシュやTLBをクリアします。このように待ったときほとんど常にそれが起こりました。

しかし、GPUシャットダウンは、GPU側の問題を解決するためのものでもあります。
