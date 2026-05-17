# DGX Spark 開発ワークフロー

## 目的

このリポジトリは、当面の開発ホストとして DGX Spark を使います。

MicroZig の現行安定版は Zig `0.15.1` を前提にしていますが、手元の macOS 環境では Zig `0.15.1` の `zig build` がビルドランナーのリンク段階で失敗します。DGX Spark 上の Linux 環境でビルドと一次確認を完結させ、RP2040 キーボードファームウェアの学習を先へ進めます。

## 初回セットアップ

```bash
uname -a
cat /etc/os-release
zig version
git --version
```

Zig は `0.15.1` を使います。

```bash
mkdir -p ~/works/toku345
cd ~/works/toku345
git clone <repository-url> zig-keyboard-lab
cd zig-keyboard-lab
git switch feat/microzig-rp2040-bringup
zig build
```

期待する成果物:

```text
zig-out/firmware/zig-keyboard-lab.uf2
zig-out/firmware/zig-keyboard-lab.elf
```

## 書き込み

RP2040 ボードを BOOTSEL モードで接続し、マウント先を確認します。

```bash
lsblk
```

UF2 ドライブのマウント先を確認できたら、生成物をコピーします。

```bash
cp zig-out/firmware/zig-keyboard-lab.uf2 /path/to/RPI-RP2/
sync
```

## USB / HID の確認

通常起動後の USB 認識確認:

```bash
lsusb
dmesg -w
```

HID 実装後の入力イベント確認:

```bash
cat /proc/bus/input/devices
ls -l /dev/input/by-id/
sudo evtest
```

必要に応じて `evtest` と `libinput-tools` をインストールします。

```bash
sudo apt install evtest libinput-tools
```

## 日常の開発ループ

```bash
git pull
zig build
cp zig-out/firmware/zig-keyboard-lab.uf2 /path/to/RPI-RP2/
sync
```

bring-up の間は、次の分担を基本にします。

```text
DGX Spark
  - build
  - flash
  - lsusb / dmesg / evtest による一次確認

必要に応じて別ホスト
  - 最終的なキーボード操作感の確認
```

## 補足

- 0.16 系の Zig へ移るときは、MicroZig 側の対応状況を確認してからまとめて更新します。
- 当面は、startup や linker などの足回りよりも、GPIO、キースキャン、debounce、USB HID の理解を優先します。
