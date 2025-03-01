2025/3/1時点の[SW-Ubuntu-Asahi-Godot](https://github.com/AsahiLinux/docs/blob/main/docs/SW-Ubuntu-Asahi-Godot.md)の翻訳

---
M1 Air上のUbuntu 22.10 Asahiで検証しています。

https://godotengine.org

現時点では4.xは開始時にクラッシュするようなので、Godot 3.5をビルドします。

https://docs.godotengine.org/en/latest/development/compiling/compiling_for_linuxbsd.html

```
sudo apt -y install build-essential scons \
pkg-config libx11-dev libxcursor-dev libxinerama-dev \
libgl1-mesa-dev libglu-dev libasound2-dev \
libpulse-dev libudev-dev libxi-dev libxrandr-dev

git clone https://github.com/godotengine/godot.git

git checkout remotes/origin/3.5

cd godot

scons -j$(nproc) platform=linuxbsd

cd bin

./godot.x11.tools.64
```
Asahi Linuxのハードウェアアクセラレーション版を持っていなくても、
より遅いソフトウェアGLを使用できます。
```
LIBGL_ALWAYS_SOFTWARE=1 ./godot.x11.tools.64
```
