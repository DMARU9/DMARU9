# API Contracts: Profile README Fixes v2 — GitHub Analytics & Featured Projects

**Created**: 2026-08-01

## shields.io Badge URL Contract

### リクエスト

```
GET https://img.shields.io/badge/{label}-{message}-{color}?style={style}
```

### パラメータ

| パラメータ | 型     | 必須 | 説明                                                       |
| ---------- | ------ | ---- | ---------------------------------------------------------- |
| label      | string | Yes  | バッジの左側に表示されるテキスト（リポジトリ名）           |
| message    | string | No   | バッジの右側に表示されるテキスト（今回は未使用）           |
| color      | string | Yes  | バッジの背景色 (`#RRGGBB` 形式、`#` は `%23` にエンコード) |
| style      | string | No   | バッジスタイル (`for-the-badge`)                           |

### エスケープルール

1. **ハイフンエスケープ**: リポジトリ名の `-` は `--` に変換
   - `OBSIDIAN-KNOWLEDGE-COMPILER` → `OBSIDIAN--KNOWLEDGE--COMPILER`
2. **カラーエンコード**: `#` は `%23` に変換
   - `#00C9FF` → `%2300C9FF`
3. **スペース**: `_` に変換（今回はスペースなし）

### レスポンス

```xml
<svg xmlns="http://www.w3.org/2000/svg" width="..." height="28" role="img" aria-label="...">
  <title>...</title>
  <g shape-rendering="crispEdges">
    <rect width="..." height="28" fill="#555"/>
    <rect x="..." width="..." height="28" fill="{color}"/>
  </g>
  <g fill="#fff" text-anchor="middle" ...>
    <text ...>{label}</text>
  </g>
</svg>
```

### 使用例

**リポジトリ名にハイフンなし**:

```
Input: name="DMARU9", color="#92FE9D"
URL: https://img.shields.io/badge/DMARU9-%2392FE9D?style=for-the-badge
```

**リポジトリ名にハイフンあり**:

```
Input: name="OBSIDIAN-KNOWLEDGE-COMPILER", color="#00C9FF"
URL: https://img.shields.io/badge/OBSIDIAN--KNOWLEDGE--COMPILER-%2300C9FF?style=for-the-badge
```

---

## GitHub REST API Contract

### リポジトリ一覧取得

```
GET https://api.github.com/users/{username}/repos?per_page={per_page}&type={type}
```

### パラメータ

| パラメータ | 型     | 必須 | 説明                                            |
| ---------- | ------ | ---- | ----------------------------------------------- |
| username   | string | Yes  | GitHub ユーザー名 (`DMARU9`)                    |
| per_page   | number | No   | 1ページあたりの件数 (デフォルト: 30, 最大: 100) |
| type       | string | No   | リポジトリタイプ (`public`)                     |

### レスポンス (配列)

```json
[
  {
    "name": "repository-name",
    "html_url": "https://github.com/DMARU9/repository-name",
    "description": "リポジトリの説明",
    "fork": false,
    "archived": false,
    "stargazers_count": 5,
    "pushed_at": "2026-08-01T00:00:00Z"
  }
]
```

### フィルタリング条件

1. `fork == false` (フォークを除外)
2. `archived == false` (アーカイブを除外)

### ソート順

1. `stargazers_count` 降順 (スター数が多い順)
2. `pushed_at` 降順 (同数なら最近更新された順)

### 使用例

```bash
# リポジトリ一覧を取得し、フィルタリング・ソートして上位4件を取得
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/users/DMARU9/repos?per_page=100&type=public" | \
  jq '[.[] | select(.fork == false and .archived == false)] | sort_by(-.stargazers_count, -.pushed_at) | .[0:4]'
```

---

## Markdown Table Contract

### テーブル構造

```markdown
|            Project             | Stars |
| :----------------------------: | :---: |
| [![NAME](badge-url)](repo-url) | ⭐ N  |
```

### カラム定義

| カラム  | タイプ           | 幅                 | 説明                                 |
| ------- | ---------------- | ------------------ | ------------------------------------ |
| Project | リンク付きバッジ | `:---:` (中央揃え) | リポジトリ名バッジ（クリックで遷移） |
| Stars   | テキスト         | `:---:` (中央揃え) | スター数 (`⭐ N` 形式)               |

### 行数制限

- 最大: 4行 (リポジトリ4件)
- 最小: 0行 (リポジトリ0件時はセクション非表示)

### エンティティリレーション

```
FeaturedProject (1) → Markdown Table Row (1)
FeaturedProject (0..4) → FeaturedProjectsTable (1)
FeaturedProjectsTable → README.md (Featured Projects セクション)
```
