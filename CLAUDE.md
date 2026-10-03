# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 概要

Qiita 記事を [Qiita CLI](https://github.com/increments/qiita-cli) (`@qiita/qiita-cli`) で GitHub 管理するためのリポジトリ。コードではなく Markdown 記事が本体。

- 記事は `public/<id>.md` に 1 ファイル 1 記事で置く。ファイル名 (basename) は Qiita の記事 ID と一致している。
- 現在の記事はすべて Organization `ec-cube` 所属 (`organization_url_name: ec-cube`)。EC-CUBE 名古屋などの勉強会資料が中心で、一部は `slide: true` のスライドモード記事。
- `public/.remote/` は `qiita pull` / preview が使うリモート側キャッシュで `.gitignore` 済み。編集対象ではない。

## コマンド

```bash
npm install                     # Qiita CLI の導入
npx qiita preview               # ローカルプレビュー (qiita.config.json により http://localhost:8888)
npx qiita new <basename>        # 新規記事ファイルを public/ に作成
npx qiita pull                  # Qiita 上の状態と記事ファイルを同期
npx qiita publish <basename>    # 単一記事を投稿・更新 (要 qiita login またはトークン)
```

ビルド・lint・テストは存在しない。記事の表示確認は `npx qiita preview` で行う。

## 公開フロー (重要)

- `.github/workflows/publish.yml` により **`main` への push で `qiita publish --all` 相当が走り、Qiita 上の記事が即座に投稿・更新される** (secret `QIITA_TOKEN` を使用)。`main` への push は外部公開と同義なので、push 前に必ずユーザーの確認を取る。
- ワークフローは `contents: write` 権限を持ち、新規記事の投稿時に採番された `id` 等を記事ファイルへ書き戻してコミットする (qiita-cli publish action の挙動)。そのため push 後はリモートに追加コミットが生えている可能性があるので、次の作業前に `git pull` する。

## フロントマター

各記事の先頭 YAML は Qiita CLI が管理する。主なフィールド:

- `title`, `tags` — 記事メタ情報。
- `private` — 限定共有記事かどうか。`qiita.config.json` の `includePrivate: false` により、限定共有記事は pull/同期対象外。
- `id` — Qiita の記事 ID。新規記事では `null` のままにしておき、公開時に CLI が埋める。手で書き換えない。
- `updated_at` — CLI が管理。手で書き換えない (Qiita 側との差分判定に使われる)。
- `organization_url_name` — 所属 Organization (`ec-cube`)。
- `slide` — `true` でスライドモード表示。
- `ignorePublish` — `true` にすると `publish --all` (= CI) の対象から外れる。下書きを `main` に置きたい場合に使う。
