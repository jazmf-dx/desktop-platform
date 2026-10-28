# アップデートの署名鍵

Tauri Updater の minisign 鍵の生成、保管、CI への設定、鍵なしのビルド、作り直し。ルールは [Governance](../standards/governance.md) §6 と [Security](../standards/security.md) §7。

## 生成（owner で 1 回だけ）

```bash
bunx tauri signer generate -w ~/.tauri/<owner>.key
```

- 秘密鍵 `~/.tauri/<owner>.key` と公開鍵（コマンドの出力）ができる。パスワードを付けた場合は `_PASSWORD` の Secret にも入れる。
- 既存の鍵（`~/.tauri/image-library.key`。stockroom の改名前の名前のまま）は stockroom と Shelf が共有している。**同じ owner の新しいツールもこの鍵を使う。** ファイル名がツールの名前でも owner の鍵として扱う。
- owner が違うツール（別の個人アカウントや org）は、その owner で 1 回だけ鍵を作り、その owner のツールで共有する。
- 公開鍵は各ツールの `tauri.conf.json` の `plugins.updater.pubkey` に入れる。

## 保管

- 秘密鍵のファイルとパスワードを **パスワードマネージャーにバックアップ** する。PC の `~/.tauri/` だけに置かない。
- リポジトリ、PR、Issue、ログ、チャットに貼らない。
- 失うと配布済みのアプリに更新を届けられなくなる（§作り直し）。

## CI の Secrets

リポジトリ（または org）の Secrets に次を入れる。

| 名前 | 中身 |
|---|---|
| `TAURI_SIGNING_PRIVATE_KEY` | 秘密鍵ファイルの **中身** |
| `TAURI_SIGNING_PRIVATE_KEY_PASSWORD` | パスワード。無ければ空 |

同居リポジトリでは 1 組で全ツールに効く。

## ローカルでのビルド

`createUpdaterArtifacts` が有効なので、鍵なしでは `bun run tauri build` が失敗する。

```bash
# 鍵あり（更新ファイルも作る）
TAURI_SIGNING_PRIVATE_KEY="$(cat ~/.tauri/<owner>.key)" bun run tauri build

# 鍵なし（インストーラだけ）
echo '{"bundle":{"createUpdaterArtifacts":false}}' > /tmp/no-updater.json
bun run tauri build --config /tmp/no-updater.json
```

CI は Secret が無いときに後者へ自動で落ちる（[Distribution](../standards/distribution.md) §6）。

## 作り直し（鍵が漏れた・失った）

鍵を替えると、配布済みのアプリは新しい鍵で署名した更新を受け付けない。順番を守る。

1. **漏れた場合**：新しい鍵を作る。古い鍵で署名した最後の版として、`pubkey` だけを新しい公開鍵に差し替えた版をリリースする。配布済みのアプリはこの版まで古い鍵で更新でき、以後は新しい鍵で検証する。
2. **失った場合**：新しい鍵で署名した版は配布済みのアプリに届かない。README と CHANGELOG に「この版だけ手動で入れ直す」と書き、初回インストールの手順で入れてもらう。以後は新しい鍵で更新できる。
3. どちらも、owner のすべてのツールで `pubkey` と Secrets を揃えて更新する。

出どころ：stockroom README「リリース」、Shelf README「アプリ内アップデートの仕組み」。
