# ブラウザで UI を作る

Rust を動かさず、ブラウザだけで画面を作って確かめる。UI の変更が多いときに `bun run tauri dev` の再ビルドを待たずに済む。

## 仕組み

`src/main.tsx` で、開発モードかつ Tauri の外（`window.__TAURI_INTERNALS__` が無い）なら、`src/dev/mockBackend.ts` のインメモリのモックを `invoke` の代わりに差し込む。

```ts
if (import.meta.env.DEV && !("__TAURI_INTERNALS__" in window)) {
  (await import("./dev/mockBackend")).installMockBackend();
}
```

- モックは `lib/api.ts` と **同じ型** で書く。`api.ts` を変えたらモックも変える。型チェックで食い違いを検出できるように、モックの関数に `api` の型を付ける。
- `isTauri()` で分岐する機能（updater、ログ、OS 通知、タイトルバー）はブラウザでは静かに何もしない。「ブラウザ版ではアップデートを確認できません」のように手動操作だけ知らせる。

## 使い方

```bash
bun run dev                 # http://localhost:1420
```

`.claude/launch.json` に `ui-mock`（`vite --port 1430 --strictPort`）を置いておくと、`tauri dev` の 1420 と同時に開ける。

## できないこと

- ファイルの取り込み、OS のダイアログ、クリップボード、ドラッグで外へ出す。これらはアプリで確かめる。
- 性能。1 万件の一覧の確認は `examples/seed.rs` で作ったデータをアプリで開く。

## モックを保つ

- モックは「それらしい」データを数十件持ち、フィルタと並び替えだけ本物と同じ結果を返す。
- 新しいコマンドを `api.ts` に足したら、同じ PR でモックにも足す。足せないなら `throw` して気づけるようにする。

出どころ：stockroom `src/main.tsx`、`src/dev/mockBackend.ts`、`.claude/launch.json`。
