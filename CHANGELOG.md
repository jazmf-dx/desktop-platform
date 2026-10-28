# Changelog

release は `v<major>.<minor>.<patch>` の tag と、この file の `## <version>` の節で行う。各節に「既存のツールで対応が必要な変更」の有無を必ず書く。

## 0.6.0

パッケージマネージャの既定を npm から Bun にした。Web アプリ向けの Platform と同じにして、コマンドと lockfile の扱いを 1 つにする。npm を選んだ理由だった `tauri-action` は、今は `bun.lock` を見て自動で Bun を使う（[ADR-0005](decisions/adr-0005-bun-as-package-manager.md)）。

### 内容

- [Stack](recommendations/stack.md)「パッケージマネージャ」：既定を Bun に。`bun.lock` を commit し、CI は `oven-sh/setup-bun` と `bun install --frozen-lockfile`。Node.js 22 は残す。`package.json` の `test` は `bun run test` で呼ぶ（`bun test` は Bun のテストランナー）。引数は `--` を挟まずに渡す（`bun run tauri build --target …`）
- [Governance](standards/governance.md) §2・§4、[Distribution](standards/distribution.md) §2・§5（`oven-sh/setup-bun` は `v2` 以上）、雛形の `CLAUDE.md`、Playbook のコマンドを Bun に

### 既存のツールで対応が必要な変更

- **Adopt going forward**：stockroom と desktop-apps の各ツールを Bun に移す。`package-lock.json` を消して `bun.lock` を commit し、CI・`tauri.conf.json` の `beforeDevCommand` / `beforeBuildCommand`・README・`CLAUDE.md` のコマンドを直す

## 0.5.0

アプリとしての基本（アップデートの知らせ方、設定画面、ヘルプメニュー）をどのツールでも同じ形にした。アップデートのダイアログにアプリの名前が無く、急に出てくるとどのアプリからか分からなかった。リリースノートをそのまま載せていたツールもあり、ダイアログが長くなっていた。

### 内容

- [Distribution](standards/distribution.md) §4.1「知らせ方」：タイトル「`<Product>` の新しいバージョンがあります」、本文は新旧のバージョンと再起動すること、ボタンは「更新して再起動」「あとで」「更新内容を見る」を OS の並びで。リリースノートは載せず、「更新内容を見る」で GitHub のリリースのページを開く（URL は Rust の `open_release_page` で組み立てる）。見送った版は設定やトレイのメニューから入れられる。「あとで」のあとにもう一度聞くのは次の自動の確認だけ（ウィンドウのあるツールは起動時、トレイ常駐は 24 時間ごと）。設定画面が開いているあいだは、新しい版が見つかってもダイアログを出さず節の中に出す。更新したあとの最初の起動でトーストを出す
- [UI](standards/ui.md) §9：設定画面の枠（ページの一覧、ページ名、最後に見たページで開く）、行の部品（スイッチ・ラジオの行・チェックボックス）、キーボード（Esc、↑↓ はページの一覧にフォーカスがあるときだけ、Tab をダイアログの中に）、「一般」の最後の「アップデート」の節、ヘルプメニューの項目（「このバージョンの更新内容」、Windows の「`<Product>` について」）
- [トレイ常駐](standards/optional/tray-app.md)：見送った版をトレイのメニューに出す

### 既存のツールで対応が必要な変更

- **Required**：アップデートのダイアログを §4.1 の形にする（アプリ名のタイトル、リリースノートを載せない、「更新内容を見る」）。stockroom と workbench の 7 ツールは対応済み
- **Adopt going forward**：設定画面は stockroom の `SettingsFrame.tsx` と `UpdateSection.tsx` を写して使う。desktop-base ができたらそちらへ移す

## 0.4.0

表示を切り替えても操作の位置を変えない、という決まりを UI Standard に足した。stockroom では表示を切り替えるたびにツールバーの行や項目が増減し、手が覚えた場所を毎回探し直すことになっていた。

### 内容

- [UI](standards/ui.md) §10「レイアウトの安定」：タブ・フィルタ・表示の切り替えなどを繰り返しても、主要な操作の位置やレイアウトを不必要に変化させない。無効表示（存在を知る必要がある操作だけ）、`min-width` や表示領域の確保、表示ごとの操作の置き場所は実現方法の例とし、設定ダイアログのような依存関係に沿った動的 UI は許容する
- [ADR-0004](decisions/adr-0004-layout-stability-ahead-of-source.md)：このルールは出どころの実装より先に載せた（ADR-0003 の例外）。stockroom で直したら出どころを差し替える

### 既存のツールで対応が必要な変更

- **Adopt going forward**：新しく作る画面、手を入れる画面から §10 を守る。stockroom は `docs/UI.md` の一覧に沿って順に直す

## 0.3.0

AI モデルのティアを使い分ける決まりを Governance に足した。Web 側の ai-dev-platform にはあった決まりで、デスクトップ側に無かったため、すべての作業が最上位のモデルで行われていた。

### 内容

- [Governance](standards/governance.md) §3.1：AI モデルを Light / Standard / High Performance のティアで使い分ける。具体的なモデル名は Standard に固定せず、各ツールの `CLAUDE.md` に書く
- `templates/CLAUDE.md` の「開発の基準」にモデルの割り当ての例を足した

### 既存のツールで対応が必要な変更

- **Adopt going forward**：ツールの `CLAUDE.md` の「開発の基準」に、ティアごとに使うモデルを 1 行書く（雛形は [templates/CLAUDE.md](templates/CLAUDE.md)）

## 0.2.0

CI の action を Node.js 24 で動く版にそろえる決まりを足した。古い版のままだと、ビルドのたびに「Node.js 20 is deprecated」の警告が出る。

### 内容

- [Distribution](standards/distribution.md) §6：JavaScript の action は Node.js 24 で動く版を使う。`checkout`、`setup-node`、`upload-artifact`、`download-artifact`、`cache`、`rust-cache`、`tauri-action` の版の表と、`setup-node` の `node-version` とは別の話であることを書いた
- この Platform の `check-docs` workflow の `actions/checkout` を `v5` に上げた

### 既存のツールで対応が必要な変更

- **Required**：CI の `actions/checkout@v4`、`actions/setup-node@v4`、`actions/upload-artifact@v4` などを [Distribution](standards/distribution.md) §6 の表の版以上に上げる。Actions の annotation に「Node.js 20 is deprecated」が出ているツールが対象

## 0.1.1

image-library が stockroom に改名したのに合わせて、名前と参照を直した。

### 内容

- すべての Standard、Recommendations、Playbook、ADR、templates の image-library を stockroom に置き換えた。README に旧名とリポジトリの場所を書いた
- `playbook/release.md` の `cargo update -p` を stockroom の crate 名に合わせた
- `playbook/updater-key.md`：鍵のファイル名は改名前のままであることと、owner が違うツールはその owner で鍵を作ることを明記した

### 既存のツールで対応が必要な変更

- なし

## 0.1.0

最初の版。image-library（v0.17.0）と Shelf（0.1.3）で有効だったものを Core Standard にした。

### 内容

- `ai/ONBOARDING.md`：AI agent の唯一の入口
- Core Standard：Governance、Architecture、Security、Distribution、UI
- Optional Standard：ローカル受付、複数ツール同居リポジトリ、トレイ常駐
- Recommendations：Stack、Rust
- Playbook：新しいツールを作る、リリース、アップデートの署名鍵、ブラウザで UI を作る、macOS と Windows の差
- ADR-0001〜0003、`docs/adoption.md`、`templates/`

### 既存のツールで対応が必要な変更

- **Required**：Shelf の `tauri.conf.json` の `csp: null`（[Security](standards/security.md) §1）
- **Adopt going forward**：image-library と Shelf に `CLAUDE.md` と `decisions/project-context.md`、CI に Linter / fmt / clippy（[Adoption Guide](docs/adoption.md) §6）
