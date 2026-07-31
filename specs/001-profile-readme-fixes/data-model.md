# Data Model: Profile README Fixes & Improvements

**Date**: 2026-07-31

## Overview

この機能では、GitHub API から取得したデータを README.md に反映します。データモデルは主に GitHub API のレスポンスに基づきます。

## Entities

### 1. Repository

GitHub のリポジトリ情報を表します。

| Field            | Type     | Description                     |
| ---------------- | -------- | ------------------------------- |
| name             | string   | リポジトリ名                    |
| full_name        | string   | 完全なリポジトリ名 (owner/repo) |
| html_url         | string   | リポジトリの URL                |
| description      | string   | リポジトリの説明                |
| stargazers_count | number   | Stars の数                      |
| language         | string   | 主なプログラミング言語          |
| pushed_at        | string   | 最終更新日時 (ISO 8601)         |
| topics           | string[] | トピックタグ                    |
| fork             | boolean  | フォークかどうか                |
| archived         | boolean  | アーカイブ済みかどうか          |

### 2. Achievement

GitHub の実績情報を表します。

| Field       | Type   | Description         |
| ----------- | ------ | ------------------- |
| name        | string | 実績名              |
| description | string | 実績の説明          |
| imageURL    | string | 実績アイコンの URL  |
| unlockedAt  | string | 解放日時 (ISO 8601) |

### 3. Language

リポジトリのプログラミング言語情報を表します。

| Field | Type   | Description    |
| ----- | ------ | -------------- |
| name  | string | 言語名         |
| bytes | number | 使用バイト数   |
| color | string | 言語の色 (HEX) |

### 4. ProfileStats

プロフィールの統計情報を表します。

| Field          | Type   | Description    |
| -------------- | ------ | -------------- |
| totalStars     | number | 総 Stars 数    |
| totalRepos     | number | 総リポジトリ数 |
| totalFollowers | number | フォロワー数   |
| totalFollowing | number | フォロー数     |

## Relationships

```
ProfileStats 1──* Repository
Repository 1──* Language
ProfileStats 1──* Achievement
```

## Data Flow

```
GitHub API ──> GitHub Actions ──> README.md ──> GitHub.com
     │              │                │
     │              │                └── shields.io バッジ
     │              └── スクリプト処理
     └── REST + GraphQL
```

## Validation Rules

### Repository

- `name` は空であってはならない
- `html_url` は有効な URL でなければならない
- `stargazers_count` は 0 以上でなければならない
- `fork` が `true` のリポジトリは除外する
- `archived` が `true` のリポジトリは除外する

### Achievement

- `name` は空であってはならない
- `imageURL` は有効な URL でなければならない

### Featured Projects ソート

1. `stargazers_count` の降順
2. 同数の場合、`pushed_at` の降順
3. 最大 6 件まで表示

## State Transitions

### Featured Projects

```
[未表示] ── GitHub Action 実行 ──> [表示中]
[表示中] ── Stars 増加 ──> [順位変動]
[表示中] ── アーカイブ ──> [非表示]
```

### Achievements

```
[未取得] ── GraphQL API 呼び出し ──> [取得済み]
[取得済み] ── 新しい実績解放 ──> [更新済み]
```

## API Endpoints

### GitHub REST API

- `GET /users/{username}/repos` - リポジトリ一覧取得
- `GET /repos/{owner}/{repo}/languages` - 言語情報取得

### GitHub GraphQL API

```graphql
query GetAchievements($login: String!) {
  user(login: $login) {
    achievements(first: 50) {
      nodes {
        name
        description
        imageURL
        unlockedAt
      }
    }
  }
}
```

### shields.io

- `https://img.shields.io/badge/{label}-{message}-{color}?style=for-the-badge`
- `https://img.shields.io/github/stars/{owner}/{repo}?style=for-the-badge`

## Storage

この機能では永続化ストレージを使用しません。データは GitHub Actions の実行ごとに API から取得し、README.md に書き込みます。
