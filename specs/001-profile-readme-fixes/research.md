# Research: Profile README Fixes & Improvements

**Date**: 2026-07-31

## 1. About Me Format Analysis

### Current State

- Kubernetes YAML 風のコードブロック形式を使用
- 不要フィールド: `api_version`, `kind`, `namespace`, `labels`
- コア情報: `name`, `stacks`, `status`

### Best Practice

- GitHub プロフィール README では、箇条書きやテーブルが一般的
- YAML スタイルは視覚的にユニークだが、コピーユーザビリティが劣化
- 決策: YAML スタイルを維持しつつ、不要フィールドを削除

## 2. GitHub Analytics Stars Display

### Current State

- shields.io バッジを使用: `https://img.shields.io/github/stars/DMARU9?style=for-the-badge`
- 0 stars の場合、バッジが非表示になる可能性

### Root Cause

- shields.io の `github/stars` エンドポイントは、0 の場合でもバッジを返す
- しかし、`github-readme-stats` の API では 0 の場合に非表示になる可能性

### Solution

- shields.io のバッジを使用（0 でも表示される）
- または、`github-readme-stats` の `show_icons=true` パラメータで 0 も表示

## 3. Top Languages Display

### Current State

- `github-readme-stats.vercel.app/api/top-langs/` を使用
- `layout=compact` で表示

### Potential Issues

- リポジトリに言語データがない場合、空になる
- API のレスポンスが遅い場合、タイムアウトする

### Solution

- `hide` パラメータで小規模な言語を非表示にして見栄えを改善
- `layout=compact` でコンパクト表示を維持
- フォールバックメッセージを追加

## 4. GitHub Achievements

### Current State

- shields.io の手動バッジを使用（実際の Achievements と関連なし）
- `github-profile-trophy` サービスはメンテナンス中

### Solution: GitHub GraphQL API

- GraphQL クエリで Achievements を取得:
  ```graphql
  query {
    user(login: "DMARU9") {
      achievements(first: 50) {
        nodes {
          name
          description
          imageURL
        }
      }
    }
  }
  ```
- 取得したデータを shields.io バッジとして動的生成

### Shields.io Badge Generation

- パターン: `https://img.shields.io/badge/{name}-{value}-{color}?style=for-the-badge`
- GitHub Actions で README を自動更新

## 5. Featured Projects Auto-Update

### Current State

- "Coming Soon" プレースホルダー
- 手動更新が必要

### Solution: GitHub REST API

- エンドポイント: `GET /users/{username}/repos`
- パラメータ: `sort=stars`, `direction=desc`
- ソート: stars 順 → 同数の場合は `pushed_at` 順
- 最大表示件数: 6 件

### Badge Generation

- 各プロジェクトに shields.io バッジを生成
- パターン: `https://img.shields.io/badge/{name}-{description}-{color}?style=for-the-badge&link={url}`

## 6. GitHub Actions Workflow Design

### UpdateFeaturedProjects.yml

- スケジュール: 6 時間ごと（既存の UpdateReadme.yml と同期）
- 権限: `contents: write`
- ステップ:
  1. リポジトリ一覧を取得
  2. stars 順にソート（同数は更新日順）
  3. 上位 6 件を選択
  4. README.md の Featured Projects セクションを更新
  5. コミット & プッシュ

### UpdateAchievements.yml

- スケジュール: 24 時間ごと（Achievements は頻繁に変わらない）
- 権限: `contents: write`
- ステップ:
  1. GitHub GraphQL API で Achievements を取得
  2. shields.io バッジを生成
  3. README.md の Achievements セクションを更新
  4. コミット & プッシュ

## 7. Color Scheme

### Current Colors

- Primary: `#00C9FF` (Cyan)
- Secondary: `#92FE9D` (Green)
- Background: `#0D1117` (Dark)
- Text: `#c9d1d9` (Light Gray)

### Badge Colors

- Stars: `#ff69b4` (Pink)
- Achievements: `#00C9FF` (Cyan)
- Projects: `#92FE9D` (Green)

## 8. Fallback Messages

### Stars: 0 の場合

- バッジ表示: `0 Stars` (shields.io が自動処理)

### Top Languages: データなしの場合

- フォールバック: `No language data available`

### Achievements: データなしの場合

- フォールバック: `> ℹ️ No achievements data available.`

### Featured Projects: プロジェクトなしの場合

- フォールバック: `> 🏗️ No public projects yet. Stay tuned!`

## 9. Risks & Mitigations

| Risk                    | Impact | Mitigation                                               |
| ----------------------- | ------ | -------------------------------------------------------- |
| GitHub API レート制限   | 高     | GitHub Actions の `GITHUB_TOKEN` を使用（5000 req/hour） |
| shields.io サービス障害 | 中     | フォールバックメッセージを表示                           |
| GraphQL API の変更      | 低     | 定期的なモニタリング                                     |
| README の競合           | 中     | `concurrency` 設定で並列実行を防止                       |

## 10. Decision Log

| Decision                         | Rationale                                      | Alternatives Considered                     |
| -------------------------------- | ---------------------------------------------- | ------------------------------------------- |
| shields.io バッジを採用          | 既存のカラースキームと統一、柔軟なカスタマイズ | GitHub のネイティブバッジ                   |
| GraphQL API で Achievements 取得 | 実際のデータを取得可能                         | github-profile-trophy（不安定）             |
| 6 件の最大表示                   | GitHub pinned と同等、見栄えのバランス         | 3 件（少なすぎる）、9 件（多すぎる）        |
| 6 時間ごとの更新頻度             | 既存ワークフローと同期、API レート制限を考慮   | 1 時間ごと（過剰）、24 時間ごと（遅すぎる） |
