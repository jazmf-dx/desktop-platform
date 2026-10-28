# 新しいツールを作る

stockroom を土台に、新しいデスクトップツールを起こして最初のリリースまで持っていく手順。

## 0. 決めること

`decisions/project-context.md`（雛形：[templates/project-context.md](../templates/project-context.md)）に先に書く。

| 項目 | 選択肢 |
|---|---|
| ツールの種類 | ウィンドウアプリ / トレイ常駐（[Optional](../standards/optional/tray-app.md)） |
| UI の実装 | React + zustand + Tailwind 4 / 素の TypeScript（トレイ常駐の小物だけ） |
| データの置き場 | `app_data_dir()` の JSON / ユーザーが選ぶフォルダ 1 つ（SQLite） |
| 外との接続 | なし / ブラウザ拡張 / MCP / ほか（[Optional](../standards/optional/local-server.md)） |
| 配布の形 | 単独リポジトリ / desktop-apps に同居（[Optional](../standards/optional/multi-tool-repo.md)） |
| `identifier` | 逆 DNS。一度決めたら変えない |

## 1. 雛形を置く

### desktop-base が v0.1.0 以降のとき

desktop-base の `template/` をツールのディレクトリへコピーし、`Cargo.toml` と `package.json` の desktop-base の依存を **tag で** 固定する。

### desktop-base がまだ無いとき

stockroom から次を写す。写した時点のコミットを `decisions/project-context.md` に書いておき、desktop-base ができたら差し替える。

| 写すもの | 変えるところ |
|---|---|
| `vite.config.ts`、`tsconfig.json`、`tsconfig.node.json`、`index.html` | そのまま |
| `src-tauri/Cargo.toml` | `name`、`version`、`description`、`[lib] name`。使わない依存を消す。`[profile.release]` はそのまま |
| `src-tauri/tauri.conf.json` | `productName`、`identifier`、`version`、window、`plugins.updater.endpoints`。`csp` と `bundle` はそのまま。`pubkey` は owner 共通 |
| `src-tauri/capabilities/default.json` | 使う plugin の権限だけ残す |
| `src-tauri/src/main.rs`、`lib.rs` の plugin 配線と `log_panics`、`main_window` | アプリ名、コマンド一覧 |
| `src-tauri/src/proxy.rs` | PAC のモード名と User-Agent の文字列 |
| `src/main.tsx`、`src/lib/log.ts`、`titleBar.ts`、`osNotify.ts`、`ime.ts` | そのまま |
| `src/lib/update.ts` | `FEED`、ストアへの依存をトーストのコールバックに |
| `src/index.css` の `@theme` | そのまま |
| `src/dev/mockBackend.ts` の骨組み | ツールの `api.ts` に合わせる |
| `.vscode/extensions.json`、`.claude/launch.json` | そのまま |
| `.github/workflows/build.yml`（単独）または desktop-apps の `shelf.yml`（同居） | ツール名、ディレクトリ、feed |

トレイ常駐なら Shelf の `lib.rs`（`setup_tray`）、`update.rs`、`proxy.rs` を土台にする。

## 2. 名前とアイコン

```bash
bun install
bunx tauri icon app-icon.svg      # src-tauri/icons/ を生成。svg と生成物を commit
```

## 3. AI の入口と README

- `CLAUDE.md`（雛形：[templates/CLAUDE.md](../templates/CLAUDE.md)）を置き、Platform の `ai/ONBOARDING.md` を最初に読ませる。
- README を [Architecture](../standards/architecture.md) §10 の順で書き始める。最初は「機能」「開発」「構成」だけでよい。
- `CHANGELOG.md` に `## 0.1.0` の節を空で作っておく。

## 4. 動かす

```bash
bun run tauri dev        # アプリを起動
bun run dev              # ブラウザで UI だけ（モック。playbook/ui-in-browser.md）
bun run test             # cargo test + tsc --noEmit
```

`package.json` の `scripts` に `test` と `lint` を置く。CI は [Governance](../standards/governance.md) §4 の gate を全部回す。

## 5. 最初のリリースまでの checklist

- [ ] `identifier` と `productName` が project-context と一致し、以後変えない前提で決まっている
- [ ] `tauri.conf.json` の `csp` が `null` でない
- [ ] `capabilities/default.json` が使う権限だけ
- [ ] updater の `pubkey` が owner 共通の公開鍵、`endpoints` がこのリポジトリの feed
- [ ] GitHub Secrets に `TAURI_SIGNING_PRIVATE_KEY`（と `_PASSWORD`）がある
- [ ] ログがファイルに出て、ヘルプ → ログを表示 で開ける
- [ ] ウィンドウの大きさと位置が次回に戻る、二重起動しない
- [ ] README に初回インストールの警告の手順がある（[Distribution](../standards/distribution.md) §3 の文面）
- [ ] CHANGELOG に `## <version>` の節がある
- [ ] `bun run test` と CI の gate が通る
- [ ] macOS と Windows の両方で一度インストールして起動した

リリースの操作は [リリース Playbook](release.md)。
