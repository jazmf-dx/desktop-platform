# Adoption Guide

`desktop-platform` を新しいツールへ導入するとき、または既存のツール（stockroom、Shelf）へ段階的に適用するときの手順です。

## 1. Platform を workspace へ配置する

ツールごとに submodule にせず、複数のツールから参照できる位置へ Platform を 1 つ置きます。

```text
workspace/
├── desktop-platform/     ルール
├── desktop-base/         共通コード（crate、npm package、template、workflow）
├── stockroom/
└── desktop-apps/
    └── shelf/
```

```bash
git clone https://github.com/<owner>/desktop-platform.git
git clone https://github.com/<owner>/desktop-base.git
```

再現性が必要なときは release tag を checkout し、ツールの `CLAUDE.md` にその tag を書きます。

## 2. ツールから唯一の AI 入口を参照する

各ツールの `CLAUDE.md` から Platform の `ai/ONBOARDING.md` を参照させます。雛形は [templates/CLAUDE.md](../templates/CLAUDE.md)。

```markdown
開発の Standard・Recommendations・Playbook は
../desktop-platform/ai/ONBOARDING.md
を最初に参照すること。Web アプリ向けの ai-dev-platform は参照しない。
```

relative path は workspace の構成に合わせます。同居リポジトリでは `../../desktop-platform/...` になります。

`standards/` や `recommendations/` の個別 file を直接 AI の入口にしません。

## 3. Project Context を用意する

ツール直下に `decisions/project-context.md` を置き、AI が実装前に確認すべき **現在の前提** を記録します。雛形は [templates/project-context.md](../templates/project-context.md)。

| 項目 | 確認すること |
|---|---|
| ツールの種類 | ウィンドウアプリか、トレイ常駐か |
| UI の実装 | React + zustand + Tailwind 4 か、素の TypeScript か |
| データの置き場 | `app_data_dir()` の JSON か、ユーザーが選ぶフォルダ（SQLite）か。構造は README のどこか |
| 外との接続 | なし / ブラウザ拡張 / MCP / ほか。ポートと token の扱い |
| 配布の形 | 単独リポジトリか desktop-apps 同居か。tag と feed の形 |
| 識別子 | `identifier`。一度配布したら変えない |
| desktop-base | 使っている tag。未採用なら stockroom のどのコミットから写したか |

該当しない項目は「該当なし」と書きます。機能要件、ロードマップ、画面仕様は置きません（それらは `docs/`）。

## 4. Core Standard を確認する

[ONBOARDING](../ai/ONBOARDING.md) を入口に、Task に必要な Core Standard を読みます。Optional Standard は該当する機能を扱うときだけ読みます。

## 5. 新しいツールで用意するもの

- `CLAUDE.md` から Platform ONBOARDING を参照する
- `decisions/project-context.md` を用意する
- `package.json` に `test` と `lint` の script を置く
- `CHANGELOG.md` を置く
- CI に [Governance](../standards/governance.md) §4 の gate を入れる
- [新しいツールを作る](../playbook/new-tool.md) の checklist を通す

## 6. 既存のツールへ適用する

Standard との違いだけを理由に全面 migration しません。最初に `CLAUDE.md` と `project-context.md` を足し、差分を 3 つに分けます。

- **Required**：今回の変更箇所で Standard に反するもの、またはセキュリティ・データ・必須 gate の現在の問題
- **Adopt going forward**：既存方式は維持し、新規・変更箇所から Standard へ寄せるもの
- **Optional improvement**：品質向上、近代化。Standard 適合とは分離するもの

2026-10-11 時点の差分：

| ツール | 区分 | 差分 |
|---|---|---|
| stockroom | Required | なし |
| stockroom | Adopt going forward | `CLAUDE.md` と `project-context.md` が無い。Linter / fmt / clippy が CI に無い。リリースの本文が CHANGELOG でなく固定文。`proxy.rs` / `update.ts` / `log.ts` を desktop-base に差し替える |
| stockroom | Optional improvement | 画像だけの機能（重複、回転）を `features/images/` に寄せる（`docs/ARCHITECTURE.md`「今後」） |
| Shelf | Required | `tauri.conf.json` の `csp` が `null` |
| Shelf | Adopt going forward | `CLAUDE.md` と `project-context.md` が無い。Linter / fmt / clippy が CI に無い。`proxy.rs` / `update.rs` を desktop-base に差し替える |
| Shelf | Optional improvement | なし |

## 7. Platform を更新する

```bash
cd desktop-platform
git switch main
git pull --ff-only
```

tag で固定している場合は `main` に切り替えず、採用する tag を checkout して `CLAUDE.md` の記載を更新します。`CHANGELOG.md` の「既存のツールで対応が必要な変更」を見て、Required があれば先に対応します。

## 完了条件

- Platform が workspace にある
- ツールの `CLAUDE.md` が Platform ONBOARDING だけを入口にしている
- `decisions/project-context.md` に 7 項目が記録されている
- Core Standard との差分が Required / Adopt going forward / Optional improvement に分かれている
