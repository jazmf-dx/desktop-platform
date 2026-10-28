# Stack

アプリの枠組みとフロントエンドの既定です。掲載条件・拘束力・健全性チェックは [README](README.md) を参照してください。

---

## デスクトップアプリの枠組み

**既定**：Tauri 2 — https://tauri.app/
**ステータス**：推奨 / **確認日**：2026-10-11 / **採用**：stockroom、Shelf

- Rust のバックエンドと OS の WebView。配布物が小さく（stockroom は 12 MB 程度）、macOS と Windows を 1 つのコードで扱える。
- plugin は [Rust](rust.md) の表のものを使う。
- 2.12 系を使っている。2.x の minor は追従してよい。
- **使わなくてよい場面**：GUI の無い CLI。Rust 単体で作る。

---

## UI

**既定**：React 19 + TypeScript（strict） — https://react.dev/
**ステータス**：推奨 / **確認日**：2026-10-11 / **採用**：stockroom

- `tsconfig` は `strict`、`noUnusedLocals`、`noUnusedParameters`、`noFallthroughCasesInSwitch`。`moduleResolution: "bundler"`。
- コンポーネントは関数。クラスコンポーネントは書かない。
- **使わなくてよい場面**：トレイ常駐の小物（Shelf）。画面が 1 枚で状態が少なければ **素の TypeScript**（`様子見`）でよい。React が要る規模になったら本則に戻る。2 系統を増やさない。

---

## 状態管理

**既定**：zustand — https://zustand.docs.pmnd.rs/
**ステータス**：推奨 / **確認日**：2026-10-11 / **採用**：stockroom

- ストアは 1 つ（`src/store.ts`）。モード、表示条件、選択、再読込、トーストを持つ。
- Rust 側の状態と二重に持たない。一覧はクエリの結果をそのまま置き、変更後は再問い合わせする。
- **併用しない**：Redux、Jotai、React Context による状態共有。

---

## CSS

**既定**：Tailwind CSS 4（`@tailwindcss/vite`） — https://tailwindcss.com/
**ステータス**：推奨 / **確認日**：2026-10-11 / **採用**：stockroom

- 色とアニメーションは `index.css` の `@theme` に置く（[UI Standard](../standards/ui.md) §1）。
- CSS-in-JS、CSS Modules は使わない。
- **shadcn/ui は使わない。** Web アプリ向けの Application UI Standard の資産で、デスクトップのトークンと窓の作法が違う。ダイアログやメニューは Tauri の OS ダイアログと自前の小さなコンポーネントで足りている。必要になったら ADR を残して検討する。

---

## アイコン

**既定**：Lucide（`lucide-react`） — https://lucide.dev/
**ステータス**：推奨 / **確認日**：2026-10-11 / **採用**：stockroom

- コマンド表の `icon` に使い、メニューとツールバーで同じアイコンを出す。
- アプリのアイコンは別（`app-icon.svg` から `tauri icon`）。

---

## 大きな一覧

**既定**：TanStack Virtual（`@tanstack/react-virtual`） — https://tanstack.com/virtual
**ステータス**：推奨 / **確認日**：2026-10-11 / **採用**：stockroom

- 位置の計算（グリッド、標準、リスト、見本）は自前の純粋関数で行い、Virtual には位置を渡すだけにする。
- **使わなくてよい場面**：100 件に満たない一覧。

---

## ビルド

**既定**：Vite — https://vite.dev/
**ステータス**：推奨 / **確認日**：2026-10-11 / **採用**：stockroom、Shelf

- Tauri の既定設定（port 1420 固定、`clearScreen: false`、`src-tauri` を watch しない）を使う。
- ブラウザだけで UI を開く開発モードを持つ（[Playbook](../playbook/ui-in-browser.md)）。

---

## パッケージマネージャ

**既定**：Bun
**ステータス**：推奨 / **確認日**：2026-10-28 / **採用**：stockroom、desktop-apps

- `bun.lock` を commit し、CI は `oven-sh/setup-bun` と `bun install --frozen-lockfile`。1 リポジトリに複数のパッケージマネージャの lockfile を置かない（`package-lock.json` は消す）。
- スクリプトは `bun run <name>` で呼ぶ。**`bun test` は Bun 自身のテストランナーなので、`package.json` の `test` は `bun run test`**。
- Node.js 22 も入れる（CI は `actions/setup-node`）。`bun run` は Vite と Tauri の CLI を Node.js で動かす（各 CLI の shebang が `node`）。
- Tauri の CLI と `tauri-action` は Bun に対応している。`tauri-action` は `bun.lock` があり runner に `bun` があれば、自動で `bun` を使う。
- Web アプリ向け Platform の既定（Bun）と同じ。2026-10-11 の時点では npm を既定にしていた（[ADR-0005](../decisions/adr-0005-bun-as-package-manager.md)）。

---

## 非推奨

新規採用しない。既存利用の即時書き換えは求めない。

| もの | 用途 | 理由 | 移行先 |
|---|---|---|---|
| HTML5 Drag and Drop API | アプリ内の並べ替え・フォルダへのドロップ | Windows で Tauri のファイルドロップ受付と両立しない | pointer イベントによる自前の DnD（[UI Standard](../standards/ui.md) §6） |
| `csp: null` | `tauri.conf.json` | Web ビューが無制限になる | [Security Standard](../standards/security.md) §1 の CSP |
| `.msi` | Windows のインストーラ | NSIS と二重になり、更新の経路も増える | NSIS（`-setup.exe`） |
| Intel Mac 向けビルド | 配布 | 手元に検証環境が無い | Apple Silicon のみ |
