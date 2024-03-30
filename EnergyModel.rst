`Energy Model of devices <https://docs.kernel.org/power/energy-model.html>`_  の非公式日本語訳です。
ライセンスは原文のライセンスGPL-2.0に従います。

訳注:

* まだ作業中(2024/3/30)
* APIについてkernelヘッダの埋め込みはそのまま貼り付け
* カーネル文書へのリンクで日本語訳があるものは日本語訳へのリンクに変更

=======================
機器エネルギーモデル
=======================

1. 概要
-------

エネルギーモデル（Energy Model, EM）フレームワークは、さまざまな性能レベルでデバイスが消費する電力を知るドライバと、
その情報を使用してエネルギーを考慮した判断を行うカーネルの間のインターフェイスとして機能します。

機器が消費する電力に関する情報源はプラットフォームによって大きく異なります。
これらの電力コストは場合によっては、Devicetreeデータを用いて推定することができます。
また、ファームウェアの方が良く知っている場合もあります。あるいは、ユーザー空間が最適かもしれません。
各クライアント・サブシステムが可能な限りの情報源への対応を独自に繰り返し実装することを避けるために、
EMフレームワークは電力コストテーブルの形式を標準化する抽象化レイヤーとして介入することで、
冗長な作業を避けることができます。

電力値はマイクロワットまたは『抽象尺度』で表現されます。複数のサブシステムがEMを使用する可能性があり、
システムインテグレータは、電力値尺度の種類の要件が満たされているかどうかをチェックする必要があります。
`Energy Aware Scheduling <https://github.com/asfdrwe/asahi-linux-translations/blob/main/EnergyAwareScheduling.rst>`_
に例があります。『抽象尺度』で表現された熱量や電力制限された電力値のようなサブシステムでは問題を引き起こす可能性があります。
これらのサブシステムは過去に使用された電力の推定に関心があり、実際のマイクロワット値が必要になるかもしれません。
このような要件の例は `Power Allocator governor tunables <https://docs.kernel.org/driver-api/thermal/power_allocator.html>`_ のインテリジェント電源配分にあります。
カーネルサブシステムは、登録されたデバイスが（EM 内部フラグに基づいて）一貫性のない尺度を持っているか
チェックする自動検出を実装できます。気をつけないといけない重要なことは、電力値が「抽象尺度』で表現されている場合
マイクロジュール単位の実エネルギーを導き出すことは不可能であることです。

下図は、ドライバの例（ここではArm専用ですがアプローチはどのアーキテクチャにも適用可能）を示しています。
フレームワークに電力コストを提供するドライバと、そこからデータを読み取るクライアントの例です:: 


       +---------------+  +-----------------+  +---------------+
       | 熱量 (IPA)     |  | スケジューラ(EAS) |  |     その他     |
       +---------------+  +-----------------+  +---------------+
               |                   | em_cpu_energy()   |
               |                   | em_cpu_get()      |
               +---------+         |         +---------+
                         |         |         |
                         v         v         v
                        +---------------------+
                        |    エネルギーモデル    |
                        |     フレームワーク       |
                        +---------------------+
                           ^       ^       ^
                           |       |       | em_dev_register_perf_domain()
                +----------+       |       +---------+
                |                  |                 |
        +---------------+  +---------------+  +--------------+
        |  cpufreq-dt   |  |   arm_scmi    |  |    その他     |
        +---------------+  +---------------+  +--------------+
                ^                  ^                 ^
                |                  |                 |
        +--------------+   +---------------+  +--------------+
        | Devicetree   |   |  ファームウェア |  |      ?       |
        +--------------+   +---------------+  +--------------+

訳注: IPAはおそらく`Intelligent Power Allocation <https://developer.arm.com/Tools%20and%20Software/Intelligent%20Power%20Allocation>`_

CPU 機器の場合、EM フレームワークは、システム内の『性能ドメイン(performance domain)』ごとに電力コストテーブルを管理します。
性能ドメインとはCPU のグループです。性能ドメインは一般的にCPUFreq ポリシーと 1 対 1 に対応付けされます。
性能ドメイン内のすべてのCPUは同じマイクロアーキテクチャである必要があります。異なる性能ドメイン内のCPUは
異なるマイクロアーキテクチャを持つことができます。

静的電力（リーク）による電力変動をより良く反映するために、EMは電力値の実行時修正に対応しています。
このメカニズムは、変更可能なEM perf_stateテーブルメモリを解放するために
RCU(訳注: `Read-Copy-Update <https://ja.wikipedia.org/wiki/%E3%83%AA%E3%83%BC%E3%83%89%E3%83%BB%E3%82%B3%E3%83%94%E3%83%BC%E3%83%BB%E3%82%A2%E3%83%83%E3%83%97%E3%83%87%E3%83%BC%E3%83%88>`_ ) 
に依存しています。そのユーザーであるタスクスケジューラもRCUを使用してこのメモリにアクセスします。
EMフレームワークは、変更可能なEMテーブルの新しいメモリを割り当て/解放するためのAPIを提供します。古いメモリは、
与えられたEMランタイムテーブルのインスタンスの所有者がいなくなったときに、RCUコールバック機構を使って自動的に解放されます。
これは、krefメカニズムを使用して追跡されます。実行時に新しいEMを提供するデバイスドライバは、それが不要になったときに
安全に解放するためにEM APIを呼び出す必要があります。EMフレームワークは可能な場合にクリーンアップ処理します。

EM値を変更しようとしているカーネルコードはmutexを使用することで同時アクセスから保護されます。したがって、
デバイスドライバコードがEMを変更しようとするときはスリープコンテキスト(sleeping context)で実行されなければなりません
(訳注: `たぶん排他制御機構に関するこの辺の話<https://docs.kernel.org/locking/mutex-design.html>`_)。

実行時に変更可能なEMにより、『単一の、実行時全体にわたって静的なEM』（システム・プロパティ）設計から、
『ワークロードなどに応じて実行時に変更可能な単一のEM』（システムとワークロードのプロパティ）設計に切り替わります。

また、各EMの性能状態に対してCPU性能値を変更することも可能です。したがって、（指数曲線である)完全な電力と性能の
プロファイルはワークロードやシステム特性などに応じて変更することができます。

2. コア API
------------

2.1 コンフィグオプション
^^^^^^^^^^^^^^^^^^^^^^^

EM フレームワークを使用するには、CONFIG_ENERGY_MODEL を有効にする必要があります。


2.2 性能ドメインの登録
^^^^^^^^^^^^^^^^^^^^

『上級(advanced)』EMの登録
~~~~~~~~~~~~~~

『上級』EMはドライバーがより正確なパワーモデルを提供することができるようになっていることから名付けられました。
(『単純(simple)』EMの場合のように）フレームワークで実装された数式に限定されるものではありません。
各性能状態に対して実行される実際の電力測定をよりよく反映することができます。したがって、この
EM静的電力（リーク）を考慮することが重要である場合には、この登録方法を優先すべきです。

ドライバは、以下の API を呼び出して、性能ドメインを EM フレームワークに登録します::

  int em_dev_register_perf_domain(struct device *dev, unsigned int nr_states,
		struct em_data_callback *cb, cpumask_t *cpus, bool microwatts);

ドライバは<周波数、電力>タプルを返すコールバック関数を提供しなければなりません。
ドライバによって提供されるコールバック関数は、どのような関連する場所（DT(訳注: Devicetree)、ファームウェア、...）から、
必要と思われるあらゆる手段で、データを取得するか自由です。
CPU 機器の場合のみ、ドライバはcpumaskを使用して性能ドメインのCPUを指定しなければなりません。
CPU 以外の機器の場合、最後の引数はNULLに設定しなければなりません(訳注: cpumask_t *cpus をNULLにするという意味です。
`後からem_dev_register_perf_domainに引数bool microwattsが追加された <https://github.com/torvalds/linux/commit/c250d50fe2ce627ca9805d9c8ac11cbbf922a4a6>`_ ので『最後の引数』の意味がずれています)。
最後の引数『microwatts』は正しい値を設定することが重要です。EMを使用するカーネルサブシステムは、すべての
EM 機器が同じ尺度を使用しているかどうかをチェックするために、このフラグに依存することがあります。
異なる尺度がある場合、これらのサブシステムは警告やエラーを返したり、動作を停止したり、パニックを起こしたりすることに
なるかもしれません。このコールバックを実装したドライバの例については3節、この API の詳細については2.4節 を参照してください。

DT(訳注:Devicetree)を利用するEMの登録
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

EMはOPP(訳注: `Operating Performance Point <https://docs.kernel.org/power/opp.html>`_)
フレームワークを使用して登録することもでき、DTの『operating-points-v2』内の情報に登録することもできます。
DT の各 OPP エントリは、マイクロワット電力値が含まれるプロパティ『opp-microwatt 』に拡張できます。
このOPP DTプロパティにより、プラットフォームは、総電力（静的＋動的）を反映する EM 電力値を登録することが
できます。これらの電力値は実験や測定から直接得られるかもしれません。

『人工(artificial)』EM の登録
~~~~~~~~~~~~~~~~~~~~~~~~~

各性能状態の電力値に関する詳細な知識が不足しているドライバーのために、カスタム・コールバックを提供するオプションがあります。
コールバック.get_cost() はオプションで、EASによって使用される『コスト』値を提供します。
これはCPUタイプ間の相対効率に関する情報のみを提供するプラットフォームにとって有用であり、
抽象的な消費電力モデルを作成することができます。しかし、抽象的な電力モデルであっても、入力電力値の
サイズ制限を考慮すると、適合させるのが難しい場合があります。これによって、EMの内部計算式が『コスト』値を計算する際に
強いられるものととは異なる関係を持つEAS情報を提供することができるようになります。
このようなプラットフォームにEMを登録するには、ドライバは、フラグ『microwatts』を0に設定し、.get_power()コールバックを
提供し、.get_cost()コールバックを提供しなければなりません。EMフレームワークは、このようなプラットフォーム
を適切に処理します。そういったプラットフォームではEM_PERF_DOMAIN_ARTIFICIALフラグが設定されます。
EMを使用している他のフレームワークでは、このフラグをテストし、適切に扱うために特別な注意を払う必要があります。

『単純』EMの登録
~~~~~~~~~~~~~~~~~~~~~~~~~~~

単純EMは、フレームワーク・ヘルパー関数cpufreq_register_em_with_opp()を使って登録されます。
これは以下の数学式に強く結びつくパワーモデルを実装しています。::

	Power = C * V^2 * f

このメソッドを使って登録されたEMは、実際の機器の物性を正しく反映しないかもしれません。
例えば、静的消費電力（リーク）が重要な場合などです。

2.3 性能ドメインへのアクセス
^^^^^^^^^^^^^^^^^^^^^^^^

エネルギーモデルへのアクセスを提供する 2 つの API 関数があります:
em_cpu_get()はCPU IDを引数にとり、em_pd_get()はデバイスポインタを引数にとります。
どちらのインターフェイスを使用するかはサブシステムによって異なりますが、CPU 機器の場合は、
どちらの関数も同じ性能ドメインを返します。

CPUのエネルギーモデルに興味のあるサブシステムはem_cpu_get() API を使用して取得できます。
エネルギーモデルテーブルは、性能ドメインの作成時に一度割り当てられ、メモリ上に保持されます。

性能ドメインが消費するエネルギーはem_cpu_energy() APIを使用して推定できます。この推定は、schedutil
CPU 機器の場合、CPUfreq governorが使用されていると仮定して計算されます。現在のところ、この計算は
他の種類の機器には提供されていません。

上記のAPIに関する詳細は、``<linux/energy_model.h>`` または2.4節にあります。

2.4 ランタイム修正
^^^^^^^^^^^^^^^^

実行時にEMを更新したいドライバは、以下の専用関数を使用して、変更されたEMの新しいインスタンスを割り当てる必要があります。
APIは以下です::

  struct em_perf_table __rcu *em_table_alloc(struct em_perf_domain *pd);

これにより、EMフレームワークが必要とするRCUとkrefを含む新しいEMテーブルを含む構造体を割り当てることができます。

『struct em_perf_table』は性能状態を昇順に並べたリストである配列『struct em_perf_state state[]』を含みます。
このリストは、EMを更新したいデバイス・ドライバによって入力されなければなりません。
周波数のリストは（ブート中に作成された）既存のEMから取得することができます。
『struct em_perf_state』内の内容も、同様にドライバが入力しなければなりません。

これはRCUポインタswapを使用してEM更新を行うAPIです::

  int em_dev_update_perf_domain(struct device *dev,
			struct em_perf_table __rcu *new_table);

ドライバは、割り当てられて初期化された新しい EM『struct em_perf_table』へのポインタを提供しなければなりません。
この新しいEMはEMフレームワーク内で安全に使用され、カーネル内の他のサブシステム（thermal(熱量)、powercap(電力制限)）から
見えるようになります。このAPIの主な設計目標は、高速で、実行時に余分な計算やメモリ割り当てを行わないことです。
事前に計算されたEMがデバイスドライバで利用可能な場合、性能のオーバーヘッドを抑えて、単純にEMを再利用できるようにすべきです。

EMを解放するために、ドライバによって先に提供された場合には（例えば、モジュールがアンロードされたときなど）、
APIを呼び出す必要があります::

  void em_table_free(struct em_perf_table __rcu *table);

これにより、他のサブシステム（EASなど）が使用していないときは、EMフレームワークがメモリを安全に削除できるようになります。

他のサブシステム（thermal、powercapなど）で電力値を使用するには、読み込み側を保護し、EM のテーブルデータの
一貫性を提供する API を呼び出す必要があります::

  struct em_perf_state *em_perf_state_from_pd(struct em_perf_domain *pd);

これは、昇順に並べた性能状態の配列である『struct em_perf_state』ポインターを返します。
この関数は、（rcu_read_lock()の後)RCUの読み取りロック・セクション内で呼ばれます。
EMテーブルが不要になったらrcu_real_unlock()を呼び出す必要があります。このようにすることで、EMは
RCU読み取りセクションを安全に使用しを安全に使用し、ユーザーを保護します。また、EMフレームワークが
メモリ解放することができます。使い方の詳細は、3.2節のにあります。

em_perf_state::costの値を計算するデバイスドライバ専用のAPIがあります。::

  int em_dev_compute_costs(struct device *dev, struct em_perf_state *table,
                           int nr_states);

EMからのこれらの『コスト』値はEASで使用されます。新しい EM テーブルは、エントリ数とデバイスポインタとともに
渡されなければなりません。コスト値の計算が適切に行われた場合、この関数の戻り値は0となります。この関数は、
各性能状態の非効率性を正しく設定するための処理も行います。それに応じてem_perf_state::flagsを更新します。
そして、そのような準備された新しいEMをem_dev_update_perf_domain()関数に渡すことができ、それを使用することがでます。

上記のAPIの詳細については、``<linux/energy_model.h>``、およびに3.2節にデバイスドライバでの更新メカニズムの
簡単な実装を示すサンプルコードがあります。

2.5 本APIの説明詳細
^^^^^^^^^^^^^^^^^

.. code-block:: C

  struct em_perf_state
    性能ドメインの性能状態


定義

.. code-block:: C

  struct em_perf_state {
    unsigned long performance;
    unsigned long frequency;
    unsigned long power;
    unsigned long cost;
    unsigned long flags;
  };

メンバー

performance
    与えられた周波数でのCPU性能(容量)
frequency
    CPUFreqと整合性を保つKHz単位での周波数
power
    （1CPUまたは登録された機器によって）このレベルで消費される電力。静的消費電力と動的消費電力の合計
cost
    エネルギー計算中に使用されるこのレベルに関連するコスト係数。次式に等しい: power * max_frequency / frequency
flags
    下記の『em_perf_state flags』の説明を参照

.. code-block:: C

  struct em_perf_table
    性能状態テーブル

定義

.. code-block:: C

  struct em_perf_table {
    struct rcu_head rcu;
    struct kref kref;
    struct em_perf_state state[];
  };

メンバー

rcu
    安全なアクセスと破壊に使用されるRCU
kref
    ユーザーを追跡するための参照カウンター
state
    昇順に並べられた性能パフォーマンス状態のリスト

.. code-block:: C

  struct em_perf_domain
    性能ドメイン

定義

.. code-block:: C

  struct em_perf_domain {
    struct em_perf_table __rcu *em_table;
    int nr_perf_states;
    unsigned long flags;
    unsigned long cpus[];
  };

メンバー
em_table
    実行時に修正可能なem_perf_tableへのポインタ
nr_perf_states
    性能状態数
flags
    『em_perf_domain flags』を参照
cpus
    ドメインのCPUをカバーするcpumask。スケジューラーでのエネルギー計算中に起こりうるキャッシュミスを避け、メモリ領域の確保／解放を簡単にするための性能上の理由より

説明

CPU機器の場合、『性能ドメイン』は、性能が一緒にスケールされるCPUのグループを表します。性能ドメインのすべてのCPUは、
同じマイクロアーキテクチャでなければなりません。性能ドメインは、多くの場合、CPUFreqポリシーと1対1のマッピングを持ちます。
その他の機器の場合、cpusフィールドは未使用です。

.. code-block:: C

  int em_pd_get_efficient_state(struct em_perf_state *table, int nr_perf_states, unsigned long max_util, unsigned long pd_flags)¶
   EMから効率的な性能状態を取得

引数

.. code-block:: C

  struct em_perf_state *table
    昇順に並べられた性能状態のリスト
  int nr_perf_states
    性能状態数
  unsigned long max_util
    EMでマップする最大稼働率
  unsigned long pd_flags
    性能ドメインフラグ

説明
スケジューラーのコードから頻繁に呼び出されるため、チェック機能は実装されていません。

戻り値

max_util要件を満たすのに十分な、効率的なパフォーマンス状態ID。

返り値
効率的な性能状態のidで、max_utilの要求に大変十分に適合

.. code-block:: C

  unsigned long em_cpu_energy(struct em_perf_domain *pd, unsigned long max_util, unsigned long sum_util, unsigned long allowed_cpu_cap)
    性能ドメインのCPUで消費されるエネルギーを算出

引数

.. code-block:: C

  struct em_perf_domain *pd
    エネルギーが算出される性能ドメイン
  unsigned long max_util
    ドメインのCPU内での最高稼働率
  unsigned long sum_util
    ドメイン内のすべてのCPUの稼働率の合計
  unsigned long allowed_cpu_cap
    (熱量により)減少させた周波数を反映させた性能ドメインでのCPUの最大許容容量

説明
この関数はCPU機器にのみ使用されなければなりません。
EMがCPUタイプで、cpumaskが割り当てられているかどうかの検証はありません。
この関数はスケジューラから頻繁に呼び出されるため、チェックは行われません。

返り値
ドメインの最大稼働率を満たす容量状態を仮定した場合の、ドメインのCPUによって消費されるエネルギーの合計。

！！！！！！！ここまで(2024/3/30)！！！！！！！！！！


int em_pd_nr_perf_states(struct em_perf_domain *pd)

    Get the number of performance states of a perf. domain

Parameters

struct em_perf_domain *pd

    performance domain for which this must be done

Return

the number of performance states in the performance domain table

struct em_perf_domain *em_pd_get(struct device *dev)

    Return the performance domain for a device

Parameters

struct device *dev

    Device to find the performance domain for

Description

Returns the performance domain to which dev belongs, or NULL if it doesn't exist.

struct em_perf_domain *em_cpu_get(int cpu)

    Return the performance domain for a CPU

Parameters

int cpu

    CPU to find the performance domain for

Description

Returns the performance domain to which cpu belongs, or NULL if it doesn't exist.

int em_dev_register_perf_domain(struct device *dev, unsigned int nr_states, struct em_data_callback *cb, cpumask_t *cpus, bool microwatts)

    Register the Energy Model (EM) for a device

Parameters

struct device *dev

    Device for which the EM is to register
unsigned int nr_states

    Number of performance states to register
struct em_data_callback *cb

    Callback functions providing the data of the Energy Model
cpumask_t *cpus

    Pointer to cpumask_t, which in case of a CPU device is obligatory. It can be taken from i.e. 'policy->cpus'. For other type of devices this should be set to NULL.
bool microwatts

    Flag indicating that the power values are in micro-Watts or in some other scale. It must be set properly.

Description

Create Energy Model tables for a performance domain using the callbacks defined in cb.

The microwatts is important to set with correct value. Some kernel sub-systems might rely on this flag and check if all devices in the EM are using the same scale.

If multiple clients register the same performance domain, all but the first registration will be ignored.

Return 0 on success

void em_dev_unregister_perf_domain(struct device *dev)

    Unregister Energy Model (EM) for a device

Parameters

struct device *dev

    Device for which the EM is registered

Description

Unregister the EM for the specified dev (but not a CPU device).



3. ドライバの例
-----------------

CPUFreqフレームワークは、与えられたCPU'policy'オブジェクトのEMを登録するための専用コールバックをサポートしています。
cpufreq_driver::register_em()。
このコールバックは、指定されたドライバに対して適切に実装されていなければなりません、
なぜなら、フレームワークがセットアップ中に適切なタイミングでそれを呼び出すからである。
このセクションでは、CPUFreqドライバがエネルギー・モデル・フレームワークにパフォーマ ンス・ドメインを登録する簡単な例を示します。
(偽の)'foo'プロトコルを使用して、Energy Modelフレームワークでパフォーマンス・ドメインを登録する簡単な例を示します。
プロトコルを使用しています。このドライバは、est_power()関数を実装し、EMフレームワークに提供します。

EMフレームワーク::

  -> drivers/cpufreq/foo_cpufreq.c

  01	static int est_power(struct device *dev, unsigned long *mW,
  02			unsigned long *KHz)
  03	{
  04		long freq, power;
  05
  06		/* Use the 'foo' protocol to ceil the frequency */
  07		freq = foo_get_freq_ceil(dev, *KHz);
  08		if (freq < 0);
  09			return freq;
  10
  11		/* Estimate the power cost for the dev at the relevant freq. */
  12		power = foo_estimate_power(dev, freq);
  13		if (power < 0);
  14			return power;
  15
  16		/* Return the values to the EM framework */
  17		*mW = power;
  18		*KHz = freq;
  19
  20		return 0;
  21	}
  22
  23	static void foo_cpufreq_register_em(struct cpufreq_policy *policy)
  24	{
  25		struct em_data_callback em_cb = EM_DATA_CB(est_power);
  26		struct device *cpu_dev;
  27		int nr_opp;
  28
  29		cpu_dev = get_cpu_device(cpumask_first(policy->cpus));
  30
  31     	/* Find the number of OPPs for this policy */
  32     	nr_opp = foo_get_nr_opp(policy);
  33
  34     	/* And register the new performance domain */
  35     	em_dev_register_perf_domain(cpu_dev, nr_opp, &em_cb, policy->cpus,
  36					    true);
  37	}
  38
  39	static struct cpufreq_driver foo_cpufreq_driver = {
  40		.register_em = foo_cpufreq_register_em,
  41	};
