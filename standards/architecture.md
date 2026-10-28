# Architecture Standard

ツールの中の **層、ディレクトリ、コマンド表、IPC、設定とデータの置き場、ログ、テスト、ドキュメント** を決めます。Tauri 2 と React を前提にします（[Stack](../recommendations/stack.md)）。トレイ常駐の小物で素の TypeScript を使う場合も、§3 以外は同じです。

## 1. 層と依存の向き

| 層 | 場所 | 中身 |
|---|---|---|
| shell | desktop-base（それまではツール内の同名ファイル） | OS とアプリ運用：proxy、updater、log の配線、single-instance、window-state、settings.json、ウィンドウ生成 |
| core | `src/lib/*`、`src/store.ts`、`src/components/*`、`src-tauri/src/*.rs` | そのツールのドメイン：データ、取り込み、一覧の骨組み、共通コマンド |
| features | `src/features/<kind>/`、`src-tauri/src/<kind>/` | 種類ごとの振る舞い（stockroom のフォント、ファイル） |

- 依存の向きは **features → core → shell** のみ。core が features の知識を持つのは、データに入っている語彙（種類の列挙、ラベル）までにとどめ、振る舞いは features に置く。
- shell にはツールのドメインを入れない。desktop-base に出せるかどうかが判定になる。
- 共通 UI（一覧、サイドバー）が features の部品を描く箇所で import が循環する場合、描画時にしか参照しないなら許容する。解きたければ core の部品を独立したファイルに出す。

出どころ：stockroom `docs/ARCHITECTURE.md`「境界」。

## 2. ディレクトリ構成

```text
<tool>/
  src/                    React UI
    App.tsx               レイアウト、ファイルのドロップ・貼り付け
    store.ts              zustand ストア（表示条件・選択・再読込）
    lib/api.ts            Rust コマンドの型付きラッパ（共通）
    lib/commands.ts       コマンド表（§3）
    lib/actions.ts        ダイアログを伴う操作
    components/           共通の UI
    features/<kind>/      種類ごとの UI と API
    dev/mockBackend.ts    ブラウザ開発用モック
  src-tauri/
    src/lib.rs            plugin の配線、setup、invoke_handler
    src/main.rs           エントリ（`--mcp` など別モードの分岐だけ）
    src/commands.rs       フロントから呼ぶコマンド（共通）
    src/<area>.rs         ドメインごとのモジュール（db、import、search …）
    examples/             アプリなしで動かす補助（seed、受付サーバー）
    capabilities/         ウィンドウごとの権限
    tauri.conf.json
  docs/                   ARCHITECTURE.md、ROADMAP.md、機能ごとの決定
  decisions/              project-context.md、ADR
  CLAUDE.md
  README.md
  CHANGELOG.md
```

- 機能を足すときは `features/<kind>/` と `src-tauri/src/<kind>/` を対にして足す。`components/` に種類固有の UI を混ぜない。
- 複数ツール同居リポジトリでは `<tool>/` がこの単位になる（[Optional](optional/multi-tool-repo.md)）。

出どころ：stockroom README「構成」。

## 3. コマンド表

ユーザーの操作は `src/lib/commands.ts` の表に **1 度だけ** 定義し、メニューバー、右クリックメニュー、一覧のキー処理、ショートカット一覧をすべてこの表から作る。名前とキーがどこでも同じに読める。

```ts
interface Command {
  id: string;                    // "item.favorite"
  title: string;                 // ショートカット一覧の名前と既定のラベル
  label?: (s: State) => string;  // 状態に応じたラベル（「お気に入りに追加」/「から外す」）
  icon?: LucideIcon;
  keys?: string[];               // "Mod+Shift+R"。Mod は macOS で ⌘、Windows で Ctrl。先頭がメニューに出る
  kinds?: ItemKind[];            // 使える種類（無ければ全部）
  enabled?: (s: State) => boolean;
  checked?: (s: State) => boolean;
  run: (s: State) => void;
  keyBy?: "grid" | "global" | "menu"; // 誰がキーを受けるか
}
```

- 操作を足すときは **まずこの表に足す**。メニューやキー処理に直接書かない。
- ⌘ / Ctrl 付きのキーだけをメニューのアクセラレータにする。1 文字のキー（F、P、B、数字）は入力欄に文字を打てなくなるので表示だけにする。
- ダイアログやメニューなど一覧の上に重なるものは外枠に `data-modal` を付け、その間は一覧のキー処理を止める。
- 入力欄のキー処理では日本語入力の変換中（`isComposing` または `keyCode === 229`）を除く。macOS の WKWebView は確定の Enter の時点で `isComposing` が false になっている。

出どころ：stockroom `src/lib/commands.ts`、`lib/menuBar.ts`、`lib/menuItems.ts`、`lib/shortcuts.ts`、`lib/ime.ts`、`docs/MENUS.md`。

## 4. IPC（フロントと Rust）

- Rust コマンドは `#[tauri::command]` で定義し、引数と戻り値は `serde` の型（`rename_all = "camelCase"`）。戻り値は `Result<T, String>` にし、エラー文はそのままユーザーに見せられる日本語にする。
- フロントは `src/lib/api.ts` の型付きラッパ経由でだけ `invoke` する。コンポーネントから直接 `invoke` しない。
- 重い処理（一覧の問い合わせ、削除、書き出し、回転、ハッシュ）はメインスレッドで実行しない。Rust 側で `async` にするか別スレッドへ逃がし、ウィンドウを固めない。並列化は rayon。
- 数秒かかる処理は進捗を出し、取り消せるようにする。10 秒以上かかってほかのアプリを使っている間に終わったものは OS の通知で知らせる（[UI](ui.md) §5）。
- 外から来る引数（ファイルパス、URL、拡張機能からの body）は Rust 側で検証する（[Security](security.md) §4）。

出どころ：stockroom `src/lib/api.ts`、`src-tauri/src/commands.rs`、commit「重い処理でウィンドウが固まらないように」。

## 5. 状態と設定の 3 層

| 層 | 保存先 | 中身 |
|---|---|---|
| アプリ全体 | `app_config_dir()/settings.json` | 起動時の挙動、アップデートの自動確認、連携の on/off、最近使ったデータの一覧 |
| データごと | データの中（stockroom は `library.db` の `settings` テーブル） | そのデータの性質に関わる設定。データを持ち運べば設定も付いてくる |
| 表示の好み | `localStorage` | レイアウト、サムネイルの大きさ、パネルの開閉。どのデータでも同じ見た目のほうが迷わない |

- フロントの状態は zustand の 1 つのストアに置く。コンポーネントの `useState` はそのコンポーネントで閉じるものだけ。
- 表示の好みをデータに付けたくなったものだけ、あとからデータごとの層へ移す。

出どころ：stockroom `docs/SETTINGS.md`「設定の置き場所（3 層）」。

## 6. データの置き場

- アプリの内部ファイルは Tauri の path API で得る場所に置く：設定は `app_config_dir()`、データは `app_data_dir()`、ログは `app_log_dir()`。場所を自分で組み立てない。識別子を変えるとこれらが変わるので、識別子は変えない（[Distribution](distribution.md) §2）。
- ユーザーのデータを大量に持つツールは、**フォルダ 1 つで完結** させる。フォルダごとコピーすればバックアップと別 PC への移動ができる形にし、構造を README に書く。
- SQLite は WAL で使い、終了時とデータの切り替え時に **すべて書き戻してから閉じる**。同期フォルダ（iCloud Drive、Dropbox、OneDrive）が途中の状態を写さないようにする。
- 同期フォルダや NAS で複数の PC から使われうるデータは、開いている間だけロックファイルを置いて定期的に更新し、別の PC で開こうとしたら確認を出す。頼り切らず、README に「開くのは 1 台ずつ」と書く。
- ユーザーの元ファイルを **黙って書き換えない**。回転などは向きの情報だけ記録する。書き換えが目的のツール（リネームなど）は、プレビューと取り消しを持つ（§7）。
- データの形式を変えるときは `CREATE TABLE IF NOT EXISTS` や `user_version` で前方互換に足す。古い版で開いても壊れない範囲で変える。

出どころ：stockroom `src-tauri/src/library.rs`、README「ライブラリの構造」「バックアップ」。Shelf `src-tauri/src/shelf.rs`（JSON 1 ファイル）。

## 7. 取り消し

- ゴミ箱へ移動、フォルダ移動、タグの貼り付けなど、結果を見てから戻したくなる操作は、トーストに「元に戻す」を付ける。
- UI の外から来た変更（MCP 経由の AI、ほかのプロセス）は、何を変えたかを記録し、あとからまとめて戻せるようにする。戻すときはその後に手で変えたものを上書きしない。
- 完全削除は確認ダイアログを出す。ゴミ箱の外の完全削除を AI や外部連携に公開しない。

出どころ：stockroom `src/lib/actions.ts`、`src-tauri/src/changes.rs`。

## 8. ログ

- `tauri-plugin-log` を Info で使い、stdout と `app_log_dir()` のファイルへ出す。1 MB で切り替え、古いものは 2 つまで残す。時刻はローカル。
- 起動時にアプリ名、バージョン、OS、アーキテクチャを 1 行出す。
- `panic` もログに残す。release ビルドは `panic = "abort"` なので、hook を入れないと何も残らない。
- フロントの警告・エラーは `warn()` / `error()` のラッパ経由で plugin-log にも送る。`window.onerror` と `unhandledrejection` も拾う。
- ヘルプメニューに「ログを表示」を置き、ログフォルダを開く。README にログの場所を書く。
- ログに秘密情報（token、鍵、ユーザーのファイルの中身）を出さない。パスは出してよい。

出どころ：stockroom `src-tauri/src/lib.rs`（`log_panics`、plugin-log の Builder）、`src/lib/log.ts`、README「ログ」。

## 9. テスト

- Rust のユニットテストは対象のモジュールの中に置く（`db.rs`、`search.rs`、`shelf.rs`）。Tauri の `AppHandle` に依存しない形で書けるよう、OS や UI への呼び出しは trait で挟む（`webimport.rs` の `Host`）。
- フロントは型チェックを最低限とし、ロジックは `src/lib/` の純粋関数に寄せる。
- UI はブラウザのモックで手で確認する（[ブラウザで UI を作る](../playbook/ui-in-browser.md)）。モックは `api.ts` と同じ型で書き、`api.ts` を変えたらモックも変える。
- 性能はテストデータの生成 example（`examples/seed.rs`）で測る。1 万件で一覧が固まらないことを確かめてから出す。

出どころ：stockroom `package.json` の `test` script（`bun run test`）、`src-tauri/examples/seed.rs`、`src/dev/mockBackend.ts`。

## 10. ドキュメント

- README はこの順で書く：**機能 / キーボード / データの構造（あれば）/ インストール / アップデート / 開発 / 構成 / リリース**。ユーザー向けの節を先に、開発者向けを後に。
- 設計の境界は `docs/ARCHITECTURE.md`、機能ごとの決定は `docs/<topic>.md` に「困っていること → 決定 → 実装」の順で残す。
- `CHANGELOG.md` に版ごとの節を書き、それをそのままリリースの本文にする（[Distribution](distribution.md) §5）。
- コードのコメントは「なぜ」を書く。何をしているかは名前で分かるようにする。

出どころ：stockroom README、`docs/MENUS.md`、`docs/SETTINGS.md`。Shelf README、`CHANGELOG.md`。
