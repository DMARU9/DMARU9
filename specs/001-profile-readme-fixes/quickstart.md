# Quickstart: Profile README Fixes & Improvements

**Date**: 2026-07-31

## Overview

このドキュメントでは、Profile README の修正と改善を検証するためのクイックスタート手順を説明します。

## Prerequisites

- GitHub アカウント
- ローカル環境でリポジトリをクローン済み
- Git CLI
- テキストエディタ

## 検証手順

### 1. About Me セクションの検証

**手順**:

1. README.md を開く
2. `## 🧑‍💻 About Me` セクションを確認
3. 以下のフィールドが存在しないことを確認:
   - `api_version`
   - `kind`
   - `namespace`
   - `labels`
4. 以下のフィールドが存在することを確認:
   - `name`
   - `spec` (stacks, status)

**期待される結果**:

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
```

---

### 2. GitHub Analytics Stars の検証

**手順**:

1. README.md を開く
2. `## 📊 GitHub Analytics` セクションを確認
3. Stars バッジが表示されることを確認
4. 0 stars の場合でもバッジが非表示にならないことを確認

**期待される結果**:

- Stars バッジが `0 Stars` と表示される
- バッジが `#ff69b4` (ピンク) の背景色で表示される

**検証コマンド**:

```bash
# README で Stars バッジの存在を確認
grep -n "stars" README.md
```

---

### 3. Top Languages の検証

**手順**:

1. README.md を開く
2. `## 📊 GitHub Analytics` セクションを確認
3. Top Languages バッジが表示されることを確認

**期待される結果**:

- Top Languages バッジが言語分布を表示
- `layout=compact` でコンパクトに表示

---

### 4. GitHub Achievements の検証

**手順**:

1. README.md を開く
2. `## 🏆 GitHub Achievements` セクションを確認
3. `<!--START_SECTION:achievements-->` マーカーが存在することを確認
4. GitHub Action を手動実行:
   ```bash
   # GitHub CLI を使用してワークフローを手動実行
   gh workflow run UpdateAchievements.yml
   ```
5. ワークフロー完了後、README.md を再確認

**期待される結果**:

- 実際の GitHub Achievements が shields.io バッジとして表示される
- または、`> ℹ️ No achievements data available.` と表示される

---

### 5. Featured Projects の検証

**手順**:

1. README.md を開く
2. `## 🚀 Featured Projects` セクションを確認
3. `<!--START_SECTION:featured_projects-->` マーカーが存在することを確認
4. GitHub Action を手動実行:
   ```bash
   # GitHub CLI を使用してワークフローを手動実行
   gh workflow run UpdateFeaturedProjects.yml
   ```
5. ワークフロー完了後、README.md を再確認

**期待される結果**:

- 最大 6 件のリポジトリが表示される
- Stars 数順にソートされている
- 同数の場合、更新日順にソートされている

---

### 6. エラーハンドリングの検証

**手順**:

1. GitHub API を呼び出すスクリプトを一時的に無効化
2. README.md のフォールバックメッセージを確認

**期待される結果**:

- API エラー時: `> ⚠️ Unable to load data from GitHub API. Please try again later.`
- Achievements データなし: `> ℹ️ No achievements data available.`
- Featured Projects が空: `> 🏗️ No public projects yet. Stay tuned!`

---

## 自動検証

### Pre-commit Hooks

```bash
# pre-commit をインストール
pip install pre-commit

# hooks をインストール
pre-commit install

# 全ファイルをチェック
pre-commit run --all-files
```

### Markdownlint

```bash
# markdownlint を実行
markdownlint README.md
```

### Link Check

```bash
# lychee を使用してリンクをチェック
lychee README.md
```

## トラブルシューティング

### 問題: バッジが表示されない

**原因**: shields.io サービス障害

**解決策**:

- しばらく待ってからリロード
- shields.io のステータスページを確認: <https://status.shields.io/>

### 問題: GitHub Action が失敗する

**原因**: API レート制限

**解決策**:

- `GITHUB_TOKEN` の権限を確認
- ワークフローの実行履歴を確認: `gh run list`

### 問題: README の形式が崩れる

**原因**: Markdown の構文エラー

**解決策**:

- markdownlint を実行してエラーを修正
- prettier でフォーマット
