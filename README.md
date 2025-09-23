# os

［作って学ぶ］OSのしくみⅠ ──メモリ管理、マルチタスク、ハードウェア制御の実装 [技術評論社](https://gihyo.jp/book/2025/978-4-297-14859-1)

## 概要

このプロジェクトは、x86_64アーキテクチャ向けのUEFIブートローダーです。本書の内容に従って、段階的にOSを実装していきます。

## 必要な環境

- Rust (nightly版)
- QEMU (x86_64エミュレータ)
- OVMF (UEFI ファームウェア)
- Task (タスクランナー)

## セットアップ

### 1. Rustのnightly版をインストール

```bash
rustup default nightly
rustup target add x86_64-unknown-uefi
rustup component add rust-src
```

### 2. Taskをインストール

```bash
# macOS
brew install go-task/tap/go-task

# その他のOSは https://taskfile.dev/installation/ 参照
```

### 3. OVMFファームウェアを配置

`third_party/ovmf/RELEASEX64_OVMF.fd` にOVMFファームウェアを配置してください。

## ビルド方法

### 基本的なコマンド

#### デバッグビルド

```bash
task build
```

#### リリースビルド

```bash
task build:release
```

#### ビルド＆実行（デバッグ版）

```bash
task run
```

#### ファイル監視＆自動実行

```bash
task run:watch
```

#### クリーンアップ

```bash
cargo clean
```

## アーキテクチャについて

このプロジェクトは **x86_64-unknown-uefi** ターゲット専用に設定されています。ARMアーキテクチャではなく、x86_64でのビルドが必要です。

### 設定ファイル

- `.cargo/config.toml`: x86_64-unknown-uefiターゲットのランナー設定
- `Taskfile.yml`: ビルドタスクの定義
- `rust-toolchain.toml`: nightlyコンパイラの指定
- `scripts/launch_qemu.sh`: QEMUランチャースクリプト

## トラブルシューティング

### Taskコマンドが見つからない場合

```bash
# macOSでのインストール
brew install go-task/tap/go-task
```

### OVMFが見つからない場合

OVMFファームウェアを以下のパスに配置してください：

```text
third_party/ovmf/RELEASEX64_OVMF.fd
```

### ビルドエラーが発生する場合

1. Rustのnightly版を使用しているか確認
2. x86_64-unknown-uefiターゲットがインストールされているか確認
3. rust-srcコンポーネントがインストールされているか確認

```bash
rustup default nightly
rustup target add x86_64-unknown-uefi
rustup component add rust-src
```

## 開発フロー

1. コードを編集
2. `task build` でビルド
3. `task run` でQEMUで実行
4. エラーがある場合は修正して再度ビルド

## 使用可能なTaskコマンド一覧

| コマンド | 説明 |
|---------|------|
| `task` | 利用可能なタスク一覧表示 |
| `task build` | x86_64 UEFIターゲット向けデバッグビルド |
| `task build:release` | x86_64 UEFIターゲット向けリリースビルド |
| `task run` | ビルド＆QEMU実行 |
| `task run:watch` | ファイル変更監視＆自動実行 |
| `task lint` | 全lintチェック実行 |
| `task lint:yaml` | YAMLファイルのlint |
| `task lint:rust` | Rustコードのlint |
| `task format` | 全フォーマット実行 |
| `task format:rust` | Rustコードのフォーマット |
| `task test` | テスト実行（UEFIビルドでは動作しません） |
| `task before-commit` | コミット前チェック |

## Docker環境での開発

Dockerコンテナ内で開発する場合：

```bash
# コンテナ起動
task up

# コンテナ内でビルド＆実行
task run-docker

# コンテナ停止
task down
```

## 注意事項

- このプロジェクトは`no_std`環境のため、標準ライブラリは使用できません
- テストは通常のRust環境では実行できないため、QEMUでの動作確認が必要です
- コミット時は`task before-commit`が自動実行されます
