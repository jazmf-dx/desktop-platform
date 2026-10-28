# リリース

バージョンを上げて配布物を作り、アプリ内アップデートで配信するまで。ルールは [Distribution Standard](../standards/distribution.md) §5–6。

## 前提

- GitHub Secrets に署名鍵がある（[アップデートの署名鍵](updater-key.md)）。
- `main` が gate を通っている。

## 手順（単独リポジトリ：stockroom の形）

1. バージョンを 3 か所で上げる。
   ```bash
   bun pm version 0.18.0 --no-git-tag-version       # package.json（bun.lock には版が無い）
   # src-tauri/tauri.conf.json の "version"
   # src-tauri/Cargo.toml の version、そのあと
   cargo update -p stockroom --manifest-path src-tauri/Cargo.toml   # Cargo.lock
   ```
2. `CHANGELOG.md` に `## 0.18.0` の節を書く。変更点 / ダウンロード / 注意 の順。
3. `chore: v0.18.0` で commit し、PR で `main` に入れる。
4. tag を打って push する。
   ```bash
   git tag v0.18.0 && git push origin v0.18.0
   ```
5. CI が macOS と Windows をビルドし、draft のリリースにインストーラと更新ファイルと `latest.json` を添付する。
6. draft の内容を確認して **Publish** する。publish するまで配布済みのアプリは旧バージョンのまま。

## 手順（同居リポジトリ：desktop-apps の形）

1. `<name>/` の中で上と同じく 3 か所のバージョンを上げる（`cargo update -p <crate>`）。
2. `<name>/CHANGELOG.md` に節を書く。
3. PR で `main` に入れる。
4. Actions → `<name>` → **Run workflow** で `main`、`release` にチェックを入れて実行する。`<name>-v<version>` の tag を直接 push しても同じ。
5. CI が tag と version の一致、CHANGELOG の節、更新ファイルの有無を確かめ、`<name> v<version>` のリリースを作って publish し、`<name>-updater` の `latest.json` を差し替える。

## 確認

- リリースに `.dmg`、`-setup.exe`、`.app.tar.gz` と `.sig`、`-setup.exe.sig`、`latest.json` が揃っている。
- 1 つ前のバージョンを入れた環境で起動し、数秒後に更新のダイアログが出て「更新して再起動」で新しい版になる。
- 新しい版を初回インストールして、README の警告の手順どおりに開ける。

## 失敗したとき

| 症状 | 原因 | 対処 |
|---|---|---|
| `Tag ... does not match tauri.conf.json version` | tag と version の不一致 | tag を消して version を直す。既に push した tag は消して打ち直す |
| `No '## x.y.z' section in CHANGELOG.md` | 節が無い | 節を足してから再実行 |
| `Update files are missing` | 署名鍵の Secret が無い、または空 | Secrets を入れて再実行 |
| 更新のダイアログが出ない | draft のまま / feed の URL 違い / 開発ビルド | publish する。`endpoints` を見る。release ビルドで試す |
| 「アップデートに失敗しました」に署名の文言 | `pubkey` と鍵の不一致 | 鍵を変えない。`pubkey` を owner 共通に戻す |
| Windows で更新後に起動しない | NSIS の `installMode` | `passive` にする |
| Rust のビルドが毎回フルで遅い | キャッシュが `main` で保存されていない | `rust-cache` の `save-if` を `main` に |

## 配布済みのアプリを壊さないために

- `identifier`、`pubkey`、`endpoints` を変えない。
- `<name>-updater` のリリースと tag を消さない。
- データ形式を変えるときは古い版で開いても壊れない形で足し、CHANGELOG に書く。
