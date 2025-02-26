2025/2/17時点の[Delete an Installation](https://github.com/AsahiLinux/docs/blob/main/docs/Delete-an-Installation.md)の翻訳

訳注: 
- Asahi Linux内のページへのリンクは対応する日本語訳に置き換え
- このページに書かれている操作はディスクを変更するので、間違えるとデータが消えてしまいます。気を付けて操作してください
- 出力例が少しおかしい気がするので(macOSが入っているのが1.では『disk3』なのに3.以降では『disk4』になっています)、出力を良くみて、どういう操作を実行しようとしているか意味を理解して行ってください

---
# これはデフォルトのバニラ Asahi インストール用です。カスタムパーティション設定には適用されません。

## Diskutil GUI を使用しないでください

1. Asahi インストーラが作成する 3 つのパーティションを調べてください

インストーラのスクリプトを実行し、ディスク情報が表示されたら終了します。

```
curl https://alx.sh | sh
```

ディスク情報の例：

```
Partitions in system disk (disk0):
  1: APFS [Macintosh HD] (380.00 GB, 6 volumes)
    OS: [B ] [Macintosh HD] macOS v12.3 [disk3s1, D44D4ED9-B162-4542-BF50-9470C7AFDA43]
  2: APFS [Asahi Linux] (2.50 GB, 4 volumes)
    OS: [ *] [Asahi Linux] incomplete install (macOS 12.3 stub) [disk4s2, 53F853CF-4851-4E82-933C-2AAEB247B372]
  3: EFI (500.17 MB)
  4: Linux Filesystem (54.19 GB)
  5: (free space: 57.19 GB)
  6: APFS (System Recovery) (5.37 GB, 2 volumes)
    OS: [  ] recoveryOS v12.3 [Primary recoveryOS]

  [B ] = Booted OS, [R ] = Booted recovery, [? ] = Unknown
  [ *] = Default boot volume
```

Asahiがインストールするパーティションは`Asahi Linux`、`EFI`、`Linux Filesystem`です。
Asahi Linux の行の横にある`disk#s#`をメモしてください。

(訳注: 『    OS: [ *] [Asahi Linux] incomplete install (macOS 12.3 stub) [disk4s2, 53F853CF-4851-4E82-933C-2AAEB247B372]』の右から2番目の『disk4s2』です』)
  
2. 『Asahi APFS container』を削除します

```
diskutil apfs deleteContainer disk<num-here>
```

(訳注: 『\<num-here\>』は『Asahi APFS container』の番号(上の例なら『disk4s2』の4なので)なので、『disk\<num-here\>』は『disk4』です)

3. EFIパーティションとLinuxファイルシステムパーティションのパーティション番号を調べます

```
diskutil list
```

出力例：

```
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *500.3 GB   disk0
   1:             Apple_APFS_ISC Container disk1         524.3 MB   disk0s1
   2:                 Apple_APFS Container disk4         362.6 GB   disk0s2
                    (free space)                         2.5 GB     -
   3:                        EFI EFI - FEDOR             524.3 MB   disk0s4
   4:           Linux Filesystem                         1.1 GB     disk0s5
   5:           Linux Filesystem                         127.7 GB   disk0s6
   6:        Apple_APFS_Recovery Container disk3         5.4 GB     disk0s7

/dev/disk4 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +362.6 GB   disk4
                                 Physical Store disk0s2
   1:                APFS Volume Macintosh HD            11.2 GB    disk4s1
   2:              APFS Snapshot com.apple.os.update-... 11.2 GB    disk4s1s1
   3:                APFS Volume Preboot                 6.9 GB     disk4s2
   4:                APFS Volume Recovery                1.0 GB     disk4s3
   5:                APFS Volume Data                    287.6 GB   disk4s5
   6:                APFS Volume VM                      20.5 KB    disk4s6
```

4. EFIパーティションとLinuxファイルシステムパーティションを削除します

```
diskutil eraseVolume free free disk<num-here>s<num-here>
diskutil eraseVolume free free disk<num-here>s<-other-num-here>
```

(訳注: 上の例の場合EFIパーティションが『disk0s4』、Linuxファイルシステムパーティションが『disk0s5』と『disk0s6』です)

## macOSのサイズを変更してディスクを再び埋めます
1. MacOSのインストールに対応する論理ディスク番号を取得します

```
diskutil list
```
出力例： 

```
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *500.3 GB   disk0
   1:             Apple_APFS_ISC Container disk1         524.3 MB   disk0s1
   2:                 Apple_APFS Container disk4         362.6 GB   disk0s2
                    (free space)                         2.5 GB     -
   3:                        EFI EFI - FEDOR             524.3 MB   disk0s4
   4:           Linux Filesystem                         1.1 GB     disk0s5
   5:           Linux Filesystem                         127.7 GB   disk0s6
   6:        Apple_APFS_Recovery Container disk3         5.4 GB     disk0s7

/dev/disk4 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +362.6 GB   disk4
                                 Physical Store disk0s2
   1:                APFS Volume Macintosh HD            11.2 GB    disk4s1
   2:              APFS Snapshot com.apple.os.update-... 11.2 GB    disk4s1s1
   3:                APFS Volume Preboot                 6.9 GB     disk4s2
   4:                APFS Volume Recovery                1.0 GB     disk4s3
   5:                APFS Volume Data                    287.6 GB   disk4s5
   6:                APFS Volume VM                      20.5 KB    disk4s6
```

上記の例では、`/dev/disk# (synthesized)`行の下にMacintosh HDが表示されている行がメモすべきディスクです。
この例では、論理ディスク『disk4』を拡張します。

2. Resize - 空き領域を埋めるために論理ボリュームを拡張します。

```
diskutil apfs resizeContainer disk<num-here> 0
```

（訳注: 上の例の場合『disk<num-here>』は『disk4』です)

### エラー？もっと背景情報が必要ですか？
[パーティション分割のチートシート](Partitioning-cheatsheet.md)
