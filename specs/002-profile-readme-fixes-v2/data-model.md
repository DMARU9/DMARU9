# Data Model: Profile README Fixes v2 — GitHub Analytics & Featured Projects

**Created**: 2026-08-01

## エンティティ定義

### GitHubStatsImage

GitHub Analytics セクションの統計画像を表す。

| 属性          | 型      | 説明                                  |
| ------------- | ------- | ------------------------------------- |
| username      | string  | GitHub ユーザー名 (`DMARU9`)          |
| service_url   | string  | 統計サービスの baseURL (代替フォーク) |
| show_icons    | boolean | アイコン表示の有無                    |
| count_private | boolean | プライベートリポジトリを含むか        |
| theme         | string  | テーマ (`transparent`)                |
| hide_border   | boolean | ボーダー非表示                        |
| title_color   | string  | タイトル色 (`00C9FF`)                 |
| icon_color    | string  | アイコン色 (`92FE9D`)                 |
| text_color    | string  | テキスト色 (`c9d1d9`)                 |
| bg_color      | string  | 背景色 (`0D1117`)                     |
| alt_text      | string  | 画像読み込み時の代替テキスト          |

**URL テンプレート**:

```
{service_url}/api?username={username}&show_icons={show_icons}&count_private={count_private}&theme={theme}&hide_border={hide_border}&title_color={title_color}&icon_color={icon_color}&text_color={text_color}&bg_color={bg_color}
```

### TopLanguagesImage

Top Languages カードを表す。

| 属性        | 型      | 説明                   |
| ----------- | ------- | ---------------------- |
| username    | string  | GitHub ユーザー名      |
| service_url | string  | 統計サービスの baseURL |
| layout      | string  | レイアウト (`compact`) |
| theme       | string  | テーマ                 |
| hide_border | boolean | ボーダー非表示         |
| title_color | string  | タイトル色             |
| text_color  | string  | テキスト色             |
| bg_color    | string  | 背景色                 |
| alt_text    | string  | 代替テキスト           |

**URL テンプレート**:

```
{service_url}/api/top-langs?username={username}&layout={layout}&theme={theme}&hide_border={hide_border}&title_color={title_color}&text_color={text_color}&bg_color={bg_color}
```

### FeaturedProject

Featured Projects テーブルの1行を表す。

| 属性         | 型     | 説明                                        |
| ------------ | ------ | ------------------------------------------- |
| name         | string | リポジトリ名                                |
| url          | string | GitHub リポジトリ URL                       |
| stars        | number | スター数                                    |
| pushed_at    | string | 最終更新日時 (ISO 8601)                     |
| badge_url    | string | shields.io バッジ URL                       |
| badge_color  | string | バッジの背景色                              |
| escaped_name | string | shields.io 用にエスケープされたリポジトリ名 |

**バッジ URL 生成ルール**:

1. リポジトリ名のハイフンを `--` でエスケープ
2. `link=` パラメータは使用しない
3. カラーサイクル: `#00C9FF`, `#92FE9D`, `#ff69b4`

**例**:

```
Input: name="OBSIDIAN-KNOWLEDGE-COMPILER", color="#00C9FF"
Output: https://img.shields.io/badge/OBSIDIAN--KNOWLEDGE--COMPILER-%2300C9FF?style=for-the-badge
```

### FeaturedProjectsTable

Featured Projects セクションのテーブル構造を表す。

| 属性         | 型     | 説明                                       |
| ------------ | ------ | ------------------------------------------ |
| columns      | array  | カラム定義 [`Project`, `Stars`]            |
| alignment    | array  | カラム配置 [`:---:`, `:---:`]              |
| max_rows     | number | 最大表示数 (`4`)                           |
| sort_key     | string | メインソートキー (`stargazers_count` 降順) |
| sub_sort_key | string | サブソートキー (`pushed_at` 降順)          |
| empty_state  | string | 空リポジトリ時の表示 (`non-visible`)       |

**テーブル構造**:

```markdown
|            Project             | Stars |
| :----------------------------: | :---: |
| [![NAME](badge-url)](repo-url) | ⭐ N  |
```

## リレーション

```
GitHubStatsImage ──→ README.md (GitHub Analytics セクション)
TopLanguagesImage ──→ README.md (GitHub Analytics セクション)
FeaturedProject ──→ FeaturedProjectsTable ──→ README.md (Featured Projects セクション)
FeaturedProjectsTable ──→ UpdateFeaturedProjects.yml (GitHub Actions)
```

## 状態遷移

### Featured Project の表示状態

```
[Public Repos] → [Fetch via API] → [Sort] → [Limit to 4] → [Generate Badges] → [Render Table]
                                          ↓
                                    [Empty Result] → [Hide Section]
```

### 統計画像の表示状態

```
[Load Image] → [200 OK] → [Display]
              → [503/Timeout] → [Show alt text]
```
