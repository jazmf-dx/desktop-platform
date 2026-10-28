# Decisions

この Platform **自体** の ADR（Architecture Decision Record）です。各ツールの判断は、そのツールの `decisions/` に置きます。

| ADR | 決定 |
|---|---|
| [ADR-0001](adr-0001-separate-from-web-platform.md) | Web アプリ向けの ai-dev-platform と分け、参照も共有もしない |
| [ADR-0002](adr-0002-rules-and-base-repositories.md) | ルール（desktop-platform）とコード（desktop-base）を 2 つのリポジトリに分ける |
| [ADR-0003](adr-0003-core-and-optional-standards.md) | Core / Optional Standard と Recommendation の区分の基準 |
| [ADR-0004](adr-0004-layout-stability-ahead-of-source.md) | 「表示を切り替えても操作の位置を変えない」を、出どころの実装より先に UI Standard へ入れる |
| [ADR-0005](adr-0005-bun-as-package-manager.md) | パッケージマネージャの既定を npm から Bun にする |

## 書き方

- 番号は 4 桁の連番。ファイル名は `adr-NNNN-<slug>.md`。
- 雛形は [adr-template.md](adr-template.md)。状況、決定、理由、影響、見直す条件を書く。
- 決定を覆すときは新しい ADR を書き、古いものに「ADR-NNNN で置き換え」と足す。消さない。
