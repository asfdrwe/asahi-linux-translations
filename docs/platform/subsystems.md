---
title: Apple Silicon プラットフォームサブシステム
---

2025/3/9時点の[subsystems](https://github.com/AsahiLinux/docs/blob/main/docs/platform/subsystems.md)の翻訳

---
これらのページは特定のプラットフォームサブシステムの特徴を詳述しています。機能別に大雑把に分類します。

### 一般的概要
* [Apple Siliconの紹介](introduction.md)
* [アクセラレーターエンジン](../hw/soc/accelerators.md)

### コプロセッサとアクセラレータ
* [AGX](../hw/soc/agx.md) - AppleのPowerVR派生のタイルベースの遅延レンダラ(訳注: [deferred renderer](https://ja.wikipedia.org/wiki/%E9%81%85%E5%BB%B6%E3%82%B7%E3%82%A7%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0))
* [SEP](../hw/soc/sep.md) - Secure Enclave(セキュアエンクレーブ)、Appleの暗号・生体認証・セキュリティエンジン

### プラットフォーム制御論理
* [AIC](../hw/soc/aic.md) - Apple Interrupt Controller, Apple割り込みコントローラ
* [WDT](../hw/soc/wdt.md) - Watchdog Timer, ウォッチドックタイマ
* [SMC](../hw/soc/smc.md) - System Management Controller, システム管理コントローラ
* [ASC](../hw/soc/asc.md) - AppleのMailbox風ファームウェアインタフェース

### プラットフォーム初期化及びに起動
* [Boot](../fw/boot.md)
* [MachO Boot Protocol](../fw/macho-boot-protocol.md)
* [Memory Map](../hw/soc/memmap.md)
* [SMP spinup](../hw/cpu/smp.md)
* [ADT](../fw/adt.md) (Apple DeviceTree)
* [NVRAM](../fw/nvram.md) 

### アプリケーションプロセッサ
* [ARM System Registers](../hw/cpu/system-registers.md)
* [SPRR と GXF](../hw/cpu/sprr-gxf.md)
* [CPU Debug Registers](../hw/cpu/debug-registers.md)
* [Apple Instructions](../hw/cpu/apple-instructions.md)

### I/O
* [APCIe](../hw/soc/apcie.md) (Apple PCIe controller, Apple PCIeコントローラ)
* [GPIO](../hw/soc/gpio.md)
* [USB Debug](../hw/soc/usb-debug.md)
* [USB PD](../hw/soc/usb-pd.md)
* [既製品パーティションレイアウト](stock-partition-layout.md)

### 周辺機器(Peripherals)
* [Camera](../hw/peripherals/camera.md) - Broadcom カメラ及びにISP
