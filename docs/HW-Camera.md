2025/3/1時点の[HW-Camera](https://github.com/AsahiLinux/docs/blob/main/docs/HW-Camera.md)の翻訳

---
# ISP（訳注:Image Signal Processor、画像処理プロセッサ）

本情報は、Macbook Pro M1 2020 ISPに基づきます。他のデバイスの場合は異なる可能性があります。

## センサーの種類 -> ISPのバージョン 

| センサーの種類｜ISPのバージョン
|---	|---	|
| 0xee | 0xf |
| 0x54 | 0xb |
| 0x55 | 0xb |
| 0x56 | 0xb |
| 0x57 | 0xb |
| 0xe4 | 0xb |
| 0xe5 | 0xb |
| 0xe0 | 0xb |
| 0xe1 | 0xb |
| 0xe2 | 0xb |
| 0xe3 | 0xb |
| 0x1 | 0xb |
| 0x2 | 0xb |
| 0x3 | 0xb |
| 0xe8 | 0xc |
| 0xe7 | 0xc |
| 0xe9 | 0xc |
| 0xea | 0xc |
| 0x1a | 0xc |
| 0x1b | 0xc |
| 0xb | 0xb |
| 0xc | 0xb |
| 0xf5 | 0xb |
| 0xd | 0xc |
| 0xe | 0xc |
| 0x13 | 0xc |
| 0x14 | 0xc |
| 0x15 | 0xc |
| 0x16 | 0xc |
| 0x9a | 0xc |
| 0x5 | 0xc |
| 0x6 | 0xc |
| 0x7 | 0xc |
| 0x18 | 0xb |
| 0x19 | 0xb |
| 0x1c | 0xf |
| 0x1d | 0xf |

## レジスタ

- ISPレジスタ(regs[0])。チップのリビジョンにより、異なるレジスタが使用される

| レジスタ名 | ISP バージョン | ISP リビジョン? | オフセット | 注 |
|---	|---	|---	|---	|---	|
| ISP_REVISION | 0xF | N/A | 0x1800000 | 31:0 bits are used.<br/> ([31:0] == 0x1) => 0x15a <br/> ([31:0] != 0x1001) => 0x15b |
|  | 0xC | N/A | 0x1800000 | 31:0 bits使用 <br/> ([31:0] == 0x90) => 0x14a <br/> ([31:0] != 0x1090) => 0x14b <br/> ([31:0] == 0x3091) => 0x14c |
|  | 0xB | N/A | 0x1800000 | 31:0 bits使用 <br/> ([31:0] == 0x90) => 0x13a <br/> ([31:0] != 0x3091) => 0x13c |
|---	|---	|---	|---	|---	|
| SENSOR_REF_CLOCK0 | 0xF  	| != 0x15b | 0x24c41f0 | Sensor Ref Clock 0 |
|  | 0xF  	| == 0x15b | 0x24c41f4 | Sensor Ref Clock 0 |
|  | 0xC  	| != 0x14c | 0x24c41d0 | Sensor Ref Clock 0 |
|  | 0xC  	| == 0x14c | 0x2104190 | Sensor Ref Clock 0 |
|  | Other     | *        | 0x2104190 | Sensor Ref Clock 0 |
| SENSOR_REF_CLOCK1 | 0xF  	| != 0x15b | 0x24c41f4 | Sensor Ref Clock 1 |
|  | 0xF  	| == 0x15b | 0x24c41f8 | Sensor Ref Clock 1 |
|  | 0xC  	| != 0x14c | 0x24c41d4 | Sensor Ref Clock 1 |
|  | 0xC  	| == 0x14c | 0x2104194 | Sensor Ref Clock 1 |
|  | Other     | *        | 0x2104194 | Sensor Ref Clock 1 |
| SENSOR_REF_CLOCK2 | 0xF  	| != 0x15b | 0x24c41f8 | Sensor Ref Clock 2 |
|  | 0xF  	| == 0x15b | 0x24c41fc | Sensor Ref Clock 2 |
|  | 0xC  	| != 0x14c | 0x24c41d8 | Sensor Ref Clock 2 |
|  | 0xC  	| == 0x14c | 0x2104198 | Sensor Ref Clock 2 |
|  | Other     | *        | 0x2104198 | Sensor Ref Clock 2 |
| SENSOR_REF_CLOCK3 | 0xF  	| != 0x15b | 0x24c41fc | Sensor Ref Clock 3 |
|  | 0xF  	| == 0x15b | 0x24c4200 | Sensor Ref Clock 3 |
|  | 0xC  	| != 0x14c | 0x24c41dc | Sensor Ref Clock 3 |
|  | 0xC  	| == 0x14c | 0x210419c | Sensor Ref Clock 3 |
|---	|---	|---	|---	|---	|
| POWER_ON? | 0xF | * | 0x24a0080 | 電源ON中は0x1にセット |
| POWER_ON? | 0xC | != 0x14c | 0x24a0080 | 電源ON中は0x1にセット |
| POWER_ON? | Other | * | 0x20e0080 | 電源ON中は0x1にセット |
|---	|---	|---	|---	|---	|
| UNKNOWN0 | 0xF | * | 0x24c41d0 |  |
|  | 0xC | != 0x14c | 0x24c41b0 |  |
|  | Other | Other | 0x2104170 |  |
| UNKNOWN1 | 0xF | * | 0x24c41d4 |  |
|  | 0xC | != 0x14c | 0x24c41b4 |  |
|  | Other | Other | 0x2104174 |  |
| UNKNOWN2 | 0xF | * | 0x24c41d8 |  |
|  | 0xC | != 0x14c | 0x24c41b8 |  |
|  | Other | Other | 0x2104178 |  |
| UNKNOWN3 | 0xF | * | 0x24c41dc |  |
|  | 0xC | != 0x14c | 0x24c41bc |  |
|  | Other | Other | 0x210417c |  |
| UNKNOWN4 | 0xF | * | 0x24c41e0 |  |
|  | 0xC | != 0x14c | 0x24c41c0 |  |
|  | Other | Other | 0x2104180 |  |
| UNKNOWN5 | 0xF | * | 0x24c41e4 |  |
|  | 0xC | != 0x14c | 0x24c41c4 |  |
|  | Other | Other | 0x2104184 |  |
| UNKNOWN6 | 0xF | * | 0x24c41e8 |  |
|  | 0xC | != 0x14c | 0x24c41c8 |  |
|  | Other | Other | 0x2104188 |  |
| UNKNOWN7 | 0xF | * | 0x24c41ec |  |
|  | 0xC | != 0x14c | 0x24c41cc |  |
|  | 0xC | * | 0x210418c |  |
|  | 0xB | * | 0x188 |  |
| SMBUS_REG_MTXFIFO0 | * | * | 0x2110000 | I2C Channel 0
| SMBUS_REG_MRXFIFO0 | * | * | 0x2110004 | 
| SMBUS_REG_UNKNOWN0_1 | * | * | 0x2110008 |
| SMBUS_REG_UNKNOWN0_2* | * | * | 0x211000c |
| SMBUS_REG_UNKNOWN0_3* | * | * | 0x2110010 | 
| SMBUS_REG_SMSTA0 | * | * | 0x2110014 | |
| SMBUS_REG_UNKNOWN0_4 | * | * | 0x2110018 |
| SMBUS_REG_CTL0 | * | * | 0x211001c |
| SMBUS_REG_UNKNOWN0_5 | * | * | 0x2110030 |
| SMBUS_REG_UNKNOWN0_6 | * | * | 0x2110034 |
| SMBUS_REG_UNKNOWN0_7 | * | * | 0x211003c |
|---	|---	|---	|---	|---	|
| SMBUS_REG_MTXFIFO1 | * | * | 0x2111000 | I2C Channel 1
| SMBUS_REG_MRXFIFO1 | * | * | 0x2111004 | 
| SMBUS_REG_UNKNOWN1_1 | * | * | 0x2111008 |
| SMBUS_REG_UNKNOWN1_2* | * | * | 0x211100c |
| SMBUS_REG_UNKNOWN1_3* | * | * | 0x2111010 | 
| SMBUS_REG_SMSTA1 | * | * | 0x2111014 | |
| SMBUS_REG_UNKNOWN1_4 | * | * | 0x2111018 |
| SMBUS_REG_CTL1 | * | * | 0x211101c |
| SMBUS_REG_UNKNOWN1_5 | * | * | 0x2111030 |
| SMBUS_REG_UNKNOWN1_6 | * | * | 0x2111034 |
| SMBUS_REG_UNKNOWN1_7 | * | * | 0x211103c |
|---	|---	|---	|---	|---	|
| SMBUS_REG_MTXFIFO2 | * | * | 0x2112000 | I2C Channel 2
| SMBUS_REG_MRXFIFO2 | * | * | 0x2112004 | 
| SMBUS_REG_UNKNOWN2_1 | * | * | 0x2112008 |
| SMBUS_REG_UNKNOWN2_2* | * | * | 0x211200c |
| SMBUS_REG_UNKNOWN2_3* | * | * | 0x2112010 | 
| SMBUS_REG_SMSTA2 | * | * | 0x2112014 | |
| SMBUS_REG_UNKNOWN2_4 | * | * | 0x2112018 |
| SMBUS_REG_CTL2 | * | * | 0x211201c |
| SMBUS_REG_UNKNOWN2_5 | * | * | 0x2112030 |
| SMBUS_REG_UNKNOWN2_6 | * | * | 0x2112034 |
| SMBUS_REG_UNKNOWN2_7 | * | * | 0x211203c |
|---	|---	|---	|---	|---	|
| SMBUS_REG_MTXFIFO3 | * | * | 0x2113000 | I2C Channel 3
| SMBUS_REG_MRXFIFO3 | * | * | 0x2113004 | 
| SMBUS_REG_UNKNOWN3_1 | * | * | 0x2113008 |
| SMBUS_REG_UNKNOWN3_2* | * | * | 0x211300c |
| SMBUS_REG_UNKNOWN3_3* | * | * | 0x2113010 | 
| SMBUS_REG_SMSTA3 | * | * | 0x2113014 | |
| SMBUS_REG_UNKNOWN3_4 | * | * | 0x2113018 |
| SMBUS_REG_CTL3 | * | * | 0x211301c |
| SMBUS_REG_UNKNOWN3_5 | * | * | 0x2113030 |
| SMBUS_REG_UNKNOWN3_6 | * | * | 0x2113034 |
| SMBUS_REG_UNKNOWN3_7 | * | * | 0x211303c |
|---	|---	|---	|---	|---	|
| REG_DPE_UNKNOWN0 | * | * | 0x2504000 |
| REG_DPE_UNKNOWN1 | * | * | 0x2508000 |
|---	|---	|---	|---	|---	|
| REG_UNKNOWN | * | * | 0x1050000 |

    - Unknown (offset: 0x1400044)
    - Unknown (offset: 0x1aa801c)

- PS registers (regs[1]): チップのリビジョンにより異なるレジスタ範囲が使われる。ほとんどのチップリビジョンでは0x4000-0x4060を使用するらしい
    - Unknown (offset: 0x00)
    - Unknown (offset: 0x08)
    - Unknown (offset: 0x10)
    - Unknown (offset: 0x18) 
    - Unknown (offset: 0x20)
    - Unknown (offset: 0x28)
    - Unknown (offset: 0x30)
    - Unknown (offset: 0x38)
    - Unknown (offset: 0x40) 
    - Unknown (offset: 0x328)
    - Unknown (offset: 0x3d8)      
    - Unknown (offset: 0x4000) 
    - Unknown (offset: 0x4008)
    - Unknown (offset: 0x4010)
    - Unknown (offset: 0x4018)
    - Unknown (offset: 0x4020)
    - Unknown (offset: 0x4028)
    - Unknown (offset: 0x4030)
    - Unknown (offset: 0x4038)
    - Unknown (offset: 0x4040)
    - Unknown (offset: 0x4048)
    - Unknown (offset: 0x4050)
    - Unknown (offset: 0x4058)
    - Unknown (offset: 0x4060)
    - Unknown (offset: 0x1c3e8)
    - Unknown (offset: 0x1c400)
    - Unknown (offset: 0x1c418)   
- SOC SPMI CSR registers (regs[2])
    - Unknown (offset: 0x80a0) ( 1 << 2^(1..5) に初期化)
    - Unknown (offset: 0x80a4) (値は次の値とORされるらしい: 0x4000000, 0x8000000, 0x1, 0x2)
    - Unknown (offset: 0x28) (0 に初期化)
    - Unknown (offset: 0x90) (1 に初期化)
    - Unknown (offset: 0x40) (0x4000000 に初期化)

- SOC SPMI0 registers (regs[3])
    - Unknown (offset: 0x80a0) (1 << 2^(1..5) に初期化)
    - Unknown (offset: 0x80a4) (値は次の値とORされるらしい: 0x4000000, 0x8000000, 0x1, 0x2)
    - Unknown (offset: 0x28) (0 に初期化)
    - Unknown (offset: 0x90) (1 に初期化)
    - Unknown (offset: 0x40) (0x4000000 に初期化)

- SOC SPMI1 registers (regs[4])
    - Unknown (offset: 0x80a0) (1 << 2^(1..5) に初期化)
    - Unknown (offset: 0x80a4) (値は次の値とORされるらしい: 0x4000000, 0x8000000, 0x1, 0x2)
    - Unknown (offset: 0x28) (0 に初期化)
    - Unknown (offset: 0x90) (1 に初期化)
    - Unknown (offset: 0x40) (0x4000000 に初期化)
