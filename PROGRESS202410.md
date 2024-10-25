[AAA gaming on Asahi Linux](https://asahilinux.org/2024/10/aaa-gaming-on-asahi-linux/)の非公式日本語訳です。

訳注: DeepLの結果を貼っただけ(2024/10/19)

---
# Asahi Linux での AAA ゲーミング

- [前回](https://github.com/asfdrwe/asahi-linux-translations/blob/main/PROGRESS202406.md)

M1でのLinuxでのゲームがここにあります！Asahi ゲームプレイツールキットを公開できることを嬉しく思います。Asahi ゲームプレイツールキットは
x86エミュレーションとWindows互換性を備え、 Vulkan 1.3 ドライバを統合しています。追加ボーナスとして、OpenCL 3.0に準拠しています。

Asahi Linuxは現在、このハードウェア向けで[OpenGL®](https://www.khronos.org/conformance/adopters/conformant-products/opengl#submission_3470)、
[OpenCL™](https://www.khronos.org/conformance/adopters/conformant-products/opencl#submission_433)、 [Vulkan®](https://www.khronos.org/conformance/adopters/conformant-products#submission_7910)に準拠する唯一のドライバを公開しています。ゲームに関して...本日のリリースはアルファ版ですが、
[Control](https://store.steampowered.com/app/870780/Control_Ultimate_Edition/)はよく動きます！

![control](https://asahilinux.org/img/blog/2024/10/Control-small.avif)

## インストール
まず、Fedora Asahi Remix をインストールします。インストール後に、`dnf upgrade --refresh && reboot` で最新のドライバを入手してください。
その後、`dnf install steam`し、プレイしてください。すべての M1/M2 シリーズで動作しますが、ほとんどのゲームはエミュレーションのオーバーヘッドの
ため 16GB のメモリを必要とします。

## スタック(訳注: 使わているソフトウェア基盤)
ゲームは通常、DirectX でレンダリングされる x86 Windows バイナリですが、私たちの標的はVulkanを搭載したArm Linuxです。それぞれの違いを処理する必要があります。

- [FEX](https://fex-emu.com/) が Arm 上で x86 をエミュレート
- [Wine](https://www.winehq.org/) が Windows を Linux に変換
- [DXVK](https://github.com/doitsujin/dxvk) と [vkd3d-proton](https://github.com/HansKristian-Work/vkd3d-proton) が DirectX を Vulkan に変換

曲者がいます。ページサイズです。オペレーティングシステムは、固定サイズの『ページ』単位でメモリを割り当てます。アプリケーションが、システムが使用するよりも
小さいページを想定すると、割り当てのアライメントが不十分なために壊れてしまいます。これは問題です。x86 は 4K ページを想定していますが、Apple システムは
16K　ページを使用しています。

Linuxはプロセス間でページサイズを混在させることはできませんが、異なるページサイズの別の Arm Linux カーネルを仮想化することは``できます``。そのため、
GPU やゲームコントローラーなどのデバイスを経由して、[muvm](https://github.com/AsahiLinux/muvm)を使用した小さな仮想マシン内でゲームを実行します。
システムが 16K なのでハードウェアは満足し、仮想マシンが 4K なのでゲームは満足し、[Fallout 4](https://store.steampowered.com/app/377160/Fallout_4/)を
プレイできるのでみんな満足します。

![Fallout4](https://asahilinux.org/img/blog/2024/10/Fallout4-small.avif)

## Vulkan
最後の1枚は成熟段階のVulkanドライバーです。DirectXを変換処理するには、多くの拡張機能を持つVulkan 1.3が必要だからです。4月にAppleハードウェア用の
唯一の Vulkan 1.3 ドライバである [Honeykrisp](https://rosenzweig.io/blog/vk13-on-the-m1-in-1-month.html) を書きました。それから、
DXVKサポートを追加しました。いくつかの新機能を見てみましょう。

### テッセレーション
テッセレーション(訳注:[解説](https://ja.wikipedia.org/wiki/%E3%83%86%E3%83%83%E3%82%BB%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3))は、
[Witcher 3](https://store.steampowered.com/app/292030/The_Witcher_3_Wild_Hunt/) のようなゲームが
ジオメトリを生成することを可能にします。M1にはハードウェア・テッセレーションがありますが、DirectX、Vulkan、OpenGLで使うには制限されすぎています。
その代わりに、[2024年10月10日のXDC2024での講演](https://www.youtube.com/live/pDsksRBLXPk)で詳しく説明するように、難解なコンピュート・シェーダー
(訳注:[解説](https://ja.wikipedia.org/wiki/%E3%82%B7%E3%82%A7%E3%83%BC%E3%83%80%E3%83%BC#%E3%82%B3%E3%83%B3%E3%83%94%E3%83%A5%E3%83%BC%E3%83%88%E3%82%B7%E3%82%A7%E3%83%BC%E3%83%80%E3%83%BC))を
使ってテッセレーションを行う必要があります。

![Witcher3](https://asahilinux.org/img/blog/2024/10/Witcher3-small.avif)

### ジオメトリ・シェーダー
ジオメトリ・シェーダーは(訳注:[解説](https://ja.wikipedia.org/wiki/%E3%82%B7%E3%82%A7%E3%83%BC%E3%83%80%E3%83%BC#%E3%82%B8%E3%82%AA%E3%83%A1%E3%83%88%E3%83%AA%E3%82%B7%E3%82%A7%E3%83%BC%E3%83%80%E3%83%BC)、
ジオメトリを生成するための、より古く、より荒っぽい手法です。テッセレーションと同様に、M1にはジオメトリ・シェーダーのハードウェアがないため、コンピュート・シェーダーで
エミュレートします。速いですか？いいえ。しかし、ジオメトリー・シェーダーは、[デスクトップ GPU でも](http://www.joshbarczak.com/blog/?p=667)遅いのです。
高速である必要はなく、[Ghostrunner](https://store.steampowered.com/app/1139900/Ghostrunner/) のようなゲームに十分な速度があればいいのです。

![Ghostrunner](https://asahilinux.org/img/blog/2024/10/Ghostrunner-small.avif)

### 堅牢性の強化
『堅牢性(robustness)』は、ハードウェアをクラッシュさせることなく、アプリケーションのシェーダーが境界外(out-of-bounds)のバッファにアクセスすることを許可します。
OpenGL と Vulkan では、境界外ロードで任意の要素を返す可能性があり、境界外ストアはバッファを破損する可能性があります。私たちのOpenGLドライバは、
M1上での効率的な堅牢性のために、[堅牢性の定義を悪用](https://rosenzweig.io/blog/conformant-gl46-on-the-m1.html)しています。

ゲームによってはより強力な保証が必要です。DirectX では、境界外ロードはゼロを返し、境界外ストアは無視されます。したがって、DXVK は、堅牢性を強化する Vulkan 拡張で
ある`VK_EXT_robustness2`を必要とします。

以前と同様に、比較選択命令で堅牢性を実装します。ナイーブな実装では、ロードされたインデックスとバッファサイズを比較し、範囲外の場合はゼロを選択します。しかし、
GPU のロードはベクトルですが、演算はスカラーです。ページフォールトを無効にしたとしても、ロードごとに最大4つの比較と選択が必要になります。

```
load R, buffer, index * 16
ulesel R[0], index, size, R[0], 0
ulesel R[1], index, size, R[1], 0
ulesel R[2], index, size, R[2], 0
ulesel R[3], index, size, R[3], 0
```

トリックがあります。仮想メモリの魔術を使って64ギガバイトのゼロを確保します。32ビットのインデックスに16を掛けると64ギガバイトに収まるので、この領域への
インデックスはすべてゼロをロードします。境界外ロードでは、インデックスを保持したまま、バッファアドレスを予約アドレスに置き換えてしまいます。
64ビットのアドレスを置き換えるには、32ビットの比較選択を2回行うだけです。

```
ulesel buffer.lo, index, size, buffer.lo, RESERVED.lo
ulesel buffer.hi, index, size, buffer.hi, RESERVED.hi
load R, buffer, index * 16
```

4命令ではなく2命令で済みます。

## 次のステップ
Honeykrispの次のステップはスパース・テクスチャ(訳注:[解説](https://docs.unity3d.com/ja/2019.4/Manual/SparseTextures.html)です。スパース・テクスチャにより
より多くのDX12ゲームが開放されます。アルファ版ではすでに[Cyberpunk 2077](https://store.steampowered.com/app/1091500/Cyberpunk_2077/)のようなスパースを必要としないDX12ゲームが動作しています。

![Cyberpunk2077](https://asahilinux.org/img/blog/2024/10/Cyberpunk2077-small.avif)

多くのゲームはプレイ可能ですが、新しいAAAタイトルはまだ60fpsを達成していません。正確性が第一です。性能向上は次です。
[Hollow Knight](https://store.steampowered.com/app/367520/Hollow_Knight/)のようなインディーズゲームはフルスピードで動きます。

![HollowKnight](https://asahilinux.org/img/blog/2024/10/HollowKnight-small.avif)

ゲーム以外にも、このスタックをベースにした汎用 x86 エミュレーションを追加する予定です。詳しくは[FAQをご覧ください](https://docs.fedoraproject.org/en-US/fedora-asahi-remix/x86-support/)。

本日のアルファ版はその一端を示すものです。最終形ではありませんが、『1.0』に向けて作業している間、[Portal 2](https://store.steampowered.com/app/620/Portal_2/)を楽しむには十分です。

![Portal2](https://asahilinux.org/img/blog/2024/10/Portal2-small.avif)

## 謝辞
今回の成果は、以下の方々の多大なご協力を得て、何年もかけて完成させたものです...

- [Alyssa Rosenzweig氏](https://rosenzweig.io/)
- [Asahi Lina氏](https://lina.yt/me)
- [chaos_princess氏](https://social.treehouse.systems/@chaos_princess)
- [Davide Cavalca氏](https://github.com/davide125)
- [Dougall Johnson氏](https://mastodon.social/@dougall)
- [Ella Stanforth氏](https://ella.gay/)
- [Faith Ekstrand氏](https://www.gfxstrand.net/faith/welcome/)
- [Janne Grunau氏](https://social.treehouse.systems/@janne)
- [Karol Herbst氏](https://chaos.social/@karolherbst)
- [marcan氏](https://social.treehouse.systems/@marcan)
- [Mary Guillemard氏](https://mary.zone/)
- [Neal Gompa氏](https://neal.gompa.dev/)
- [Sergio López氏](https://sinrega.org/)
- [TellowKrinkle氏](https://github.com/TellowKrinkle)
- [Teoh Han Hui氏](https://github.com/teohhanhui)
- [Rob Clark氏](https://mastodon.gamedev.place/@robclark)
- [Ryan Houdek氏](https://github.com/sonicadvance1)

...加えて、Linux、Mesa、Wine、FEXの各プロジェクトにまたがる何百人もの開発者のみなさま。今日のリリースは、オープンソースの魔法のおかげです。

この魔法を楽しんでほしいです。

ゲームを楽しみましょう。

#### Alyssa Rosenzweig - 2024-10-10
