2025/3/1時点の[perf-on-M1-systems](https://github.com/AsahiLinux/docs/blob/main/docs/perf-on-M1-systems.md)の翻訳

訳注: perfはLinuxでよく使われている性能解析ツールです

---
`perf stat` はベアメタル上の Asahi Linux で、Apple 専用のパフォーマンスカウンタ (カーネル内で対応している) を使って動作します。

これはbig.LITTLEのシステムなので、いくつかの注意点があります。コアタイプを跨ぐプロファイリングは混乱を招くので、タスクは1つの
コアタイプに固定する必要があります。また、パフォーマンスカウンタはコアタイプごとに異なることがあるので、カウンタを指定するときは
コアタイプを明示的に指定する必要があります。

例えば、`echo`をコア4（すべてのM1バリエーションに存在するFirestorm bigコア）でプロファイルする場合:

```
$ taskset -c 4 perf stat -e apple_firestorm_pmu/cycles/ -e apple_firestorm_pmu/instructions/ echo

Performance counter stats for 'echo':

        116,874      apple_firestorm_pmu/cycles/u                                   
        181,687      apple_firestorm_pmu/instructions/u                                   

    0.000352959 seconds time elapsed

    0.000000000 seconds user
    0.000357000 seconds sys
```

コア0(すべてのM1バリエーションに存在するIcestorm LITTLEコア)の場合:

```
$ taskset -c 0 perf stat -e apple_icestorm_pmu/cycles/ -e apple_icestorm_pmu/instructions/ echo

Performance counter stats for 'echo':

        185,564      apple_icestorm_pmu/cycles/u                                   
        181,669      apple_icestorm_pmu/instructions/u                                   

    0.000491126 seconds time elapsed

    0.000510000 seconds user
    0.000000000 seconds sys
```

この例では、IcestormのIPCが1程度であるのに対し、FirestormのIPCは1.6程度であることに注意してください。技術的には、タスクを固定せずすべてのカウンターを指
定することもできます。そうすれば、各コアタイプで費やされたサイクル/命令の独立したカウントが得られます（そのタイプのすべてのコアにわたって集計されます）。
これが実際に誰かにとって有用であるかどうかは不明です。

これはVMでは決して機能しません。なぜならば、Appleは標準的なARMパフォーマンスカウンタに対応しておらず（カスタムPMUを使用）、VMゲストに独自の機能を
公開していないからです（KVM/qemuですら同様）。しかし、ベアメタルでは動作します（m1n1ハイパーバイザーでも動作するが、ベンチマークすることはおそらく
非常に悪い考え）。

また、生のイベントIDを指定することで、サイクル/インストラクション以外のカウンタを取得することができます。これらは現在Linuxではフレンドリーな名前に
マッピングされていませんが、 `r<hex ID>` を使うことができます。Dougallはそれらの束を
[こちらで](https://github.com/dougallj/applecpu/blob/main/timer-hacks/bench.py#L85) 文書化しています。
例えば、DCACHE_LOAD_MISSを取得する場合:

```
$ taskset -c 4 perf stat -e apple_firestorm_pmu/rbf/ echo  

Performance counter stats for 'echo':

            3,136      apple_firestorm_pmu/rbf/u                                   

    0.000288042 seconds time elapsed

    0.000301000 seconds user
    0.000000000 seconds sys
```

なお、big/LITTLEコア間ですべて同じになるとは限りませんし、新しいタイプのコアでは異なるものもあります。
