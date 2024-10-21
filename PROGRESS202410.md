[AAA gaming on Asahi Linux](https://asahilinux.org/2024/10/aaa-gaming-on-asahi-linux/)の非公式日本語訳です。

訳注: DeepLの結果を貼っただけ(2024/10/19)

---
# Asahi Linux での AAA ゲーミング

- [前回](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS202406.md)

M1でのLinuxでのゲームがここにあります！x86エミュレーションとWindows互換性を備えたVulkan 1.3ドライバを統合したAsahiゲームプレイツールキットをリリースできることを嬉しく思います。さらに、ボーナスとして、OpenCL 3.0に準拠しています。
Asahi Linuxは現在、このハードウェアのための唯一の準拠OpenGL®、 OpenCL™、 Vulkan®ドライバを出荷しています。ゲームに関しては...本日のリリースはアルファ版ですが、Controlはよく動きます！
![image1](https://asahilinux.org/img/blog/2024/10/Control-small.avif)

## インストール
まず、Fedora Asahi Remixをインストールします。インストールしたら、dnf upgrade --refresh && reboot で最新のドライバを入手する。その後、dnf install steamして遊ぶ。すべてのM1/M2シリーズが動作するが、ほとんどのゲームはエミュレーションのオーバーヘッドのため16GBのメモリを必要とする。

## スタック
ゲームは通常、DirectXでレンダリングされるx86 Windowsバイナリですが、私たちのターゲットはVulkanを搭載したArm Linuxです。それぞれの違いを処理する必要がある：
FEXはArm上でx86をエミュレートします。
WineはWindowsをLinuxに変換します。
DXVKと vkd3d-protonはDirectXをVulkanに変換する。
ページサイズという曲者もある。オペレーティングシステムは、固定サイズの「ページ」でメモリを割り当てます。アプリケーションが、システムが使用するよりも小さいページを期待すると、割り当てのアライメントが不十分なために壊れてしまいます。x86は4Kページを想定しているが、Appleシステムは16Kページを使っている。
Linuxはプロセス間でページ・サイズを混在させることはできないが、異なるページ・サイズの別のArm Linuxカーネルを仮想化することはできる。そのため、GPUやゲームコントローラーなどのデバイスを経由して、muvmを使用した小さな仮想マシン内でゲームを実行する。システムが16Kなのでハードウェアは満足し、仮想マシンが4Kなのでゲームは満足し、Fallout 4をプレイできるので満足する。

![image2](https://asahilinux.org/img/blog/2024/10/Fallout4-small.avif)

## Vulkan
DirectXを翻訳するには、多くの拡張機能を持つVulkan 1.3が必要だからだ。4月に、私はAppleハードウェア用の唯一のVulkan 1.3ドライバであるHoneykrispを書いた。その後、DXVKサポートを追加しました。いくつかの新機能を見てみよう。

## テッセレーション
テッセレーションは、『ウィッチャー3』のようなゲームがジオメトリを生成することを可能にします。M1にはハードウェア・テッセレーションがありますが、DirectX、Vulkan、OpenGLでは制限されすぎています。その代わりに、今日のXDC2024での講演で詳しく説明するように、難解なコンピュート・シェーダーを使ってテッセレーションを行う必要があります。

## ジオメトリ・シェーダー
ジオメトリ・シェーダーは、ジオメトリを生成するための、より古く、より粗雑な方法です。テッセレーションと同様に、M1にはジオメトリ・シェーダーのハードウェアがないため、コンピュート・シェーダーでエミュレートします。それは速いか？いいえ。しかし、ジオメトリー・シェーダーは、デスクトップGPUでも遅いのです。高速である必要はなく、Ghostrunnerのようなゲームに十分な速度があればいいのです。

![image3](https://asahilinux.org/2024/10/aaa-gaming-on-asahi-linux/)

## ロバストネスの強化
「ロバスト性」は、ハードウェアをクラッシュさせることなく、アプリケーションのシェーダが境界外のバッファにアクセスすることを許可します。OpenGLとVulkanでは、アウトオブバウンズのロードは任意の要素を返す可能性があり、アウトオブバウンズのストアはバッファを破損する可能性があります。私たちのOpenGLドライバは、M1上での効率的なロバスト性のために、この定義を悪用しています。
ゲームによっては、より強力な保証が必要です。DirectXでは、アウトオブバウンズロードはゼロを返し、アウトオブバウンズストアは無視されます。したがって、DXVKは、ロバスト性を強化するVulkan拡張であるVK_EXT_robustness2を必要とします。
以前と同様に、比較と選択命令でロバストネスを実装します。ナイーブな実装では、ロードされたインデックスとバッファサイズを比較し、範囲外の場合はゼロを選択します。しかし、GPUのロードはベクトルであり、演算はスカラーです。ページフォールトを無効にしたとしても、ロードごとに最大4つの比較と選択が必要になります。

```
load R, buffer, index * 16
ulesel R[0], index, size, R[0], 0
ulesel R[1], index, size, R[1], 0
ulesel R[2], index, size, R[2], 0
ulesel R[3], index, size, R[3], 0
```

仮想メモリの魔術を使って64ギガバイトのゼロを確保するのだ。32ビットのインデックスに16を掛けると64ギガバイトに収まるので、この領域へのインデックスはすべてゼロをロードする。アウトオブバウンズのロードでは、インデックスを保持したまま、バッファアドレスを予約アドレスに置き換えるだけです。64ビットのアドレスを置き換えるには、32ビットのコンペア＆セレクトを2回行うだけです。

```
ulesel buffer.lo, index, size, buffer.lo, RESERVED.lo
ulesel buffer.hi, index, size, buffer.hi, RESERVED.hi
load R, buffer, index * 16
```

4命令ではなく2命令。

## 次のステップ
Honeykrispの次のステップはスパーステクスチャリングです。アルファ版では、Cyberpunk 2077のようなスパースを必要としないDX12ゲームがすでに動作しています。

![image4](https://asahilinux.org/img/blog/2024/10/Cyberpunk2077-small.avif)


多くのゲームはプレイ可能だが、新しいAAAタイトルはまだ60fpsを達成していない。正しさが第一。次にパフォーマンスが向上する。Hollow Knightのようなインディーズゲームはフルスピードで動きます。

![image5](https://asahilinux.org/img/blog/2024/10/HollowKnight-small.avif)


ゲーム以外にも、このスタックをベースにした汎用x86エミュレーションを追加する予定です。詳しくはFAQをご覧ください。

本日のアルファ版は、その一端を示すものだ。最終形ではありませんが、「1.0 」に向けて作業している間、Portal 2を楽しむには十分です。

![image6](https://asahilinux.org/img/blog/2024/10/Portal2-small.avif)

## 謝辞
本作品は、以下の方々の多大なご協力を得て、何年もかけて完成させたものである。

- Alyssa Rosenzweig
- Asahi Lina
- chaos_princess
- Davide Cavalca
- Dougall Johnson
- Ella Stanforth
- Faith Ekstrand
- Janne Grunau
- Karol Herbst
- marcan
- Mary Guillemard
- Neal Gompa
- Sergio López
- TellowKrinkle
- Teoh Han Hui
- Rob Clark
- Ryan Houdek

...さらに、Linux、Mesa、Wine、FEXの各プロジェクトにまたがる何百人もの開発者たち。今日のリリースは、オープンソースの魔法のおかげです。
この魔法を楽しんでほしい。
ゲームを楽しんでください。

#### Alyssa Rosenzweig - 2024-10-10
