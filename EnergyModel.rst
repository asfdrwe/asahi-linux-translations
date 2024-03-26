`Energy Model of devices <https://docs.kernel.org/power/energy-model.html>`_  の非公式日本語訳です。
ライセンスは原文のライセンスGPL-2.0に従います。

訳注:

* まだ作業中(2024/3/26)
* APIについてkernelヘッダの埋め込みはそのまま貼り付け
* カーネル文書へのリンクで日本語訳があるものは日本語訳へのリンクに変更

=======================
Energy Model of devices
=======================

1. 概要
-------

エネルギーモデル（EM）フレームワークは、さまざまな性能レベルでデバイスが消費する電力を知るドライバと、
その情報を使用してエネルギーを考慮した判断を行うカーネルの間のインターフェイスとして機能します。

デバイスが消費する電力に関する情報源はプラットフォームによって大きく異なります。
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

『上級』EMの登録
~~~~~~~~~~~~~~

『上級』EMはドライバーがより正確なパワーモデルを提供することができるようになっていることから名付けられました。
(『単純』EMの場合のように）フレームワークで実装された数式に限定されるものではありません。
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

！！！！2024/3/26ここまで！！！！！

DTを利用するEMの登録
~~~~~~~~~~~~~~~~~

EMはOPPフレームワークを使用して登録することもでき、DTの情報
"operating-points-v2 "に登録することもできる。DT の各 OPP エントリは、プロパティ
"opp-microwatt "には、マイクロワット電力値が含まれる。このOPP DTプロパティ
このOPP DTプロパティにより、プラットフォームは、総電力（静的＋動的）を反映するEM電力値
(静的＋動的）を反映する EM 電力値を登録することができる。これらの電力値は
実験や測定から直接得られるかもしれない。

『人工』EM の登録
~~~~~~~~~~~~~~


各性能状態の電力値に関する詳細な知識が不足しているドライバーのために、カスタム・コールバックを提供するオプションがあります。
各パフォーマンス・ステートのパワー値に関する詳細な知識が不足しているドライバーのために、カスタム・コールバックを提供するオプションがあります。コールバック
.get_cost() はオプションで、EASによって使用される「コスト」値を提供します。
これは、CPUタイプ間の相対効率に関する情報のみを提供するプラットフォー
これは、CPUタイプ間の相対効率に関する情報のみを提供するプラットフォームにとって有用である。
抽象的な消費電力モデルを作成することができる。しかし、抽象的な電力モデルであっても
しかし、抽象的な電力モデルであっても、入力電力値のサイズ制限を考慮すると、適合させるのが難しい場合がある。
.get_cost()は、CPUの効率を反映する'cost'値を提供することができる。
を提供することができる。これにより
を提供することができる。
とは異なる関係を持つEAS情報を提供することができる。このようなプラットフォームにEMを登録するには
ドライバは、フラグ'microwatts'を0に設定し、.get_power()コールバックを提供し、.get_cost()
.get_cost()コールバックを提供しなければならない。EMフレームワークは、このようなプラットフォーム
を適切に処理する。EM_PERF_DOMAIN_ARTIFICIALフラグが設定されます。
フラグが設定される。EMを使用している他のフレームワークでは、このフラグをテストし、適切に扱うために特別な注意を払う必要があります。
を使用している他のフレームワークは、このフラグをテストし、適切に扱うために特別な注意を払う必要があります。

シンプルな'EMの登録
~~~~~~~~~~~~~~~~~~~~~~~~~~~

単純な'EMは、フレームワーク・ヘルパー関数
cpufreq_register_em_with_opp()を使って登録される。これは以下のようなパワーモデルを実装しています。
数学式::

	Power = C * V^2 * f

このメソッドを使って登録されたEMは、実際のデバイスの物理を正しく反映しないかもしれません。
例えば、静的消費電力（リーク）が重要な場合などである。


2.3 性能ドメインへのアクセス
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

エネルギー・モデルへのアクセスを提供する 2 つの API 関数があります：
em_cpu_get()はCPU IDを引数にとり、em_pd_get()はデバイスポインタを引数にとります。
ポインタを引数にとります。どのインターフェイスを使用するかはサブシステムに依存します。
どちらのインターフェイスを使用するかはサブシステムによって異なりますが、CPU デバイスの場合は、どちらの関数も同じ性能領域を返します。
を返します。

CPUのエネルギー・モデルに興味のあるサブシステムは
em_cpu_get() API を使用して取得できます。エネルギー・モデル・テーブルは、性能ドメインの作成時に一度割り当てられ、メモリ上に保持されます。
エネルギー・モデル・テーブルは、パフォーマンス・ドメインの作成時に一度割り当てられ、そのままメモリに保持されます。

パフォーマンス・ドメインが消費するエネルギーは
em_cpu_energy() APIを使用して推定できます。この推定は、schedutil
CPUデバイスの場合、CPUfreqガバナーが使用されていると仮定して計算されます。現在のところ、この計算は
は提供されていない。

上記のAPIに関する詳細は、``<linux/energy_model.h>`` または2.4節

2.4 本APIの説明詳細
^^^^^^^^^^^^^^^^^

struct em_perf_state

    Performance state of a performance domain

Definition:

struct em_perf_state {
    unsigned long frequency;
    unsigned long power;
    unsigned long cost;
    unsigned long flags;
};

Members

frequency

    The frequency in KHz, for consistency with CPUFreq
power

    The power consumed at this level (by 1 CPU or by a registered device). It can be a total power: static and dynamic.
cost

    The cost coefficient associated with this level, used during energy calculation. Equal to: power * max_frequency / frequency
flags

    see "em_perf_state flags" description below.

struct em_perf_domain

    Performance domain

Definition:

struct em_perf_domain {
    struct em_perf_state *table;
    int nr_perf_states;
    unsigned long flags;
    unsigned long cpus[];
};

Members

table

    List of performance states, in ascending order
nr_perf_states

    Number of performance states
flags

    See "em_perf_domain flags"
cpus

    Cpumask covering the CPUs of the domain. It's here for performance reasons to avoid potential cache misses during energy calculations in the scheduler and simplifies allocating/freeing that memory region.

Description

In case of CPU device, a "performance domain" represents a group of CPUs whose performance is scaled together. All CPUs of a performance domain must have the same micro-architecture. Performance domains often have a 1-to-1 mapping with CPUFreq policies. In case of other devices the cpus field is unused.

struct em_perf_state *em_pd_get_efficient_state(struct em_perf_domain *pd, unsigned long freq)

    Get an efficient performance state from the EM

Parameters

struct em_perf_domain *pd

    Performance domain for which we want an efficient frequency
unsigned long freq

    Frequency to map with the EM

Description

It is called from the scheduler code quite frequently and as a consequence doesn't implement any check.

Return

An efficient performance state, high enough to meet freq requirement.

unsigned long em_cpu_energy(struct em_perf_domain *pd, unsigned long max_util, unsigned long sum_util, unsigned long allowed_cpu_cap)

    Estimates the energy consumed by the CPUs of a performance domain

Parameters

struct em_perf_domain *pd

    performance domain for which energy has to be estimated
unsigned long max_util

    highest utilization among CPUs of the domain
unsigned long sum_util

    sum of the utilization of all CPUs in the domain
unsigned long allowed_cpu_cap

    maximum allowed CPU capacity for the pd, which might reflect reduced frequency (due to thermal)

Description

This function must be used only for CPU devices. There is no validation, i.e. if the EM is a CPU type and has cpumask allocated. It is called from the scheduler code quite frequently and that is why there is not checks.

Return

the sum of the energy consumed by the CPUs of the domain assuming a capacity state satisfying the max utilization of the domain.

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
