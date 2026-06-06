# DGX Spark / macOS 開発ワークフロー

## 目的

このリポジトリは、当面の開発ホストとして DGX Spark を使います。macOS でも Xcode / SDK の組み合わせを調整すれば Zig `0.15.1` で `zig build` できるため、編集とビルド確認は macOS、書き込みや USB / HID の一次確認は DGX Spark という分担も可能です。

MicroZig の現行安定版は Zig `0.15.1` を前提にしています。Xcode 26.4 / macOS SDK 26 系では Zig `0.15.1` の `zig build` がビルドランナーのリンク段階で失敗しましたが、Xcode 16.4 / macOS SDK 15.5 に切り替えると macOS 上でも build できることを確認しました。

## 初回セットアップ

### DGX Spark

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

### macOS

Zig は `0.15.1` を使います。Xcode 26.4 / macOS SDK 26 系ではリンクエラーになるため、Xcode 16.4 / macOS SDK 15.5 の組み合わせで確認します。

```bash
xcodebuild -version
xcrun --show-sdk-version
zig version
zig build
```

確認済みの組み合わせ:

```text
Xcode 16.4
macOS SDK 15.5
Zig 0.15.1
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
macOS
  - edit
  - build

DGX Spark
  - build
  - flash
  - lsusb / dmesg / evtest による一次確認

必要に応じて別ホスト
  - 最終的なキーボード操作感の確認
```

## 補足

- Xcode 26.4 / macOS SDK 26 系では Zig `0.15.1` のビルドランナーリンクに失敗したため、macOS で build する場合は Xcode 16.4 / macOS SDK 15.5 を使います。
- 0.16 系の Zig へ移るときは、MicroZig 側の対応状況を確認してからまとめて更新します。
- 当面は、startup や linker などの足回りよりも、GPIO、キースキャン、debounce、USB HID の理解を優先します。
