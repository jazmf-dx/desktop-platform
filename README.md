# Desktop Platform

Tauri で作るデスクトップツールの **開発基準** です。何を守るか（Standards）、普段は何を選ぶか（Recommendations）、どう実施するか（Playbook）を 1 か所にまとめ、人と AI agent が同じ入口から必要な資産だけへ進めるようにします。

Web アプリ向けの `ai-dev-platform`（Django / PostgreSQL / React Islands）とは **別軸** です。参照も共有もせず、語彙と構造だけを揃えています（[ADR-0001](decisions/adr-0001-separate-from-web-platform.md)）。

## 構成

```text
desktop-platform/
├── ai/
│   └── ONBOARDING.md       AI agent が最初に読む唯一の入口
├── standards/              何を守るか。Core と Optional
│   ├── governance.md       Git、AI 利用、機械的 gate、逸脱の記録
│   ├── architecture.md     層、ディレクトリ、コマンド表、IPC、データの置き場、ログ
│   ├── security.md         CSP、capabilities、外から来るファイル、ネットワーク
│   ├── distribution.md     対象 OS、署名なし配布、アプリ内アップデート、リリース、CI
│   ├── ui.md               テーマ、キーボード、ウィンドウ、フィードバック
│   └── optional/           ローカル受付、複数ツール同居、トレイ常駐
├── recommendations/        現時点の選択の正本（Tauri plugin、React、Rust crate）
├── playbook/               新しいツールを作る、リリース、鍵、ブラウザで UI、OS の差
├── decisions/              この Platform 自体の ADR
├── docs/adoption.md        導入と既存アプリへの適用
└── templates/              アプリ側に置く CLAUDE.md と project-context.md の雛形
```

コード（Rust crate、npm package、template、再利用 workflow）はここに置きません。`desktop-base` が所有します（[ADR-0002](decisions/adr-0002-rules-and-base-repositories.md)）。

## 各領域の役割

| 領域 | 問い | 正本 |
|---|---|---|
| Standards | 何を守る？ | `standards/` |
| Recommendations | 普段は何を選ぶ？ | `recommendations/` |
| Playbook | どう作る・出す・直す？ | `playbook/` |
| desktop-base | 共通コードは何を使う？ | `desktop-base` リポジトリ |
| Project Context / ADR | このツール固有ではどうする？ | 各ツールの `decisions/` |

同じ内容を複数箇所へ copy しません。

## 導入

Platform は複数のツールから参照できる workspace 上の位置へ 1 つ配置し、各ツールの `CLAUDE.md` から `ai/ONBOARDING.md` を最初に読むよう参照させます。ツール側には `decisions/project-context.md` を用意します。

```text
workspace/
├── desktop-platform/
├── desktop-base/
├── stockroom/
└── desktop-apps/
```

stockroom は旧名 image-library（[2204601/stockroom](https://github.com/2204601/stockroom)）。Standard の「出どころ」はこのリポジトリのファイルを指す。

手順、Project Context の項目、既存ツールへの段階適用は [Adoption Guide](docs/adoption.md) を参照してください。

## 更新方法

- release は SemVer の tag `v<major>.<minor>.<patch>` と `CHANGELOG.md` の節で行います。Release Please などの自動化は入れません
- 再現性が必要なときは、ツールの `CLAUDE.md` に参照した tag を書きます
- 既存利用者が採用方法を変える必要があるか、で MAJOR を判断します

## 知識・資産の成熟

```text
ツール固有の判断・実装（stockroom、Shelf …）
        ↓
2 つ目のツールでも同じ形が有効だった
        ↓
Recommendation / Playbook
        ↓
コードとして共有するなら desktop-base
        ↓
不一致そのものが実害になる（配布、セキュリティ、ログ、運用）
        ↓
必要な場合のみ Standard
```

最初からすべてを Standard や共有コードへ昇格させません。今の Core Standard は、stockroom と Shelf の 2 つで有効だったものだけを載せています。

## このリポジトリに置かないもの

- Rust / TypeScript のコード、template、CI workflow の本体（desktop-base）
- ツール固有の機能要件、ロードマップ、画面仕様（各ツールの `docs/`）
- Web アプリの基準（ai-dev-platform）
- 採用実績のない候補、比較表、調査メモ

## メンテナンス

```bash
just check-docs   # Markdown のリンク切れを lychee で検査（offline）
```
