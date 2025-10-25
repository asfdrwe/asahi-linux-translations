---
title: Windows 11 VM の作成方法
---

2025/10/25時点の[How to create a Windows 11 VM](https://github.com/AsahiLinux/docs/blob/main/docs/sw/windows-11-vm.md)の翻訳

訳注: 
- 原文の見出しレベルが変なので修正
- Microsoft の英語版サイトへのリンクは対応する日本語サイトへのリンクに変更

---

# はじめに
Fedora Asahi Remix に Windows 11 をインストールするための簡単なガイドです！
Windows から必要なものがあっても、ハードウェアを切り替えたりベアメタル Windows マシンを利用したりせずに、Windows 11 インスタンスをインストール、設定、管理する方法を説明します。

貢献者: aykevl 氏及びに Davide Calvaca 氏

!!! note
   `dnf install @virtualization` はこのガイドの範囲外で、対応していません。
このガイド執筆時点では未対応で破損しています。virt-manager、libvirtd などが含まれます。
詳細は https://github.com/AsahiLinux/docs/pull/206#issuecomment-3274648383 を参照してください。このような実験を行う場合は注意が必要です。

## 手順
1. `dnf install qemu` コマンドで QEMU をインストール
2. デスクトップ上または任意の場所に新しいディレクトリを作成し、任意の名前を付ける。例えば、 `windows11` という名前で `mkdir windows11` で作成。適切なターミナルアプリケーションを使用するか、デスクトップ上で右クリックしてディレクトリを作成
3. `cd windows11` でそのディレクトリに移動
4. Windows 11 の ISO をダウンロード。適切に ARM64 用の Windows 11 Professional ビルドを [こちら](https://www.microsoft.com/ja-JP/software-download/windows11arm64) から取得。任意のターミナルアプリケーションで `mv` を使用して、`windows-11.iso` のような適切な名前にリネーム
5. その ISO とともに、virtio-drivers を使用してマシンのパフォーマンスを向上させるのが望ましい。[こちら](https://github.com/virtio-win/kvm-guest-drivers-windows/wiki/Driver-installation) からダウンロードし、`win11-virtio.iso` のように適切にリネーム
6. `qemu-img create -f qcow2 win11.qcow2 25G` コマンドを使用して Windows 11 VM の仮想ディスクを作成。ディスク容量を希望のサイズに調整。25GB はプレースホルダー
7. ここで Windows 11 VM の起動スクリプトを作成。`win11.sh` という名前のファイルを作成し、`chmod +x win11.sh` で実行可能にする。内容は以下の通り：

``` {.md .copy}
 #!/bin/sh

performance_cores=$(awk '
  /^processor/ { proc=$3 } 
  /^CPU part/ {
    if ($4 == "0x023" || $4 == "0x025" || $4 == "0x029" || $4 == "0x033" || $4 == "0x035" || $4 == "0x039")
      procs=procs ? procs","proc : proc
  } END { print procs }
' /proc/cpuinfo)

taskset -c "$performance_cores" \
  qemu-system-aarch64 \
    -display sdl,gl=on \
    -cpu host \
    -M virt \
    -enable-kvm \
    -m 2G \
    -smp 2 \
    -bios /usr/share/edk2/aarch64/QEMU_EFI.fd \
    -hda win11.qcow2 \
    -device qemu-xhci \
    -device ramfb \
    -device usb-storage,drive=install \
    -drive if=none,id=install,format=raw,media=cdrom,file=windows-11-iot.iso \
    -device usb-storage,drive=virtio-drivers \
    -drive if=none,id=virtio-drivers,format=raw,media=cdrom,file=virtio-win.iso \
    -object rng-random,filename=/dev/urandom,id=rng0 \
    -device virtio-rng-pci,rng=rng0 \
    -audio driver=pipewire,model=virtio \
    -device usb-kbd \
    -device usb-tablet \
    -nic user,model=virtio-net-pci
```

!!! note
    windows-11.iso や virtio-win.iso とは異なる名前でファイル名を付けた場合は、ファイル引数を調整してください。

8. そして `./win11.sh` を実行。QEMU ウィンドウが表示。いくつかのブート画面の後、「Press any key to boot from CD or DVD…」が表示。Windows をブートするために任意のキーを押す（UEFI コンソールに入らないよう素早く押す）
9. Windows がブートし、Windows 11 のセットアップ画面が表示されるはず。簡単なはずだが、1 つ注意点があり：
『Windows 11 のインストール場所を選んでください。』画面で、ドライブが表示されない。これを修正するには、『ドライバーの読み込み』をクリックし、『参照』をクリックして virtio-win ドライブを展開し、viostor → w11 → ARM64 を選択。OK をクリック。表示された『Red Hat VirtIO SCSI controller』を選択してインストールをクリック
10. インストール中に VM が数回再起動。何もせずにそのままにしておく
11. インストール後、最初のブートウィザード（場所、キーボードなどの設定）に入る
『ネットワークに接続しましょう』画面で、『ドライバーのインストール』をクリックし、virtio-win ドライブを開いて NetKVM → w11 → ARM64 に移動し、「フォルダーの選択」をクリック。数秒待つと、ネットワークアダプターが表示。これで続行可能に(訳注: 確認していませんがこの段階ではなく 9 の段階でネットワークのドライバもインストールする必要があるかもしれません)
12. インストールが完了すると、動作する Windows 11 がインストールされているはず！

## インストール後
通常通り VM をシャットダウンしてください（win11.sh スクリプトをただ終了しないでください、Windows を怖がらせないように）。再起動するには、単に `./win11.sh` を再度実行するだけです。

インストール後、2 つの ISO を削除できます。win11.sh スクリプトから以下の 4 行を削除します：
``` md
-device usb-storage,drive=install \
    -drive if=none,id=install,format=raw,media=cdrom,file=windows-11-iot.iso \
    -device usb-storage,drive=virtio-drivers \
    -drive if=none,id=virtio-drivers,format=raw,media=cdrom,file=virtio-win.iso \
```

## SSH 手順
1. QEMU コマンドラインを変更し、`-nic user,model=virtio-net-pci` を `-device virtio-net-pci,netdev=net0 -netdev user,id=net0,hostfwd=tcp::5555-:22` に置き換えて、VM のポート 22 をホストのポート 5555 に転送（任意のポートを選択可能）
2. 『オプション機能』を開始し、『機能を表示』をクリックして OpenSSH Server を検索し、インストール（これには時間がかかる）
3. Windows ファイアウォールでポート 22 を許可します。[この StackOverflow 投稿](https://stackoverflow.com/questions/68594235/allow-ssh-protocol-through-win-10-firewall) を参照
4. 『サービス』を起動し、『OpenSSH SSH Server』を探して開始（右クリック → 開始）。また、右クリックしてプロパティに移動し、起動の種類を自動に設定してブート時に常に開始するように設定
5. これで `ssh yourusername@localhost -p 5555` を使用して Windows システムに SSH で接続可能に！
6. パブリックキー認証でシステムにログインしたい場合、パブリックキーを C:\ProgramData\ssh\administrators_authorized_keys に配置（アカウントが管理者アカウントの場合）

