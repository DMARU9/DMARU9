# Contracts: Profile README Fixes & Improvements

**Date**: 2026-07-31

## Overview

このドキュメントでは、GitHub Actions ワークフローと README.md の間で交換されるデータの契約を定義します。

## Contract 1: Featured Projects Section

### README.md → GitHub Actions

**セクションマーカー**:

```markdown
<!--START_SECTION:featured_projects-->
<!--END_SECTION:featured_projects-->
```

**期待されるフォーマット**:

```markdown
<!--START_SECTION:featured_projects-->
<div align="center">

|                                                                                                   |                                                                                                   |                                                                                                   |
| :-----------------------------------------------------------------------------------------------: | :-----------------------------------------------------------------------------------------------: | :-----------------------------------------------------------------------------------------------: |
| [![Project1](https://img.shields.io/badge/Project1-Description-00C9FF?style=for-the-badge)](url1) | [![Project2](https://img.shields.io/badge/Project2-Description-92FE9D?style=for-the-badge)](url2) | [![Project3](https://img.shields.io/badge/Project3-Description-ff69b4?style=for-the-badge)](url3) |
|                                               ⭐ 10                                               |                                               ⭐ 5                                                |                                               ⭐ 3                                                |
| [![Project4](https://img.shields.io/badge/Project4-Description-00C9FF?style=for-the-badge)](url4) | [![Project5](https://img.shields.io/badge/Project5-Description-92FE9D?style=for-the-badge)](url5) | [![Project6](https://img.shields.io/badge/Project6-Description-ff69b4?style=for-the-badge)](url6) |
|                                               ⭐ 2                                                |                                               ⭐ 1                                                |                                               ⭐ 0                                                |

</div>
<!--END_SECTION:featured_projects-->
```

### GitHub Actions → README.md

**入力データ**:

```json
{
  "repositories": [
    {
      "name": "string",
      "full_name": "string",
      "html_url": "string",
      "description": "string",
      "stargazers_count": "number",
      "language": "string",
      "pushed_at": "string"
    }
  ]
}
```

**出力データ**:

- README.md の `<!--START_SECTION:featured_projects-->` と `<!--END_SECTION:featured_projects-->` の間を更新

**フォールバック フォーマット** (プロジェクトなし時):

```markdown
<!--START_SECTION:featured_projects-->
<div align="center">

> 🏗️ No public projects yet. Stay tuned!

</div>
<!--END_SECTION:featured_projects-->
```

---

## Contract 2: Achievements Section

### README.md → GitHub Actions

**セクションマーカー**:

```markdown
<!--START_SECTION:achievements-->
<!--END_SECTION:achievements-->
```

**期待されるフォーマット**:

```markdown
<!--START_SECTION:achievements-->
<div align="center">

[![Achievement1](https://img.shields.io/badge/🏆_Achievement1-Description-00C9FF?style=for-the-badge)](url1)
[![Achievement2](https://img.shields.io/badge/⭐_Achievement2-Description-92FE9D?style=for-the-badge)](url2)

</div>
<!--END_SECTION:achievements-->
```

**フォールバック フォーマット** (Achievements データなし時):

```markdown
<!--START_SECTION:achievements-->
<div align="center">

> ℹ️ No achievements data available.

</div>
<!--END_SECTION:achievements-->
```

### GitHub Actions → README.md

**入力データ (GraphQL API Response)**:

```json
{
  "data": {
    "user": {
      "achievements": {
        "nodes": [
          {
            "name": "string",
            "description": "string",
            "imageURL": "string",
            "unlockedAt": "string"
          }
        ]
      }
    }
  }
}
```

**出力データ**:

- README.md の `<!--START_SECTION:achievements-->` と `<!--END_SECTION:achievements-->` の間を更新

---

## Contract 3: GitHub Analytics Section

### shields.io Badge URLs

**Stars Badge**:

```
https://img.shields.io/badge/⭐_Stars-{count}-ff69b4?style=for-the-badge&logo=github
```

**Top Languages Badge**:

```
https://github-readme-stats.vercel.app/api/top-langs/?username=DMARU9&layout=compact&theme=transparent&hide_border=true&title_color=00C9FF&text_color=c9d1d9&bg_color=0D1117
```

**Streak Badge**:

```
https://github-readme-streak-stats.herokuapp.com/?user=DMARU9&theme=transparent&hide_border=true&ring=00C9FF&fire=92FE9D&currStreakLabel=00C9FF&bg_color=0D1117
```

---

## Contract 4: About Me Section

### README.md 内のフォーマット

**マーカー**:

````markdown
## 🧑‍💻 About Me

```yaml

```
````

````

**期待されるフォーマット**:
```yaml
name: DMARU9
spec:
  location: Japan 🇯🇵
  stacks:
    backend:
      - Python (FastAPI)
      - Go (Gin)
      - Node.js
    mobile:
      - Flutter / Dart
      - Cross-platform
    database:
      - PostgreSQL
      - Redis
      - SQLite
    tools:
      - Docker
      - Git
      - CI/CD
  status:
    learning: Go, System Design, Clean Architecture
    building: Next-gen API platform with FastAPI + PostgreSQL
    goal: 'Ship quality code that makes a difference 🚀'
````

---

## Contract 5: Error Handling

### API エラー時のフォールバック

**GitHub API エラー**:

```markdown
> ⚠️ Unable to load data from GitHub API. Please try again later.
```

**Achievements 取得失敗**:

```markdown
> ℹ️ No achievements data available.
```

**Featured Projects が空の場合**:

```markdown
> 🏗️ No public projects yet. Stay tuned!
```

---

## Contract 6: shields.io Badge Color Scheme

### カラーパレット

| Purpose    | Color      | HEX     |
| ---------- | ---------- | ------- |
| Primary    | Cyan       | #00C9FF |
| Secondary  | Green      | #92FE9D |
| Stars      | Pink       | #ff69b4 |
| Background | Dark       | #0D1117 |
| Text       | Light Gray | #c9d1d9 |

### バッジスタイル

- 全てのバッジ: `style=for-the-badge`
- ロゴ: `logo=github` (GitHub 関連)
- ロゴカラー: `logoColor=white`
