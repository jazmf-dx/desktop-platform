# Distribution Standard

**対象 OS、識別子、署名なし配布、アプリ内アップデート、リリース、CI** を決めます。配布まわりは不一致がそのまま実害（更新が届かない、鍵を失う、タグの取り合い）になるので、Core にしています。

## 1. 対象

| OS | 配布物 | 備考 |
|---|---|---|
| macOS（Apple Silicon） | `<Product>_<version>_aarch64.dmg` と `.app` | Intel Mac 向けは配布しない |
| Windows（x64） | `<Product>_<version>_x64-setup.exe`（NSIS） | `.msi` は作らない |

`bundle.targets` は `["app", "dmg", "nsis"]`。Linux は対象外（CI の runner で Rust テストに使うだけ）。

出どころ：stockroom と Shelf の `tauri.conf.json`、Shelf CHANGELOG 0.1.1（`.msi` と Intel の廃止）。

## 2. 識別子と名前とバージョン

- `identifier` は逆 DNS 形式で、**一度配布したら変えない**。設定・データ・ログのフォルダ名になるので、変えると既存ユーザーのデータが見えなくなる。新規ツールは `decisions/project-context.md` に決めてから初回リリースする。
- `productName` は人が読む名前。インストーラのファイル名と `.app` の名前になる。
- バージョンは **3 か所を同じ値** にする：`package.json`、`src-tauri/tauri.conf.json`、`src-tauri/Cargo.toml`。lockfile も更新する（`bun pm version X --no-git-tag-version`、`cargo update -p <crate>`）。
- SemVer。MAJOR は 0 のままでよい。データ形式の変更があるときは MINOR を上げ、CHANGELOG に書く。

出どころ：stockroom README「リリース」、Shelf README「リリース手順」。

## 3. 署名なし配布

Apple Developer ID と Windows のコード署名は使わない。代わりに次を守る。

- macOS は **アドホック署名**（`bundle.macOS.signingIdentity: "-"`）。「壊れているため開けません」でなく「開発元を検証できない」になり、システム設定から開ける。
- README と CHANGELOG の「注意」に **初回だけ出る警告の手順** を書く。文面は揃える：
  - macOS：`.dmg` を開いて「アプリケーション」へコピー → 起動すると「開発元を検証できない」と出るので「完了」→ **システム設定 › プライバシーとセキュリティ** の下にある「このまま開く」。ターミナルで `xattr -dr com.apple.quarantine "/Applications/<Product>.app"` でもよい
  - Windows：`-setup.exe` を実行 → SmartScreen が出たら「詳細情報」→「実行」
- 2 回目以降はアプリ内アップデートで入れ替える（§4）。アプリ自身が書き出したファイルには quarantine 属性が付かないので、この経路では警告が出ない。

出どころ：stockroom README「インストール」、Shelf README「初回起動」。

## 4. アプリ内アップデート

配布するツールは **必ず** `tauri-plugin-updater` を入れる。

- `bundle.createUpdaterArtifacts: true`。更新ファイル（`.app.tar.gz` と `.sig`、`-setup.exe` の `.sig`）と `latest.json` をリリースに添付する。
- `plugins.updater.endpoints` は 1 つ：単独リポジトリは `releases/latest/download/latest.json`、同居リポジトリは `releases/download/<name>-updater/latest.json`（[Optional](optional/multi-tool-repo.md)）。
- 起動から数秒後（4 秒）に確認する。**release ビルドだけ**。見つかったときだけダイアログで知らせ、「更新して再起動」で適用する。
- 手動の確認を設定画面の「アップデート」の節とメニュー（トレイ常駐ならトレイのメニュー）に置く。手動のときは「最新版です」と失敗も知らせる。自動のときは黙る。
- **設定画面が開いているあいだは、確認の結果をすべて「アップデート」の節に出し、ダイアログもトーストも出さない。** 新しい版が見つかったときも同じで、節に「v`<new>` が使えます」と［更新内容］［更新して再起動］が出る。設定から確かめたときも、設定を開いたまま自動の確認が終わったときもこの形。設定を開いて操作している人の前にダイアログを重ねない。§4.1 のダイアログは、設定画面が開いていないときだけ出す。

### 4.1 知らせ方

前触れなく出るダイアログは、どのアプリからか分からないと怪しく見える。どのツールも同じ形で、アプリの名前を先に出す。

| 部分 | 中身 |
|---|---|
| ダイアログ | OS のダイアログ（`tauri-plugin-dialog`、`kind: info`）。トレイ常駐はアプリを前面にしてから出す |
| タイトル | 「`<Product>` の新しいバージョンがあります」。macOS では太字の 1 行目、Windows ではウィンドウのタイトルになる |
| 本文 | 「v`<new>` が使えます（今のバージョンは v`<current>`）。」改行して「更新すると `<Product>` が再起動します。」 |
| ボタン | 「更新して再起動」「あとで」「更新内容を見る」の 3 つ。並びは OS に合わせる。macOS：［更新内容を見る］［あとで］［**更新して再起動**］、Windows：［**更新して再起動**］［更新内容を見る］［あとで］（Esc は「あとで」） |

- **更新内容（リリースノート）はダイアログに載せない**。長い文がダイアログを広げ、リリースの本文はダウンロードの案内も含むのでアプリの中では意味をなさない。「更新内容を見る」で、そのバージョンの GitHub のリリースのページ（単独リポジトリは `releases/tag/v<version>`、同居リポジトリは `releases/tag/<name>-v<version>`）をブラウザで開く。
- URL は Rust のコマンド（`open_release_page(version)`）で組み立て、フロントから渡すのはバージョンだけにする。`opener` の capability を広げずに済み、任意の URL を開く入口も作らない。
- 「あとで」と「更新内容を見る」はダイアログを閉じる。その版は、設定の「アップデート」の節（「更新して再起動」のボタンが出る）、トレイ常駐ならトレイのメニュー（「アップデートを確認…」が「v`<new>` に更新…」に変わる）からいつでも入れられる。
- **「あとで」のあとにもう一度聞くのは、次の自動の確認だけ。** 見送った版は覚えておかず（「この版をスキップ」は作らない）、次の確認で同じ版か新しい版があれば、もう一度ダイアログを出す。
  - ウィンドウのあるツール：自動の確認は起動したときの 1 回だけ。起動しているあいだに同じダイアログがもう一度出ることはない。
  - トレイ常駐のツール：起動したときと、そのあと 24 時間ごと（壁時計で測る。[トレイ常駐](optional/tray-app.md)）。1 日に 1 回より多くは出ない。
  - 確認やダウンロードの途中に次の確認を始めないので、ダイアログが 2 つ重なることはない。
  - 手動の確認（メニュー、トレイ）は、そのたびに結果を出す。設定画面からの確認は上のとおり節の中に出す。
- 「最新版です」「アップデートに失敗しました」などのダイアログも、タイトルは「`<Product>` のアップデート」にする。
- ウィンドウのあるツールは、更新して起動し直した最初の 1 回だけ「`<Product>` を v`<version>` に更新しました」のトーストを出し、［更新内容を見る］を付ける（前回のバージョンは `localStorage` の `lastVersion`）。何も言わずに再起動すると、何が起きたか分からない。
- 設定で自動確認をオフにできるようにしてよい（stockroom の `autoUpdate`）。
- 確認とダウンロードは OS のプロキシ設定（PAC を含む）を通す。updater の HTTP クライアントは PAC を評価できないので、アプリ側で解決して `proxy` に渡す。
- Windows は `installMode: "passive"`。インストーラが先にアプリを終了させるので、`relaunch` は no-op でよい。macOS は `.app` が差し替わるので再起動する。
- ダウンロードの進捗は 1 秒に数回以上描き直さない。チャンクは 1 秒に数百回届く。

出どころ：stockroom `src/lib/update.ts`、`src-tauri/src/proxy.rs`、`src-tauri/src/commands.rs`（`open_release_page`）、`docs/SETTINGS.md`「アップデートの知らせ方」、Shelf `src-tauri/src/update.rs`。

## 5. リリース

- tag を push すると CI が配布物を作り、GitHub Release を作る。単独リポジトリの tag は `v<version>`、同居リポジトリは `<name>-v<version>`。
- tag と `tauri.conf.json` の version が一致しなければ CI を失敗させる。
- リリースの本文は `CHANGELOG.md` の `## <version>` の節をそのまま使う。節が無ければ失敗させる。節の中は「変更点 / ダウンロード / 注意」の順。
- updater の feed は **publish されたリリース** だけを指す。draft の間は配信されない。draft で作って人が確認してから publish するか、CI がそのまま publish するかはリポジトリで決め、README に書く。
- 更新ファイル（`.sig`）が揃わないリリースは作らない。鍵が無い環境のビルドは updater artifacts を作らず、インストーラだけを出す（§6）。
- 同じバージョンの tag が既にあれば失敗させる。バージョンを上げてからやり直す。

出どころ：stockroom `.github/workflows/build.yml`（tauri-action、draft release）、Shelf `.github/workflows/shelf.yml`（version check、CHANGELOG からの本文、固定リリース）。手順は [リリース Playbook](../playbook/release.md)。

## 6. CI

- **test job** は Linux の runner で `cargo test` と `tsc --noEmit`（と [Governance](governance.md) §4 の残りの gate）。Tauri の Linux 依存（`libwebkit2gtk-4.1-dev` など）を入れる。
- **build job** は macOS（`macos-latest`、`--target aarch64-apple-darwin`）と Windows（`windows-latest`）の matrix。`fail-fast: false`。
- Rust のキャッシュ（`swatinem/rust-cache`）は **`main` だけ保存** する。tag のビルドは `main` のキャッシュを復元できるが、tag で保存しても二度と使われない。
- `main` への push は新しい push で古いビルドを取り消す（`concurrency`）。tag のビルドは取り消さない。
- `**.md` と `docs/**` だけの変更ではビルドしない（tag には path filter が効かないことに注意）。
- 署名鍵の Secret が無いとき（fork からの PR、Secret 設定前）は `createUpdaterArtifacts: false` の設定を重ねてインストーラだけを作る。release job はその場合に失敗させる。
- 配布物は Actions の Artifacts にも上げる（tag でない手動実行の確認用）。
- 再利用 workflow（desktop-base）ができたら各ツールはそれを `uses:` で呼び、ツール名・ディレクトリ・feed の場所だけを渡す。
- JavaScript の action は **Node.js 24 で動く版** を使う。GitHub は action の実行環境の Node.js 20 を廃止しており、古い版のままだと毎回「Node.js 20 is deprecated」の警告が annotation に出て、廃止の完了後は動かなくなる。下の表の版以上にする。警告が出たら、そこに列挙された action を上げる。
- 警告は action の実行環境の話で、`setup-node` の `node-version`（ツールのビルドに使う Node.js、[Stack](../recommendations/stack.md) の「パッケージマネージャ」）とは別。`node-version` を上げても消えない。

| action | Node.js 24 で動く版 |
|---|---|
| `actions/checkout` | `v5` 以上 |
| `actions/setup-node` | `v5` 以上 |
| `oven-sh/setup-bun` | `v2` 以上 |
| `actions/upload-artifact` | `v6` 以上（`v5` は既定が Node.js 20 のまま） |
| `actions/download-artifact` | `v7` 以上（`v6` は既定が Node.js 20 のまま） |
| `actions/cache` | `v5` 以上 |
| `swatinem/rust-cache` | `v2.9.0` 以上。`@v2` が古い版を指していることがあるので、`@v2.9.2` のように patch まで書く |
| `tauri-apps/tauri-action` | `v0.6.0` 以上（`@v0` は最新の v0.x を指す） |

`dtolnay/rust-toolchain` と `lycheeverse/lychee-action` は composite action なので対象外。表に無い action は、その `action.yml` の `runs.using` が `node24` であることを確かめる。

出どころ：stockroom `.github/workflows/build.yml`、Shelf `.github/workflows/shelf.yml`。Node.js 24 の版は各 action のリリースノート（確認日：2026-10-12。`oven-sh/setup-bun` は 2026-10-28）。

## 7. ビルド設定

`src-tauri/Cargo.toml` の release profile は次を既定にする。

```toml
[profile.release]
lto = "thin"       # fat LTO + 1 unit より再ビルドが 2.5 倍速い。バイナリは +2.4 MB、圧縮後 +0.5 MB 程度
opt-level = 3
panic = "abort"
strip = true
```

- Windows の release では `#![cfg_attr(not(debug_assertions), windows_subsystem = "windows")]` でコンソールを出さない。
- `tauri` の features は使うものだけ（`protocol-asset`、必要なら `macos-private-api`、トレイなら `tray-icon`）。

出どころ：stockroom `src-tauri/Cargo.toml`（計測値のコメント）、`src-tauri/src/main.rs`。
