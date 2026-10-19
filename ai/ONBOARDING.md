# AI エージェント オンボーディング

> AI agent がデスクトップツールの開発に参加するときの **唯一の入口** です。
> 最初にこの file を読み、Task に必要な資産だけへ進みます。

## 必守事項

1. **共有 document を最初から全量読まない。** Task に必要な Core Standard だけを読み、Optional Standard / Recommendation / Playbook は該当する場合だけ読む。
2. **行動規範は [Governance Standard](../standards/governance.md) に従う。** Git、AI 利用、逸脱の記録、必須の機械的 gate はすべての Task に適用する Core Standard であり、この file へ再掲しない。
3. **新しい crate / package を追加する前に [Recommendations](../recommendations/) を確認する。** 未収録のものは [Recommendations README](../recommendations/README.md) の健全性チェックを行い、結果を短く提示してから採用する。
4. **実装へ大きく影響する場合、対象ツールの `decisions/project-context.md` を先に読む。** ツールの種類（ウィンドウ / トレイ常駐）、UI の実装、データの置き場、外との接続、配布の形を勝手に仮定しない。
5. **Web アプリの基準（ai-dev-platform）を持ち込まない。** Django、PostgreSQL、認証・認可、Layout Profile、Lighthouse はデスクトップツールに当てはまらない。

---

## Router

### 1. 判断・制約 → Standards

👉 [Standards](../standards/)

Core Standard はすべてのツールに適用する。

- [Governance](../standards/governance.md) — Git、AI 利用、機械的 gate、逸脱の記録。すべての Task で読む
- [Architecture](../standards/architecture.md) — 層と依存の向き、ディレクトリ、コマンド表、IPC、設定とデータの置き場、ログ、テスト、ドキュメント
- [Security](../standards/security.md) — CSP、capabilities、asset protocol、外から来るファイル、ネットワーク、updater の署名
- [Distribution](../standards/distribution.md) — 対象 OS、識別子、署名なし配布、アプリ内アップデート（ダイアログの形と文言）、リリース、CI
- [UI](../standards/ui.md) — テーマ、言語、キーボード、ウィンドウ、フィードバック、ドラッグ&ドロップ、一覧、設定画面とヘルプメニュー、レイアウトの安定（表示を切り替えても主要な操作の位置を不必要に変えない）

Optional Standard は該当機能を扱う場合だけ参照する。

- [ローカル受付](../standards/optional/local-server.md) — 127.0.0.1 で待ち受ける HTTP（ブラウザ拡張、MCP）
- [複数ツール同居リポジトリ](../standards/optional/multi-tool-repo.md) — desktop-apps のように 1 リポジトリに複数ツールを置く
- [トレイ常駐](../standards/optional/tray-app.md) — Shelf のようにメニューバー / タスクトレイから使う

### 2. Current library / tool choice → Recommendations

👉 [Recommendations](../recommendations/)

新規採用・依存変更時に参照する。Standard より拘束力は弱く、逸脱に ADR は要求しない。

- [Stack](../recommendations/stack.md) — Tauri、React、状態管理、CSS、ビルド、パッケージマネージャ
- [Rust](../recommendations/rust.md) — Tauri plugin、SQLite、画像、並列、ローカル HTTP、OS API

### 3. 共通コード → desktop-base

👉 `desktop-base` リポジトリ（proxy、updater、log の配線、settings、template、再利用 workflow）

- ツールの `Cargo.toml` / `package.json` が pin している tag の desktop-base を使う。main を見ない
- desktop-base に無いものを 2 つ目のツールが必要としたら、ツール側で作ってから昇格を検討する（README「知識・資産の成熟」）
- desktop-base がまだ v0.1.0 に達していない間の代替は [新しいツールを作る](../playbook/new-tool.md) に書いてある

### 4. 作る・出す・直す → Playbook

👉 [Playbook](../playbook/)

具体例、command、checklist、troubleshooting は Task に必要な Playbook だけを参照する。

- [新しいツールを作る](../playbook/new-tool.md)
- [リリース](../playbook/release.md)
- [アップデートの署名鍵](../playbook/updater-key.md)
- [ブラウザで UI を作る](../playbook/ui-in-browser.md)
- [macOS と Windows の差](../playbook/platform-differences.md)

---

## 品質

- 必須 gate（型チェック、Linter、fmt、clippy、Rust テスト、macOS と Windows のビルド）の扱いは [Governance](../standards/governance.md) の「必須の機械的検証」に従う。
- Web の Lighthouse や Core Web Vitals は対象外。代わりに、1 万件規模の一覧が固まらないこと、重い処理でウィンドウが固まらないことを [Architecture](../standards/architecture.md) と [UI](../standards/ui.md) が求める。

---

## 新しい共有資産を追加するとき

配置先の判断は [ADR-0003](../decisions/adr-0003-core-and-optional-standards.md) を正とする。

```text
守る判断・制約                    -> Standards（Core / Optional）
current library / tool choice     -> Recommendations
作る・出す・直す                  -> Playbook
共有するコード                    -> desktop-base
ツール固有                        -> Project Context / ADR / code
```

同じ内容を複数 layer へ copy しない。
