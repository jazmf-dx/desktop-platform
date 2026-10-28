# Optional Standard: 複数ツール同居リポジトリ

**適用条件**：1 つのリポジトリに複数の小さなツールを置く（workbench）。

単独リポジトリとの違いは、tag と updater の feed が **ツール名で区別** されることだけにする。

## ルール

- ツールは `<name>/` ディレクトリにまとめ、使い方と開発手順はそのディレクトリの `README.md` に書く。リポジトリ直下の README はツールの一覧表だけを持つ。
- 各ツールは自分の `package.json`、`bun.lock`、`src-tauri/Cargo.lock` を持つ。リポジトリ全体の workspace にしない。
- CI はツールごとに `.github/workflows/<name>.yml` を置き、`paths` でそのディレクトリと workflow 自身が変わったときだけ動かす。tag には path filter が効かないので、tag の pattern を別に書く。
- リリースの tag は `<name>-v<version>`。tag と `<name>/src-tauri/tauri.conf.json` の version が違えば失敗させる。
- アプリ内アップデートの feed は **ツールごとの固定リリース `<name>-updater`** の `latest.json`。`releases/latest` は他のツールのリリースと取り合いになるので使わない。
  - 固定リリースは `prerelease: true`、`make_latest: false` にし、リリースのたびに `latest.json` を差し替える。
  - **`<name>-updater` のリリースと tag は消さない。** 配布済みのアプリが更新を見つけられなくなる。
- `workflow_dispatch` に `release` の入力を持たせ、手動で `main` から tag を作ってリリースできるようにしてよい。
- 署名鍵は owner で 1 本を共有する（[Governance](../governance.md) §6）。

## 単独に分けるとき

ツールが大きくなり、リリースの頻度や依存が他と合わなくなったら単独リポジトリに出す。その版からは `releases/latest` を feed にしてよいが、配布済みのアプリは古い feed を見ているので、**古い `<name>-updater` を新しいリポジトリへ向ける `latest.json` を最後に 1 回出す**。

出どころ：desktop-apps（旧 workbench）README「リポジトリの決まりごと」、`.github/workflows/shelf.yml`。
