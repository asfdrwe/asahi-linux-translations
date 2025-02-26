2025/2/17時点の[Apple Silicon Subsystems](https://github.com/AsahiLinux/docs/blob/main/docs/Apple-Silicon-Subsystems.md)の翻訳

---
これらのページは特定のプラットフォームサブシステムの特徴を詳述しています。機能別に大雑把に分類します。

### 一般的概要
* [Apple Siliconの紹介](Introduction-to-Apple-Silicon.md)
* [アクセラレーターエンジン](Accelerator-Engines.md)

### コプロセッサとアクセラレータ
* [HW:AGX](HW-AGX.md) - AppleのPowerVR派生のタイルベースの遅延レンダラ(訳注: [deferred renderer](https://ja.wikipedia.org/wiki/%E9%81%85%E5%BB%B6%E3%82%B7%E3%82%A7%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0))
* [HW:SEP](HW-SEP.md) - Secure Enclave(セキュアエンクレーブ)、Appleの暗号・生体認証・セキュリティエンジン

### プラットフォーム制御論理
* [HW:AIC](HW-AIC.md) - Apple Interrupt Controller, Apple割り込みコントローラ
* [HW:WDT](HW-WDT.md) - Watchdog Timer, ウォッチドックタイマ
* [HW:SMC](HW-SMC.md) - System Management Controller, システム管理コントローラ
* [HW:ASC](HW-ASC.md) - AppleのMailbox風ファームウェアインタフェース

### プラットフォーム初期化及びに起動
* [SW:Boot](SW-Boot.md)
* [SW:MachO Boot Protocol](SW-MachO-Boot-Protocol.md)
* [HW:Memory map](HW-Memory-map.md)
* [HW:SMP spin up](HW-SMP-spin-up.md)
* [FW:ADT](FW-ADT.md) (Apple DeviceTree)
* [SW:NVRAM](SW-NVRAM.md) 

### アプリケーションプロセッサ
* [HW:ARM System Registers](HW-ARM-System-Registers.md)
* [HW:SPRR and GXF](HW-SPRR-and-GXF.md)
* [HW:CPU debug registers](HW-CPU-debug-registers.md)
* [HW:Apple Instructions](HW-Apple-Instructions.md)

### I/O
* [HW:APCIe](HW-APCIe.md) (Apple PCIe controller, Apple PCIeコントローラ)
* [HW:GPIO](HW-GPIO.md)
* [HW:Debug USB](HW-Debug-USB.md)
* [HW:USB PD](HW-USB-PD.md)
* [SW:Storage](SW-Storage.md)

### 周辺機器(Peripherals)
* [HW:Camera](HW-Camera.md) - Broadcom カメラ及びにISP
