# ADR-0005: パッケージマネージャの既定を Bun にする

**状態**：採用
**日付**：2026-10-28

## 状況

[Stack](../recommendations/stack.md) の「パッケージマネージャ」は npm を既定にしていた（2026-10-11）。理由は「Tauri の CLI と `tauri-action` が npm を既定にしており、既存 2 リポジトリも npm のため」だった。

一方、Web アプリ向けの ai-dev-platform は Bun を既定にしている。同じ人と AI が Web とデスクトップを行き来するので、パッケージマネージャが違うと、コマンド（`npm test` と `bun run test` など）と lockfile の扱いを毎回切り替えることになる。

## 決定

デスクトップツールのパッケージマネージャの既定を **Bun** にする。lockfile は `bun.lock`、CI は `oven-sh/setup-bun` と `bun install --frozen-lockfile`。Node.js 22 も入れたままにする。

## 理由

- npm を選んだ理由のうち、`tauri-action` については前提が変わっていた。`tauri-action` は `bun.lock`（または `bun.lockb`）があり runner に `bun` があれば自動で `bun` を使う（`src/utils.ts` の `usesBun`、確認日 2026-10-28）。Tauri の CLI（`@tauri-apps/cli`）は OS ごとのビルド済み本体を入れるパッケージで、Bun からも同じに動く。
- 残る理由は「既存のリポジトリが npm」だけで、これは移行すれば解消する。
- Web と同じにすることで、コマンドと lockfile の扱いを 1 つ覚えれば済む。[ADR-0001](adr-0001-separate-from-web-platform.md) の「参照も共有もしない」はそのまま。Web の Platform を参照するのではなく、同じ選択をこちらにも書く。
- Node.js を残すのは、`bun run` が Vite と Tauri の CLI を Node.js で動かすため（各 CLI の shebang が `node`）。Bun の互換に頼るより、今までと同じ実行環境のほうが安全。

退けた案：

- **npm のまま**：Web との差が残る。速さの得は小さいが、揃える得のほうが大きい。
- **Node.js を外し Bun だけにする**（`bun --bun`）：Vite と Tauri の CLI が Bun の互換層で動くことになる。差が出たときに原因を切り分けにくい。

## 影響

- Recommendation の変更（[ADR-0003](adr-0003-core-and-optional-standards.md)）。ツールが npm のままでも他に害は無いが、既存のツール（stockroom、desktop-apps）は合わせて移行する。
- 変わるファイル：Stack、Governance §2・§4、Distribution §2・§5、Architecture、UI のアイコン、Optional の複数ツール同居、雛形の `CLAUDE.md`、Playbook（新しいツール・リリース・署名鍵・ブラウザで UI）。
- **`bun test` は Bun のテストランナー** なので、`package.json` の `test`（`cargo test` と `tsc --noEmit`）は `bun run test` で呼ぶ。

## 見直す条件

- `tauri-action` か Tauri の CLI が Bun を外す、または Bun で動かない不具合が 2 回以上続いた場合。
- Web アプリ向け Platform の既定が Bun から変わった場合。
