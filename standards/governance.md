# Governance Standard

すべての Task に適用する行動規範です。Git の扱い、AI 利用、必須の機械的検証、Standard からの逸脱の記録、秘密情報を決めます。

## 1. 適用範囲

- この Platform を `CLAUDE.md` から参照するすべてのデスクトップツール。
- 人も AI agent も同じルールに従う。AI にだけ緩い、または厳しいルールを作らない。

## 2. Git

- `main` は常にリリースできる状態に保つ。直接 push せず、branch と Pull Request で入れる。
- commit message は 1 行目に種別の接頭辞を付け、本文は日本語でよい。種別は `feat` / `fix` / `docs` / `chore` / `refactor` / `test` / `ci`。
  - 例：`fix: 重い処理でウィンドウが固まらないように（一覧の問い合わせ・削除・書き出し・回転）`
  - バージョンを上げる commit は `chore: vX.Y.Z`（単独リポジトリ）または `chore: <name> vX.Y.Z`（同居リポジトリ）
- lockfile（`bun.lock`、`Cargo.lock`）は commit する。1 リポジトリに複数のパッケージマネージャの lockfile を置かない。
- 生成物（`dist/`、`src-tauri/target/`、`src-tauri/gen/schemas/`）は commit しない。
- tag は配布の単位にだけ打つ（[Distribution](distribution.md) §5）。

出どころ：stockroom の commit 履歴、desktop-apps（旧 workbench）の README「リポジトリの決まりごと」。

## 3. AI 利用

- Task の最初に [ONBOARDING](../ai/ONBOARDING.md) を読み、必要な Standard だけへ進む。
- 実装へ大きく影響する場合は、対象ツールの `decisions/project-context.md` を先に読む。無ければ作ることを提案し、仮定を置いて進めない。
- 依存を増やすときは [Recommendations](../recommendations/README.md) の健全性チェックを行い、結果を短く提示してから追加する。黙って追加しない。
- 次は **人の判断なしに変えない**：`identifier`、`productName`、updater の `pubkey` と `endpoints`、署名まわり、CSP の緩和、capabilities の追加、待ち受けポート。
- 秘密情報（署名鍵、token）をファイルや commit、ログ、PR の説明に書かない。
- PR を出す前に §4 の gate をローカルで通す。通らないものを「CI で確認する」と言って出さない。
- 変更の説明は、何を変えたかだけでなく **なぜそうしたか** を README や docs のコメントに残す。stockroom のコードコメントと同じ密度を保つ。

### 3.1. モデルの選び方

- タスクの難しさに応じて AI モデルのティアを使い分ける。具体的なモデル名はこの Standard に固定せず、ツールの `CLAUDE.md` に書く。
- **Light**：テスト実行、ログ収集、スクリーンショットなどの単純な確認、軽微な修正
- **Standard**：通常の実装、ドキュメント、一般的なレビュー（既定）
- **High Performance**：複雑な設計、難しい障害調査、重要なレビュー。必要なときだけ使い、通常のタスクで常用しない
- 1 つのセッションの中で、設計とレビューは上のティア、実装と確認は下のティアへ委譲する形でよい。委譲した側は結果を確認してから次へ進む。

出どころ：ai-dev-platform の Governance Standard §2.1「モデル選定」。Web 側と同じティア分けを、名前を固定しない形で持ち込んだ。

## 4. 必須の機械的検証

次を **PR と `main` への push** で CI が実行し、すべて通ることを merge の条件にする。

| gate | command | 対象 |
|---|---|---|
| 型チェック | `bunx tsc --noEmit` | `src/` |
| Linter | ツールの `package.json` に定義した lint script | `src/` |
| フォーマット | `cargo fmt --check --manifest-path src-tauri/Cargo.toml` | `src-tauri/` |
| 静的検査 | `cargo clippy --manifest-path src-tauri/Cargo.toml -- -D warnings` | `src-tauri/` |
| Rust テスト | `cargo test --manifest-path src-tauri/Cargo.toml` | `src-tauri/` |
| ビルド | `bun run tauri build`（macOS Apple Silicon と Windows の両方） | 配布物 |

- Rust テストと型チェックは Linux の runner で走らせてよい（安い）。ビルドは配布対象の OS で走らせる。
- 既存ツール（stockroom、Shelf）は Linter、fmt、clippy が CI に無い。desktop-base の再利用 workflow に入れる形で埋め、それまでは **Adopt going forward** として扱う（[Adoption Guide](../docs/adoption.md) §6）。
- gate を通すためにテストを skip、`#[allow]`、`// @ts-ignore` で黙らせない。必要なら理由を書いて PR で説明する。

出どころ：stockroom の `.github/workflows/build.yml`（`cargo test` と `tsc --noEmit` の test job、macOS / Windows のビルド matrix）、`package.json` の `test` script（`bun run test`）。

## 5. Standard からの逸脱

- Core / Optional Standard から外れるときは、そのツールの `decisions/adr-NNNN-<slug>.md` に理由と影響を残す（雛形は [decisions/adr-template.md](../decisions/adr-template.md)）。
- Recommendation から外れるときは PR の説明に理由を書けば足りる。ADR は不要。
- 逸脱が 2 つ目のツールでも必要になったら、Standard 側を直す PR を出す。ツール側の ADR を増やし続けない。

## 6. 秘密情報

- updater の署名鍵は 1 つの owner で 1 本。GitHub Actions の Secrets（`TAURI_SIGNING_PRIVATE_KEY`、`TAURI_SIGNING_PRIVATE_KEY_PASSWORD`）に置き、秘密鍵のファイル自体はパスワードマネージャーなどに **必ずバックアップ** する。失うと配布済みのアプリに更新を届けられない。
- ローカル受付の token、ペアリングの接続キーはアプリが生成し、ユーザーのアプリ設定だけに置く。リポジトリや拡張機能のソースに書かない。
- 鍵の生成、保管、作り直しは [アップデートの署名鍵](../playbook/updater-key.md)。

出どころ：stockroom README「リリース」、Shelf README「アプリ内アップデートの仕組み」。
