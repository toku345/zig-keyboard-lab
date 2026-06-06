# ADR 0002: 当面の開発ホストとして DGX Spark を使う

## Status

Accepted

## Context

MicroZig の現行安定版に合わせるため、このリポジトリでは Zig `0.15.1` を使う必要があります。

手元の macOS 環境では、Xcode 26.4 / macOS SDK 26 系と Zig `0.15.1` の組み合わせで、`zig build` がビルドランナーのリンク段階で失敗しました。これはリポジトリ固有のコードではなく、Apple SDK と Zig `0.15.1` の組み合わせに起因する問題です。

その後、Xcode 16.4 / macOS SDK 15.5 に切り替えることで、macOS 上でも Zig `0.15.1` の `zig build` が成功することを確認しました。

一方で、ローカルネットワーク上の DGX Spark には SSH 接続でき、RP2040 ボードも直接接続できます。DGX Spark 上であれば、ビルド、UF2 書き込み、USB 認識確認、HID イベント確認まで一台で完結できます。

## Decision

MicroZig が Zig `0.15.1` を要求する間は、DGX Spark を主な bring-up 用ホストとして使います。

DGX Spark 上で `zig build`、UF2 書き込み、`lsusb` / `dmesg` / `evtest` による一次確認を行います。macOS 環境は Xcode 16.4 / macOS SDK 15.5 を使う場合に限り、編集と `zig build` の確認にも使えます。

## Consequences

- Xcode 16.4 / macOS SDK 15.5 では、macOS 上でも Zig `0.15.1` の build を確認できます。
- Xcode 26.4 / macOS SDK 26 系では、Zig `0.15.1` の build は避けます。
- build、flash、一次確認のループを Linux 上で閉じられます。
- 開発ホストが二つになるため、作業手順を明文化しておく必要があります。
- MicroZig が Zig `0.16.0` 以降へ追従した後は、開発ホストの見直し余地があります。
