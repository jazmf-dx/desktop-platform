# Project Context

AI と人が実装前に確認する **現在の前提**。機能要件、ロードマップ、画面仕様は `docs/` に置き、ここには書かない。
変更したら日付を更新する。変更の理由を将来説明する価値がある場合だけ、別の ADR を残す。

**更新日**：YYYY-MM-DD

| 項目 | 現在の前提 |
|---|---|
| ツールの種類 | ウィンドウアプリ / トレイ常駐 |
| UI の実装 | React + zustand + Tailwind 4 / 素の TypeScript |
| データの置き場 | `app_data_dir()` の JSON / ユーザーが選ぶフォルダ 1 つ（SQLite）。構造は README「<節名>」 |
| 外との接続 | なし / ブラウザ拡張（port NNNNN）/ MCP（port NNNNN）/ ほか |
| 配布の形 | 単独リポジトリ（tag `v*`、feed `releases/latest`）/ desktop-apps 同居（tag `<name>-v*`、feed `<name>-updater`） |
| 識別子 | `com.example.tool`（一度配布したら変えない） |
| desktop-base | v0.1.0 / 未採用（stockroom の <commit> から写した） |

## 該当しない Standard

- 例：Optional「ローカル受付」は該当なし（外との接続が無い）

## 逸脱

- 例：ADR-0001 <ファイル名>（Standard のどれから、なぜ）
