2025/3/1時点の[SW-Linux-WiFi](https://github.com/AsahiLinux/docs/blob/main/docs/SW-Linux-WiFi.md)の翻訳

---
# WiFi対応
## MacOS上で[Glanzmann氏のメモ](https://tg.st/u/asahi.txt) に従って WiFi ファームウェアを入手
 * インストーラをクローン `git clone https://github.com/AsahiLinux/asahi-installer`
 * srcディレクトリに移動 `cd asahi-installer/src`。
 * ファームウェアをtarファイルに取り込む　`python3 -m firmware.wifi /usr/share/firmware/wifi /tmp/linux-firmware.tar`

## ファームウェアをインストール
 * [USBドライブ](SW-Linux-USB-drive.md)または
[nvme](SW-Linux-NVME.md)で起動したLinuxの場合 
rootfs ファームウェアのディレクトリを作成 `sudo mkdir -p /usr/lib/firmware`
 * 伸長したwifiファームウェアをインストール `sudo tar -C /usr/lib/firmware -xf firmware.tar`
 * wpasupplicantなど必要な他のネットワーク/WiFi パッケージをインストール

## WiFiを有効化
 * [wifi/take5](https://github.com/AsahiLinux/linux/tree/wifi/take5) ブランチなどのM1 WiFIに対応した 
Asahi Linux カーネルをビルドしておく必要あり
 * [USB経由のm1n1](SW-Linux.md#%E7%9B%B4%E6%8E%A5%E8%B5%B7%E5%8B%95)
でカーネルを起動する前に、以下のスクリプトを実行して、WiFi ハードウェアを有効化 `python3 ./proxyclient/experiments/pcie_enable_devices.py`
 * 他の方法も存在。私はDebian linuxで行ったのはこの方法
 * Linux カーネルが起動した後は通常のツールで WiFi デバイス (wlan0) を見ることができるようになるはず `ip a l`
 * 通常のLinuxツールでネットワーク接続を開始可能
 * 設定ファイルを編集

```
auto wlan0
iface wlan0 inet dhcp
    wpa-ssid YOUR_SSID
    wpa-psk YOUR_WIFI_PASSPHRASE
```

 * 次に、インターフェース（wlan0）を起動（注: -v => 詳細情報）`sudo ifup -v wlan0`
