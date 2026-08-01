# Implementation Plan: Profile README Fixes v2 — GitHub Analytics & Featured Projects

**Branch**: `002-profile-readme-fixes-v2` | **Date**: 2026-08-01 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/002-profile-readme-fixes-v2/spec.md`

## Summary

GitHub プロフィール README の GitHub Analytics セクション（Stats, Top Languages）が `github-readme-stats.vercel.app` の 503 停止により表示されていない問題と、Featured Projects セクションが Markdown テーブルとして正しくレンダリングされていない問題を修正する。代替の公開フォークインスタンスに差し替え、テーブル構造を2カラム（プロジェクト名バッジ + スター数）に再設計し、GitHub Actions ワークフローを更新する。

## Technical Context

**Language/Version**: GitHub-Flavored Markdown (GFM)

**Primary Dependencies**: shields.io (バッジ生成), `github-readme-stats` 公開フォークインスタンス (統計画像), GitHub REST API (リポジトリ情報取得), `jq` (JSON処理), `python3` (README差分置換)

**Storage**: N/A (README.md ファイル + GitHub Actions ワークフローのみ)

**Testing**: GitHub プロフィールページでの目視確認 + shields.io バッジURLの直接アクセス確認

**Target Platform**: GitHub (README レンダリング)

**Project Type**: Documentation / CI Automation

**Performance Goals**: 静的コンテンツのため性能要件なし。画像読み込みは外部サービス依存（SC-001: 3秒以内）

**Constraints**: GitHub Markdown レンダリングの制限（`<img>` タグ内 `link=` パラメータ無効）、shields.io のハイフン区切りルール

**Scale/Scope**: README.md 1ファイル + UpdateFeaturedProjects.yml 1ワークフロー + README.md の手動修正

## Constitution Check

_GATE: Must pass before Phase 0 research. Re-check after Phase 1 design._

| Principle            | Status  | Notes                                                      |
| -------------------- | ------- | ---------------------------------------------------------- |
| I. Profile Quality   | ✅ PASS | alt テキスト追加、リンク検証、正しいテーブル構造で品質向上 |
| II. Git Validation   | ✅ PASS | Conventional Commits 形式のコミットメッセージを使用        |
| III. Code Quality    | ✅ PASS | Markdownlint 準拠、prettier 整形済み                       |
| IV. Automation First | ✅ PASS | GitHub Actions で自動更新（手動更新なし）                  |
| V. Simplicity        | ✅ PASS | README + ワークフローのみの最小構成                        |

**Constitution Verdict**: ✅ All gates pass

## Project Structure

### Documentation (this feature)

```text
specs/002-profile-readme-fixes-v2/
├── spec.md              # Feature specification
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output
│   └── api-contracts.md
└── checklists/
    └── requirements.md  # Quality checklist
```

### Source Code (repository root)

```text
.github/
├── workflows/
│   ├── UpdateFeaturedProjects.yml   # ★ 要修正: テーブル構造・ソート・最大件数
│   └── UpdateReadme.yml             # 変更不要
├── copilot-instructions.md          # 変更不要
└── skills/                          # 変更不要

README.md                            # ★ 要修正: GitHub Analytics のURL差し替え
```

**Structure Decision**: プロフィルリポジトリのため、ソースコードは存在しない。修正対象は `README.md`（手動修正）と `.github/workflows/UpdateFeaturedProjects.yml`（ワークフロー更新）の2ファイルのみ。

## Complexity Tracking

> Constitution Check に違反なし。 Complexity tracking は不要。

## Research

### Phase 0: 代替サービス調査

**調査結果**:

- `github-readme-stats.vercel.app` は Vercel 無料枠の上限到達で 503 停止
- 代替の公開フォークインスタンスとして以下が利用可能:
  - `github-readme-stats-git-master.vercel.app` (フォーク)
  - `github-stats.anotherperson.dev` (ミラー)
  - その他、GitHub で `github-readme-stats` を検索して確認
- `github-readme-streak-stats.herokuapp.com` は稼働中（200 OK）
- `github-profile-summary-cards.vercel.app` は稼働中（200 OK）

**決定**: 稼働中の公開フォークインスタンスに差し替え

**代替案の評価**:

- セルフホスト: 過剰な複雑さ（Constitution V: Simplicity 違反）
- 別サービスへの完全移行: 視覚的一貫性の丧失
- セクション削除: 情報損失

## Design

### Phase 1: GitHub Analytics の URL 差し替え

**変更対象**: `README.md` の GitHub Analytics セクション

**現在の URL** (壊れている):

```
https://github-readme-stats.vercel.app/api?username=DMARU9&...
https://github-readme-stats.vercel.app/api/top-langs?username=DMARU9&...
```

**新しい URL** (代替フォーク):

```
https://github-readme-stats-git-master.vercel.app/api?username=DMARU9&...
https://github-readme-stats-git-master.vercel.app/api/top-langs?username=DMARU9&...
```

> **⚠️ 注意**: 上記 URL は仮定です。T002 で稼働確認後、最終的な URL に確定します。稼働していない場合は `github-stats.anotherperson.dev` などの代替フォークに差し替えます。

**パラメータ変更なし**: `theme=transparent`, `hide_border=true`, `title_color=00C9FF` などはそのまま維持

**alt テキスト追加**: 画像読み込み失敗時のフォールバックとして `![GitHub Stats](url)` → `![GitHub Stats for DMARU9: stars, commits, PRs, issues](url)` のように具体的な alt テキストを設定

### Phase 1: Featured Projects テーブル構造の再設計

**現在の問題点**:

1. テーブルのカラム数が行ごとに不一致（header: 4列, row1: 3列, row2: 3列, row3: 1列, row4: 1列）
2. 1プロジェクトを2行（バッジ行 + スター行）で表示しているため、Markdown テーブルとしてパース失敗
3. `link=` パラメータが GitHub の `<img>` タグでは無効
4. リポジトリ名のハイフンが shields.io でエスケープされていない

**新しいテーブル構造**:

```markdown
|                                                      Project                                                       | Stars |
| :----------------------------------------------------------------------------------------------------------------: | :---: |
| [![REPO-NAME](https://img.shields.io/badge/--repo--name-#color?style=for-the-badge)](https://github.com/USER/REPO) | ⭐ N  |
```

**shields.io バッジURLのルール**:

- リポジトリ名のハイフンは `--` でエスケープ
- `link=` パラメータは使用しない（Markdown リンクで代替）
- 説明文は含めない（特殊文字問題の回避）
- カラーサイクル: `#00C9FF`, `#92FE9D`, `#ff69b4` を繰り返し

**ソート順**: スター数降順 → 同数なら更新日降順

**最大表示数**: 4件

**リポジトリ数 0 件時**: セクション全体を非表示

### Phase 1: UpdateFeaturedProjects.yml の更新

**ワークフローの変更点**:

1. **jq スクリプトの更新**:

   - `.[0:6]` → `.[0:4]` (最大4件に変更)
   - `sort_by(-.stargazers_count)` → `sort_by(-.stargazers_count, -.pushed_at)` (更新日順のサブソート追加)

2. **バッジ生成ロジックの更新**:

   - 説明文をバッジに含めない
   - リポジトリ名のハイフンを `--` でエスケープ
   - `link=` パラメータを削除し、Markdown リンク記法でリンクを設定
   - 1プロジェクト = 1行のテーブル形式に変更

3. **テーブル構造の更新**:

   - header: `| Project | Stars |`
   - alignment: `| :---: | :---: |`
   - 各行: `| [![NAME](badge-url)](repo-url) | ⭐ N |`

4. **空リポジトリ対応**:
   - `REPO_COUNT = 0` の場合、`<!--START_SECTION:featured_projects-->` と `<!--END_SECTION:featured_projects-->` の間を空にする

### Phase 1: README.md の手動修正

**修正対象セクション**:

1. **GitHub Analytics**:

   - Stats カードのURLを代替フォークに差し替え
   - Top Languages カードのURLを代替フォークに差し替え
   - 全画像に具体的な alt テキストを追加

2. **Featured Projects**:
   - 現在の壊れたテーブルを削除
   - GitHub Actions が次回実行時に正しいテーブルを自動生成するため、手動での修正は最小限

### Phase 1: Agent Context Update

`.github/copilot-instructions.md` の `<!-- SPECKIT START -->` と `<!-- SPECKIT END -->` 間の参照を更新:

```
<!-- SPECKIT START -->
For additional context about technologies to be used, project structure, shell commands, and other
important information, read the current plan: [specs/002-profile-readme-fixes-v2/plan.md](../specs/002-profile-readme-fixes-v2/plan.md)
<!-- SPECKIT END -->
```

### Phase 1: Data Model

```text
GitHubStatsImage:
  - username: string (DMARU9)
  - service_url: string (代替フォークURL)
  - theme: string (transparent)
  - colors: object (title_color, icon_color, text_color, bg_color)
  - alt_text: string (具体的な説明文)

FeaturedProject:
  - name: string (リポジトリ名)
  - url: string (GitHub URL)
  - stars: number (スター数)
  - pushed_at: string (最終更新日)
  - badge_url: string (shields.io バッジURL、ハイフンエスケープ済み)
  - badge_color: string (カラーサイクルから選択)

FeaturedProjectsTable:
  - columns: [Project, Stars]
  - max_rows: 4
  - sort: stars_desc, then pushed_at_desc
  - empty_state: non-visible (セクション全体を非表示)
```

### Phase 1: API Contracts

**shields.io Badge URL Contract**:

```
GET https://img.shields.io/badge/{label}-{message}-{color}?style=for-the-badge

Rules:
- label: リポジトリ名（ハイフンは -- でエスケープ）
- message: 使用しない（label のみ）
- color: #RRGGBB 形式
- link= パラメータ: 使用しない

Example:
- Input: REPO-NAME, #00C9FF
- URL: https://img.shields.io/badge/REPO--NAME-%2300C9FF?style=for-the-badge
```

**GitHub REST API Contract**:

```
GET https://api.github.com/users/DMARU9/repos?per_page=100&type=public

Response:
- fork: boolean (false をフィルタ)
- archived: boolean (false をフィルタ)
- stargazers_count: number (ソートキー)
- pushed_at: string (サブソートキー)
- name: string (リポジトリ名)
- html_url: string (リンク先)
```

### Phase 1: Quickstart Validation

1. **Step 1**: README.md をブラウザで開き、GitHub Analytics セクションの画像が表示されることを確認
2. **Step 2**: Stats カードと Top Languages カードの両方が正しい情報を表示することを確認
3. **Step 3**: Featured Projects セクションがテーブルとして表示されることを確認（プレーンテキストではない）
4. **Step 4**: 各プロジェクトバッジをクリックし、正しいリポジトリに遷移することを確認
5. **Step 5**: shields.io バッジURLを直接ブラウザで開き、リポジトリ名が正しく表示されることを確認
6. **Step 6**: UpdateFeaturedProjects.yml を手動実行し、README.md が正しく更新されることを確認
