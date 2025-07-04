---
title: 既製品 SSD パーティションレイアウト
---

2025/6/30時点の[stock-partition-layout](https://github.com/AsahiLinux/docs/blob/main/docs/platform/stock-partition-layout.md)の翻訳

---
# まとめ

* **disk0**：メインSSD
    * **disk0s1 = disk1**: 『iBootSystemContainer』 - システム全体のブートデータ
    * **disk1s1**: 『iSCPreboot』 - ブートポリシー、システムファームウェア(NOR)バージョンメタデータ、SEPファームウェア(時々)、APチケット
    * **disk1s2**: 『xARTS』 - SEPの信頼できるストレージ
    * **disk1s3**: 『ハードウェア』 - ログ、ファクトリーデータキャッシュ、アクティベーション関連ファイル
    * **disk1s4**: 『Recovery』 - 空
    * **disk0s2 = disk3**: 『コンテナ』 - macOSインストール
    * **disk3s1**: 『System』 - OS (root filesystem、封印(seal)されている)
    * **disk3s2**: 『Preboot』 - iBoot2 (OSローダー)、iBootにロードされたファームウェア、Darwinのカーネルキャッシュ、ファームウェア、DeviceTree、その他preboot用情報
    * **disk3s3**: 『Recovery』 - ペアのRecoveryOS: iBoot2、 ファームウェア、 Darwinのカーネルキャッシュ、ラムディスクイメージ
    * **disk3s4**: 『Update』 - macOSアップデート用の一時ストレージとログ
    * **disk3s5**: 『Data』 - ユーザーデータ（root filesystem、統合）。このボリュームのUUIDはOSインストールのアイデンティティを定義
    * **disk3s6**: 『VM』 - スワップパーティション(必要時)
    * **disk0s3 = disk2**: 『RecoveryOSContainer』 - System RecoveryOS
    * **disk2s1**: 『Recovery』- 1つまたは複数の{iBoot2 (OSローダー)、Darwinのカーネルキャッシュ、ファームウェア、DeviceTree、その他のpreboot用情報}のセット
    * **disk2s2**: 『Update』 - システムのファームウェアアップデート用の一時ストレージとログ

# パーティショニング
<details>
  <summary>gdisk dump</summary>

```
Disk /dev/disk0: 61279344 sectors, 233.8 GiB
Sector size (logical): 4096 bytes
Disk identifier (GUID): 284E6CE4-CABA-4B49-8106-CE39AB7B5CD9
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 5
First usable sector is 6, last usable sector is 61279338
Partitions will be aligned on 2-sector boundaries
Total free space is 0 sectors (0 bytes)

Number  Start (sector)    End (sector)  Size       Code  Name
   1               6          128005   500.0 MiB   FFFF  iBootSystemContainer
   2          128006        59968629   228.3 GiB   AF0A  Container
   3        59968630        61279338   5.0 GiB     FFFF  RecoveryOSContainer
```
</details>

生のディスクには、標準的な保護されたMBRを持つGUIDパーティションテーブルと、3つのパーティションが含まれています。これらはmacOSの/dev/disk0s1、/dev/disk0s2、/dev/disk0s3に対応しています。

3つのパーティションはそれぞれAPFSコンテナで、いくつかのサブボリュームを含んでいます。GUIDの種類は以下の通りです:

* 69646961-6700-11AA-AA11-00306543ECAC: iBoot System Container (ASCII: 『idiag』、 diskutil: `Apple_APFS_ISC`)
* 7C3457EF-0000-11AA-AA11-00306543ECAC: APFS (diskutil: `Apple_APFS`)
* 52637672-7900-11AA-AA11-00306543ECAC: Recovery OS (ASCII reads: "Rcvry", diskutil: `Apple_APFS_Recovery`)

なお、このページで紹介されているユニークな（型でない）GUIDのほとんどは、ユーザーごとにユニークなものになります。

## disk0s1 / disk1: iBootシステムコンテナ
<details>
  <summary>diskutil情報</summary>

```
# diskutil apfs list /dev/disk1
|
+-- Container disk5 E0718E49-1903-4793-B427-FCFEC4A3E72C
    ====================================================
    APFS Container Reference:     disk1
    Size (Capacity Ceiling):      524288000 B (524.3 MB)
    Capacity In Use By Volumes:   18010112 B (18.0 MB) (3.4% used)
    Capacity Not Allocated:       506277888 B (506.3 MB) (96.6% free)
    |
    +-< Physical Store disk1 (No UUID)
    |   ------------------------------
    |   APFS Physical Store Disk:   disk0s1
    |   Size:                       524288000 B (524.3 MB)
    |
    +-> Volume disk5s1 B33E8594-382A-41EA-A9FE-6D2362B31141
    |   ---------------------------------------------------
    |   APFS Volume Disk (Role):   disk1s1 (Preboot)
    |   Name:                      iSCPreboot (Case-insensitive)
    |   Mount Point:               Not Mounted
    |   Capacity Consumed:         6213632 B (6.2 MB)
    |   Sealed:                    No
    |   FileVault:                 No
    |
    +-> Volume disk5s2 CA25E52A-3425-4232-926F-F840D359A9E2
    |   ---------------------------------------------------
    |   APFS Volume Disk (Role):   disk1s2 (xART)
    |   Name:                      xART (Case-insensitive)
    |   Mount Point:               Not Mounted
    |   Capacity Consumed:         6311936 B (6.3 MB)
    |   Sealed:                    No
    |   FileVault:                 No
    |
    +-> Volume disk5s3 0566ABD3-9EA7-46CA-90C7-CDF4DD0E94B4
    |   ---------------------------------------------------
    |   APFS Volume Disk (Role):   disk1s3 (Hardware)
    |   Name:                      Hardware (Case-insensitive)
    |   Mount Point:               Not Mounted
    |   Capacity Consumed:         507904 B (507.9 KB)
    |   Sealed:                    No
    |   FileVault:                 No
    |
    +-> Volume disk5s4 F4E0743D-D91F-410B-9569-4196540E4B8D
        ---------------------------------------------------
        APFS Volume Disk (Role):   disk1s4 (Recovery)
        Name:                      Recovery (Case-insensitive)
        Mount Point:               Not Mounted
        Capacity Consumed:         20480 B (20.5 KB)
        Sealed:                    No
        FileVault:                 No
```
</details>

これは標準的なレイアウトの最初のパーティションです。デフォルトでは `diskutil` から隠されており、以下はダンプされたイメージからの出力です。

### disk1s1 (Preboot)
<details>
 <summary>diskutil情報</summary>

```
# diskutil info /dev/disk1s1
   Device Identifier:         disk1s1
   Device Node:               /dev/disk1s1
   Whole:                     No
   Part of Whole:             disk1

   Volume Name:               iSCPreboot
   Mounted:                   Yes
   Mount Point:               /System/Volumes/iSCPreboot

   Partition Type:            41504653-0000-11AA-AA11-00306543ECAC
   File System Personality:   APFS
   Type (Bundle):             apfs
   Name (User Visible):       APFS
   Owners:                    Enabled

   OS Can Be Installed:       No
   Booter Disk:               disk1s1
   Recovery Disk:             disk1s4
   Media Type:                Generic
   Protocol:                  Apple Fabric
   SMART Status:              Verified
   Volume UUID:               19D7B85B-D5EC-41E9-8441-EEAE52D964F1
   Disk / Partition UUID:     19D7B85B-D5EC-41E9-8441-EEAE52D964F1

   Disk Size:                 524.3 MB (524288000 Bytes) (exactly 1024000 512-Byte-Units)
   Device Block Size:         4096 Bytes

   Container Total Space:     524.3 MB (524288000 Bytes) (exactly 1024000 512-Byte-Units)
   Container Free Space:      506.3 MB (506347520 Bytes) (exactly 988960 512-Byte-Units)
   Allocation Block Size:     4096 Bytes

   Media OS Use Only:         Yes
   Media Read-Only:           No
   Volume Read-Only:          No

   Device Location:           Internal
   Removable Media:           Fixed

   Solid State:               Yes
   Hardware AES Support:      Yes

   This disk is an APFS Volume.  APFS Information:
   APFS Container:            disk1
   APFS Physical Store:       disk0s1
   Fusion Drive:              No
   Encrypted:                 No
   FileVault:                 No
   Sealed:                    No
   Locked:                    No
```
</details>

Mountpoint: `/System/Volumes/iSCPreboot`

システムファームウェアのiBootが直接使用するファイルが含まれています。

ファイル:

* /(uuid)/ - OS のインストール用と 1TR の各バージョン用にそれぞれ 1 つずつ
    * LocalPolicy/ - ブートポリシー
        * (long hash).img4 - OSインストール用のローカルブートポリシー
        * (long hash).recovery.img4 - 1TR用のローカルブートポリシー
* SFR/current/ - 現在のバージョンのSFRの情報
    * apticket.der - AP チケット (?)
    * RestoreVersion.plist
    * SystemVersion.plist
    * sep-firmware.img4 - SEP ファームウェア、常に存在するわけではない
* SFR/fallback/ - 前バージョンのSFRの情報（上記と同じ構造）

外部メディアのOSを1TRで起動すると、そのOSのブートファイル（<Preboot>/(uuid)/boot以下のすべて）がPrebootボリュームから同じパスのこのパーティションにコピーされます。このようにして、iBoot 自体が起動できなくても、Apple Silicon mac は外部メディアから『起動』することができるのです。

### disk1s2 (xART)
<details>
 <summary>diskutil情報</summary>

```
# diskutil info /dev/disk1s2
   Device Identifier:         disk1s2
   Device Node:               /dev/disk1s2
   Whole:                     No
   Part of Whole:             disk1

   Volume Name:               xART
   Mounted:                   Yes
   Mount Point:               /System/Volumes/xarts

   Partition Type:            41504653-0000-11AA-AA11-00306543ECAC
   File System Personality:   APFS
   Type (Bundle):             apfs
   Name (User Visible):       APFS
   Owners:                    Enabled

   OS Can Be Installed:       No
   Booter Disk:               disk1s1
   Recovery Disk:             disk1s4
   Media Type:                Generic
   Protocol:                  Apple Fabric
   SMART Status:              Verified
   Volume UUID:               E9FD7E0B-391D-42DD-997A-15B5FE0CE73C
   Disk / Partition UUID:     E9FD7E0B-391D-42DD-997A-15B5FE0CE73C

   Disk Size:                 524.3 MB (524288000 Bytes) (exactly 1024000 512-Byte-Units)
   Device Block Size:         4096 Bytes

   Container Total Space:     524.3 MB (524288000 Bytes) (exactly 1024000 512-Byte-Units)
   Container Free Space:      506.3 MB (506347520 Bytes) (exactly 988960 512-Byte-Units)
   Allocation Block Size:     4096 Bytes

   Media OS Use Only:         Yes
   Media Read-Only:           No
   Volume Read-Only:          No

   Device Location:           Internal
   Removable Media:           Fixed

   Solid State:               Yes
   Hardware AES Support:      Yes

   This disk is an APFS Volume.  APFS Information:
   APFS Container:            disk1
   APFS Physical Store:       disk0s1
   Fusion Drive:              No
   Encrypted:                 No
   FileVault:                 No
   Sealed:                    No
   Locked:                    No
```
</details>

Mountpoint: `/System/Volumes/xarts`

ここには1つのファイル(uuid).gl。つまりSEPストレージが入っています。これを管理するには `xartutil` を使用します。

### disk1s3 (Hardware)

<details>
 <summary>diskutil情報</summary>

```
# diskutil info /dev/disk1s3
   Device Identifier:         disk1s3
   Device Node:               /dev/disk1s3
   Whole:                     No
   Part of Whole:             disk1

   Volume Name:               Hardware
   Mounted:                   Yes
   Mount Point:               /System/Volumes/Hardware

   Partition Type:            41504653-0000-11AA-AA11-00306543ECAC
   File System Personality:   APFS
   Type (Bundle):             apfs
   Name (User Visible):       APFS
   Owners:                    Enabled

   OS Can Be Installed:       No
   Booter Disk:               disk1s1
   Recovery Disk:             disk1s4
   Media Type:                Generic
   Protocol:                  Apple Fabric
   SMART Status:              Verified
   Volume UUID:               6ED3C985-5971-4874-ABCA-841BB76CC6E5
   Disk / Partition UUID:     6ED3C985-5971-4874-ABCA-841BB76CC6E5

   Disk Size:                 524.3 MB (524288000 Bytes) (exactly 1024000 512-Byte-Units)
   Device Block Size:         4096 Bytes

   Container Total Space:     524.3 MB (524288000 Bytes) (exactly 1024000 512-Byte-Units)
   Container Free Space:      506.3 MB (506347520 Bytes) (exactly 988960 512-Byte-Units)
   Allocation Block Size:     4096 Bytes

   Media OS Use Only:         Yes
   Media Read-Only:           No
   Volume Read-Only:          No

   Device Location:           Internal
   Removable Media:           Fixed

   Solid State:               Yes
   Hardware AES Support:      Yes

   This disk is an APFS Volume.  APFS Information:
   APFS Container:            disk1
   APFS Physical Store:       disk0s1
   Fusion Drive:              No
   Encrypted:                 No
   FileVault:                 No
   Sealed:                    No
   Locked:                    No
```
</details>

Mountpoint: `/System/Volumes/Hardware`

ハードウェア関連の情報とログ

* /recoverylogd/ - リカバリログ
* FactoryData/ - APチケット、デバイスのパーソナライゼーション情報
* srvo/ - センサー関連の情報？
* MobileActivation/ - アクティベーション関連データ

<details>
 <summary>diskutil情報</summary>

```
# diskutil info /dev/disk1s4
   Device Identifier:         disk1s4
   Device Node:               /dev/disk1s4
   Whole:                     No
   Part of Whole:             disk1

   Volume Name:               Recovery
   Mounted:                   No

   Partition Type:            41504653-0000-11AA-AA11-00306543ECAC
   File System Personality:   APFS
   Type (Bundle):             apfs
   Name (User Visible):       APFS
   Owners:                    Disabled

   OS Can Be Installed:       No
   Booter Disk:               disk1s1
   Recovery Disk:             disk1s4
   Media Type:                Generic
   Protocol:                  Apple Fabric
   SMART Status:              Verified
   Volume UUID:               6A900409-C5A5-47CC-84AA-F0FE24E0D629
   Disk / Partition UUID:     6A900409-C5A5-47CC-84AA-F0FE24E0D629

   Disk Size:                 524.3 MB (524288000 Bytes) (exactly 1024000 512-Byte-Units)
   Device Block Size:         4096 Bytes

   Container Total Space:     524.3 MB (524288000 Bytes) (exactly 1024000 512-Byte-Units)
   Container Free Space:      506.3 MB (506347520 Bytes) (exactly 988960 512-Byte-Units)

   Media OS Use Only:         Yes
   Media Read-Only:           No
   Volume Read-Only:          Not applicable (not mounted)

   Device Location:           Internal
   Removable Media:           Fixed

   Solid State:               Yes
   Hardware AES Support:      Yes

   This disk is an APFS Volume.  APFS Information:
   APFS Container:            disk1
   APFS Physical Store:       disk0s1
   Fusion Drive:              No
   Encrypted:                 No
   FileVault:                 No
   Sealed:                    No
   Locked:                    No
```
</details>

空。

## disk0s2 / disk3: macOSコンテナ

(注：この出力はルートボリュームの封印を解除した後のものです)

<details>
  <summary>diskutil apfs list</summary>

```
# diskutil apfs list /dev/disk3
|
+-- Container disk3 CEF76C65-8EAE-4346-A09E-AB98301B36AA
    ====================================================
    APFS Container Reference:     disk3
    Size (Capacity Ceiling):      245107195904 B (245.1 GB)
    Capacity In Use By Volumes:   61710045184 B (61.7 GB) (25.2% used)
    Capacity Not Allocated:       183397150720 B (183.4 GB) (74.8% free)
    |
    +-< Physical Store disk0s2 BDB006E1-54AA-43CD-B7FE-FF021547D51E
    |   -----------------------------------------------------------
    |   APFS Physical Store Disk:   disk0s2
    |   Size:                       245107195904 B (245.1 GB)
    |
    +-> Volume disk3s1 424FEA98-2296-48FD-8DFF-0866835572E9
    |   ---------------------------------------------------
    |   APFS Volume Disk (Role):   disk3s1 (System)
    |   Name:                      Macintosh HD (Case-insensitive)
    |   Mount Point:               /Volumes/Macintosh HD
    |   Capacity Consumed:         15053312000 B (15.1 GB)
    |   Sealed:                    Broken
    |   FileVault:                 No (Encrypted at rest)
    |
    +-> Volume disk3s2 B065CC7B-CC03-44F1-8A58-CD9AB099D57C
    |   ---------------------------------------------------
    |   APFS Volume Disk (Role):   disk3s2 (Preboot)
    |   Name:                      Preboot (Case-insensitive)
    |   Mount Point:               /Volumes/Preboot
    |   Capacity Consumed:         361050112 B (361.1 MB)
    |   Sealed:                    No
    |   FileVault:                 No
    |
    +-> Volume disk3s3 FDC764F5-0EF0-44F4-AA34-D011195207CA
    |   ---------------------------------------------------
    |   APFS Volume Disk (Role):   disk3s3 (Recovery)
    |   Name:                      Recovery (Case-insensitive)
    |   Mount Point:               Not Mounted
    |   Capacity Consumed:         939421696 B (939.4 MB)
    |   Sealed:                    No
    |   FileVault:                 No
    |
    +-> Volume disk3s5 DCBCA6BD-BFF1-4F8F-AE1A-6E937D2D4BDC
    |   ---------------------------------------------------
    |   APFS Volume Disk (Role):   disk3s5 (Data)
    |   Name:                      Data (Case-insensitive)
    |   Mount Point:               /Volumes/Data
    |   Capacity Consumed:         15799300096 B (15.8 GB)
    |   Sealed:                    No
    |   FileVault:                 No (Encrypted at rest)
    |
    +-> Volume disk3s6 D2247B63-54E9-411F-94C0-FF3FAB2A17A0
        ---------------------------------------------------
        APFS Volume Disk (Role):   disk3s6 (VM)
        Name:                      VM (Case-insensitive)
        Mount Point:               Not Mounted
        Capacity Consumed:         20480 B (20.5 KB)
        Sealed:                    No
        FileVault:                 No
```
</details>

これはOSをインストールするためのメインのmacOSコンテナです。

隠れた*Update*ボリューム(disk3s4)があります。

### disk3s1 (Macintosh HD)

<details>
 <summary>diskutil情報</summary>

```
# diskutil info /dev/disk3s1
   Device Identifier:         disk3s1
   Device Node:               /dev/disk3s1
   Whole:                     No
   Part of Whole:             disk3

   Volume Name:               Macintosh HD
   Mounted:                   Yes
   Mount Point:               /Volumes/Macintosh HD

   Partition Type:            41504653-0000-11AA-AA11-00306543ECAC
   File System Personality:   APFS
   Type (Bundle):             apfs
   Name (User Visible):       APFS
   Owners:                    Enabled

   OS Can Be Installed:       No
   Booter Disk:               disk3s2
   Recovery Disk:             disk3s3
   Media Type:                Generic
   Protocol:                  Apple Fabric
   SMART Status:              Verified
   Volume UUID:               424FEA98-2296-48FD-8DFF-0866835572E9
   Disk / Partition UUID:     424FEA98-2296-48FD-8DFF-0866835572E9

   Disk Size:                 245.1 GB (245107195904 Bytes) (exactly 478724992 512-Byte-Units)
   Device Block Size:         4096 Bytes

   Container Total Space:     245.1 GB (245107195904 Bytes) (exactly 478724992 512-Byte-Units)
   Container Free Space:      183.4 GB (183397150720 Bytes) (exactly 358197560 512-Byte-Units)
   Allocation Block Size:     4096 Bytes

   Media OS Use Only:         No
   Media Read-Only:           No
   Volume Read-Only:          Yes (read-only mount flag set)

   Device Location:           Internal
   Removable Media:           Fixed

   Solid State:               Yes
   Hardware AES Support:      Yes

   This disk is an APFS Volume.  APFS Information:
   APFS Container:            disk3
   APFS Physical Store:       disk0s2
   Fusion Drive:              No
   APFS Volume Group:         DCBCA6BD-BFF1-4F8F-AE1A-6E937D2D4BDC
   EFI Driver In macOS:       1677080005000000
   Encrypted:                 No
   FileVault:                 No
   Sealed:                    Broken
   Locked:                    No
```
</details>

OSのルートを含むメインのAPFSボリュームです。スナップショットを使用して、アトミックな更新とOSの完全性を保証するために封印付け(sealing、dm-verityに類似）を可能にしています。これは無効にすることもできますが、今のところ、ルートファイルシステムを読み書き可能な状態でマウントすることはできません。変更は、メインパーティションをマウントし変更を加え新しいスナップショットを作成しそれをblessingすることで可能になります。

スナップショットは通常 /dev/disk3s1s1 で、これは / に読み込みのみでマウントされます。名前は `com.apple.os.update<16進数の長い文字列>` のようになります。

これはデータボリュームとともにAPFSボリュームグループの一部となります。実行時にこれらはともに統合されます。

<details>
  <summary>apfs listVolumeGroups</summary>

```
+-- Container disk3 CEF76C65-8EAE-4346-A09E-AB98301B36AA
|   |
|   +-> Volume Group DCBCA6BD-BFF1-4F8F-AE1A-6E937D2D4BDC
|       =================================================
|       APFS Volume Disk (Role):   disk3s1 (System)
|       Name:                      Macintosh HD
|       Volume UUID:               424FEA98-2296-48FD-8DFF-0866835572E9
|       Capacity Consumed:         15053312000 B (15.1 GB)
|       -------------------------------------------------
|       APFS Volume Disk (Role):   disk3s5 (Data)
|       Name:                      Data
|       Volume UUID:               DCBCA6BD-BFF1-4F8F-AE1A-6E937D2D4BDC
|       Capacity Consumed:         15799300096 B (15.8 GB)
```
</details>

### disk3s2 (Preboot)

<details>
  <summary>diskutil情報</summary>
  
```
# diskutil info /dev/disk3s2
   Device Identifier:         disk3s2
   Device Node:               /dev/disk3s2
   Whole:                     No
   Part of Whole:             disk3

   Volume Name:               Preboot
   Mounted:                   Yes
   Mount Point:               /Volumes/Preboot

   Partition Type:            41504653-0000-11AA-AA11-00306543ECAC
   File System Personality:   APFS
   Type (Bundle):             apfs
   Name (User Visible):       APFS
   Owners:                    Enabled

   OS Can Be Installed:       No
   Booter Disk:               disk3s2
   Recovery Disk:             disk3s3
   Media Type:                Generic
   Protocol:                  Apple Fabric
   SMART Status:              Verified
   Volume UUID:               B065CC7B-CC03-44F1-8A58-CD9AB099D57C
   Disk / Partition UUID:     B065CC7B-CC03-44F1-8A58-CD9AB099D57C

   Disk Size:                 245.1 GB (245107195904 Bytes) (exactly 478724992 512-Byte-Units)
   Device Block Size:         4096 Bytes

   Container Total Space:     245.1 GB (245107195904 Bytes) (exactly 478724992 512-Byte-Units)
   Container Free Space:      183.4 GB (183397150720 Bytes) (exactly 358197560 512-Byte-Units)
   Allocation Block Size:     4096 Bytes

   Media OS Use Only:         No
   Media Read-Only:           No
   Volume Read-Only:          No

   Device Location:           Internal
   Removable Media:           Fixed

   Solid State:               Yes
   Hardware AES Support:      Yes

   This disk is an APFS Volume.  APFS Information:
   APFS Container:            disk3
   APFS Physical Store:       disk0s2
   Fusion Drive:              No
   Encrypted:                 No
   FileVault:                 No
   Sealed:                    No
   Locked:                    No

```
</details>

Mountpoint: /System/Volumes/Preboot

このパーティションにはカーネルを含むブート関連のデータが入っています。ルートにある唯一のディレクトリはAPFSの`Data`ボリュームのUUID/ボリュームグループのUUIDの後に名付けられています。

ファイルは次のようになります:

* /(Data UUID)/
    * usr/standalone/i386/EfiLoginUI/ - preboot UI 用の "EFI" (?) リソース
    * PreLoginData/ - uuidtext (翻訳？)やログやその他のジャンク
    * boot/
        * active - 以下のlong hashを含むファイル
        * (long hash)/
            * usr/standalone/firmware
                * iBoot.img4 - iBoot2 (OS ローダー)
                * base_system_root_hash.img4, root_hash.img4 - 封印(sealing)関連
                * FUD/ - iBootでロードされるファームウエア
                * devicetree.img4 - devicetree
            * System/Library/Caches/com.apple.kernelcaches/
                * kernelcache - Darwinのkernelcache
    * var/db/ - ユーザーリストおよび認証関連情報、preboot UI 用
    * Library/Preferences/ - ネットワークインターフェース情報、その他preboot UIのための雑多な設定
    * System/Library/
        * CoreServices/
            * boot.efi - EFI macのbooter。ここには0バイトのみ
        * Caches/com.apple.corestorage/ - FileVault関連？
    * restore/ - システムファームウェアアップデートバンドル？


### disk3s3 (Recovery)

<details>
  <summary>diskutil情報</summary>
```
# diskutil info /dev/disk3s3
   Device Identifier:         disk3s3
   Device Node:               /dev/disk3s3
   Whole:                     No
   Part of Whole:             disk3

   Volume Name:               Recovery
   Mounted:                   No

   Partition Type:            41504653-0000-11AA-AA11-00306543ECAC
   File System Personality:   APFS
   Type (Bundle):             apfs
   Name (User Visible):       APFS
   Owners:                    Disabled

   OS Can Be Installed:       No
   Booter Disk:               disk3s2
   Recovery Disk:             disk3s3
   Media Type:                Generic
   Protocol:                  Apple Fabric
   SMART Status:              Verified
   Volume UUID:               FDC764F5-0EF0-44F4-AA34-D011195207CA
   Disk / Partition UUID:     FDC764F5-0EF0-44F4-AA34-D011195207CA

   Disk Size:                 245.1 GB (245107195904 Bytes) (exactly 478724992 512-Byte-Units)
   Device Block Size:         4096 Bytes

   Container Total Space:     245.1 GB (245107195904 Bytes) (exactly 478724992 512-Byte-Units)
   Container Free Space:      183.4 GB (183397150720 Bytes) (exactly 358197560 512-Byte-Units)

   Media OS Use Only:         No
   Media Read-Only:           No
   Volume Read-Only:          Not applicable (not mounted)

   Device Location:           Internal
   Removable Media:           Fixed

   Solid State:               Yes
   Hardware AES Support:      Yes

   This disk is an APFS Volume.  APFS Information:
   APFS Container:            disk3
   APFS Physical Store:       disk0s2
   Fusion Drive:              No
   Encrypted:                 No
   FileVault:                 No
   Sealed:                    No
   Locked:                    No
```
</details>

### disk3s4 (Update)

<details>
  <summary>diskutil情報</summary>

OSのリカバリーパーティション。下記のSystem RecoveryOSとほぼ同じレイアウト/内容です。現在のデフォルトの起動OSがmacOS 12以降の場合、
起動時に電源ボタンを長押しするとそのRecoveryOSパーティションがOne True RecoveryOSとして起動します。それ以外場合、macOS 11.xでは、
代わりにSystem RecoveryOSが使用されます。

```
# diskutil info /dev/disk3s4
   Device Identifier:         disk3s4
   Device Node:               /dev/disk3s4
   Whole:                     No
   Part of Whole:             disk3

   Volume Name:               Update
   Mounted:                   Yes
   Mount Point:               /System/Volumes/Data/private/tmp/tmp-mount-spRxyx

   Partition Type:            41504653-0000-11AA-AA11-00306543ECAC
   File System Personality:   APFS
   Type (Bundle):             apfs
   Name (User Visible):       APFS
   Owners:                    Enabled

   OS Can Be Installed:       No
   Booter Disk:               disk3s2
   Recovery Disk:             disk3s3
   Media Type:                Generic
   Protocol:                  Apple Fabric
   SMART Status:              Verified
   Volume UUID:               C983230F-1974-4283-A0A8-E1F892C7C835
   Disk / Partition UUID:     C983230F-1974-4283-A0A8-E1F892C7C835

   Disk Size:                 245.1 GB (245107195904 Bytes) (exactly 478724992 512-Byte-Units)
   Device Block Size:         4096 Bytes

   Container Total Space:     245.1 GB (245107195904 Bytes) (exactly 478724992 512-Byte-Units)
   Container Free Space:      183.4 GB (183397150720 Bytes) (exactly 358197560 512-Byte-Units)
   Allocation Block Size:     4096 Bytes

   Media OS Use Only:         Yes
   Media Read-Only:           No
   Volume Read-Only:          No

   Device Location:           Internal
   Removable Media:           Fixed

   Solid State:               Yes
   Hardware AES Support:      Yes

   This disk is an APFS Volume.  APFS Information:
   APFS Container:            disk3
   APFS Physical Store:       disk0s2
   Fusion Drive:              No
   Encrypted:                 No
   FileVault:                 No
   Sealed:                    No
   Locked:                    No

```
</details>

これにはOSアップデータの一時ファイルやログなどが格納されています。

### disk3s5 (Data)

<details>
  <summary>diskutil情報</summary>

```
# diskutil info /dev/disk3s5
   Device Identifier:         disk3s5
   Device Node:               /dev/disk3s5
   Whole:                     No
   Part of Whole:             disk3

   Volume Name:               Data
   Mounted:                   Yes
   Mount Point:               /Volumes/Data

   Partition Type:            41504653-0000-11AA-AA11-00306543ECAC
   File System Personality:   APFS
   Type (Bundle):             apfs
   Name (User Visible):       APFS
   Owners:                    Enabled

   OS Can Be Installed:       Yes
   Booter Disk:               disk3s2
   Recovery Disk:             disk3s3
   Media Type:                Generic
   Protocol:                  Apple Fabric
   SMART Status:              Verified
   Volume UUID:               DCBCA6BD-BFF1-4F8F-AE1A-6E937D2D4BDC
   Disk / Partition UUID:     DCBCA6BD-BFF1-4F8F-AE1A-6E937D2D4BDC

   Disk Size:                 245.1 GB (245107195904 Bytes) (exactly 478724992 512-Byte-Units)
   Device Block Size:         4096 Bytes

   Container Total Space:     245.1 GB (245107195904 Bytes) (exactly 478724992 512-Byte-Units)
   Container Free Space:      183.4 GB (183397150720 Bytes) (exactly 358197560 512-Byte-Units)
   Allocation Block Size:     4096 Bytes

   Media OS Use Only:         No
   Media Read-Only:           No
   Volume Read-Only:          No

   Device Location:           Internal
   Removable Media:           Fixed

   Solid State:               Yes
   Hardware AES Support:      Yes

   This disk is an APFS Volume.  APFS Information:
   APFS Container:            disk3
   APFS Physical Store:       disk0s2
   Fusion Drive:              No
   APFS Volume Group:         DCBCA6BD-BFF1-4F8F-AE1A-6E937D2D4BDC
   FileVault:                 No
   Sealed:                    No
   Locked:                    No
```
</details>

Mountpoint: /System/Volumes/Data

The main user data partition, which is overlaid on top of the OS root with firmlinks.
メインのユーザーデータパーティションです。OS rootの上にfirmlinksで重ねられています。

### disk3s6 (VM)

<details>
  <summary>diskutil情報</summary>

```
# diskutil info /dev/disk3s6
   Device Identifier:         disk3s6
   Device Node:               /dev/disk3s6
   Whole:                     No
   Part of Whole:             disk3

   Volume Name:               VM
   Mounted:                   No

   Partition Type:            41504653-0000-11AA-AA11-00306543ECAC
   File System Personality:   APFS
   Type (Bundle):             apfs
   Name (User Visible):       APFS
   Owners:                    Disabled

   OS Can Be Installed:       No
   Booter Disk:               disk3s2
   Recovery Disk:             disk3s3
   Media Type:                Generic
   Protocol:                  Apple Fabric
   SMART Status:              Verified
   Volume UUID:               D2247B63-54E9-411F-94C0-FF3FAB2A17A0
   Disk / Partition UUID:     D2247B63-54E9-411F-94C0-FF3FAB2A17A0

   Disk Size:                 245.1 GB (245107195904 Bytes) (exactly 478724992 512-Byte-Units)
   Device Block Size:         4096 Bytes

   Container Total Space:     245.1 GB (245107195904 Bytes) (exactly 478724992 512-Byte-Units)
   Container Free Space:      183.4 GB (183397150720 Bytes) (exactly 358197560 512-Byte-Units)

   Media OS Use Only:         No
   Media Read-Only:           No
   Volume Read-Only:          Not applicable (not mounted)

   Device Location:           Internal
   Removable Media:           Fixed

   Solid State:               Yes
   Hardware AES Support:      Yes

   This disk is an APFS Volume.  APFS Information:
   APFS Container:            disk3
   APFS Physical Store:       disk0s2
   Fusion Drive:              No
   Encrypted:                 No
   FileVault:                 No
   Sealed:                    No
   Locked:                    No
```
</details>

スワップパーティション。必要なければ空になるようです。

## disk0s3: System RecoveryOS

<details>
  <summary>diskutil apfs list</summary>

```
# diskutil apfs list /dev/disk2
|
+-- Container disk5 160EFEBB-B539-42EE-800D-2FE4723FB25F
    ====================================================
    APFS Container Reference:     disk5
    Size (Capacity Ceiling):      5368664064 B (5.4 GB)
    Capacity In Use By Volumes:   1919692800 B (1.9 GB) (35.8% used)
    Capacity Not Allocated:       3448971264 B (3.4 GB) (64.2% free)
    |
    +-< Physical Store disk2 (No UUID)
    |   ------------------------------
    |   APFS Physical Store Disk:   disk0s3
    |   Size:                       5368664064 B (5.4 GB)
    |
    +-> Volume disk2s1 DDD6CA1C-2FAC-4990-B20E-89F5323DAABB
        ---------------------------------------------------
        APFS Volume Disk (Role):   disk2s1 (Recovery)
        Name:                      Recovery (Case-insensitive)
        Mount Point:               Not Mounted
        Capacity Consumed:         1899917312 B (1.9 GB)
        Sealed:                    No
        FileVault:                 No
```
</details>

これはSystem RecoveryOS パーティションで、1つまたは2つのバージョンのRecoveryOSが入っています。

隠れた*Update*ボリューム(disk2s2)があります。

## disk2s1: Recovery

<details>
  <summary>diskutil情報</summary>

```
# diskutil info /dev/disk2s1
   Device Identifier:         disk2s1
   Device Node:               /dev/disk2s1
   Whole:                     No
   Part of Whole:             disk2

   Volume Name:               Recovery
   Mounted:                   Yes
   Mount Point:               /System/Volumes/Data/private/tmp/SystemRecovery

   Partition Type:            41504653-0000-11AA-AA11-00306543ECAC
   File System Personality:   APFS
   Type (Bundle):             apfs
   Name (User Visible):       APFS
   Owners:                    Enabled

   OS Can Be Installed:       No
   Recovery Disk:             disk2s1
   Media Type:                Generic
   Protocol:                  Apple Fabric
   SMART Status:              Verified
   Volume UUID:               67B148BE-39ED-493A-99AE-1C1D28247C54
   Disk / Partition UUID:     67B148BE-39ED-493A-99AE-1C1D28247C54

   Disk Size:                 5.4 GB (5368664064 Bytes) (exactly 10485672 512-Byte-Units)
   Device Block Size:         4096 Bytes

   Container Total Space:     5.4 GB (5368664064 Bytes) (exactly 10485672 512-Byte-Units)
   Container Free Space:      2.5 GB (2500202496 Bytes) (exactly 4883208 512-Byte-Units)
   Allocation Block Size:     4096 Bytes

   Media OS Use Only:         Yes
   Media Read-Only:           No
   Volume Read-Only:          Yes (read-only mount flag set)

   Device Location:           Internal
   Removable Media:           Fixed

   Solid State:               Yes
   Hardware AES Support:      Yes

   This disk is an APFS Volume.  APFS Information:
   APFS Container:            disk2
   APFS Physical Store:       disk0s4
   Fusion Drive:              No
   Encrypted:                 No
   FileVault:                 No
   Sealed:                    No
   Locked:                    No
```
</details>

これはRecovertyOSの1つまたは複数のバージョンを含んでいます。

ファイル:

* /(unknown UUID)/boot/(long hash)/.
    * usr/standalone/firmware
        * arm64eBaseSystem.dmg - rootfs ramdisk image (単一のボリュームと単一のAPFSコンテナを持つGPTディスクイメージ)
        * iBoot.img4 - iBoot2 (OS ローダー)
        * base_system_root_hash.img4, root_hash.img4 - 封印(sealing)関連
        * sep-firmware.img4 - SEPのファームウェア
        * FUD/ - iBootでロードされるファームウェア
        * devicetree.img4 - devicetree
    * System/Library/Caches/com.apple.kernelcaches/
        * kernelcache - Darwinのkernelcache

RecoveryOSがロードされると、このパーティションは全体的にtmpfsにコピーされ、arm64eBaseSystem.dmgが添付されます。
        
## disk2s2: Update

<details>
  <summary>diskutil情報</summary>

```
   Device Identifier:         disk2s2
   Device Node:               /dev/disk2s2
   Whole:                     No
   Part of Whole:             disk2

   Volume Name:               Update
   Mounted:                   Yes
   Mount Point:               /Volumes/Update

   Partition Type:            41504653-0000-11AA-AA11-00306543ECAC
   File System Personality:   APFS
   Type (Bundle):             apfs
   Name (User Visible):       APFS
   Owners:                    Enabled

   OS Can Be Installed:       No
   Recovery Disk:             disk2s1
   Media Type:                Generic
   Protocol:                  Apple Fabric
   SMART Status:              Verified
   Volume UUID:               812D7035-B392-4B1C-AC3E-131A4DFA2726
   Disk / Partition UUID:     812D7035-B392-4B1C-AC3E-131A4DFA2726

   Disk Size:                 5.4 GB (5368664064 Bytes) (exactly 10485672 512-Byte-Units)
   Device Block Size:         4096 Bytes

   Container Total Space:     5.4 GB (5368664064 Bytes) (exactly 10485672 512-Byte-Units)
   Container Free Space:      2.5 GB (2500202496 Bytes) (exactly 4883208 512-Byte-Units)
   Allocation Block Size:     4096 Bytes

   Media OS Use Only:         Yes
   Media Read-Only:           No
   Volume Read-Only:          No

   Device Location:           Internal
   Removable Media:           Fixed

   Solid State:               Yes
   Hardware AES Support:      Yes

   This disk is an APFS Volume.  APFS Information:
   APFS Container:            disk2
   APFS Physical Store:       disk0s4
   Fusion Drive:              No
   Encrypted:                 No
   FileVault:                 No
   Sealed:                    No
   Locked:                    No
```
</details>

一時的なシステムファームウェアの更新データとログが入っています。
