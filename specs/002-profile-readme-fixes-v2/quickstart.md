# Quickstart: Profile README Fixes v2 — GitHub Analytics & Featured Projects

**Created**: 2026-08-01

## 前提条件

- GitHub アカウント（DMARU9）
- リポジトリへのプッシュ権限
- `curl` (API テスト用)
- ブラウザ (目視確認用)

## Step 1: GitHub Analytics の動作確認

### 1.1 Stats カードの確認

```bash
# 公開フォークインスタンスが稼働しているか確認
curl -s -o /dev/null -w "%{http_code}" "https://github-readme-stats-git-master.vercel.app/api?username=DMARU9&show_icons=true&theme=transparent"
# 期待値: 200
```

### 1.2 Top Languages カードの確認

```bash
curl -s -o /dev/null -w "%{http_code}" "https://github-readme-stats-git-master.vercel.app/api/top-langs?username=DMARU9&layout=compact&theme=transparent"
# 期待値: 200
```

### 1.3 目視確認

1. ブラウザで `https://github.com/DMARU9` を開く
2. GitHub Analytics セクションをスクロール
3. Stats カードにスター数・コミット数・PR数・Issue数が表示されることを確認
4. Top Languages カードに使用言語の分布が表示されることを確認

## Step 2: Featured Projects の動作確認

### 2.1 テーブル表示の確認

1. ブラウザで `https://github.com/DMARU9` を開く
2. Featured Projects セクションをスクロール
3. テーブル形式で表示されていることを確認（プレーンテキストではない）
4. 各プロジェクトバッジにリポジトリ名が正しく表示されることを確認

### 2.2 バッジリンクの確認

1. 各プロジェクトバッジをクリック
2. 正しい GitHub リポジトリに遷移することを確認

### 2.3 ハイフンエスケープの確認

```bash
# shields.io バッジURLを直接開いて確認
curl -s "https://img.shields.io/badge/OBSIDIAN--KNOWLEDGE--COMPILER-%2300C9FF?style=for-the-badge" | grep -o 'aria-label="[^"]*"'
# 期待値: aria-label="OBSIDIAN-KNOWLEDGE-COMPILER"
```

## Step 3: GitHub Actions の動作確認

### 3.1 ワークフローの手動実行

1. GitHub で `https://github.com/DMARU9/DMARU9/actions/workflows/UpdateFeaturedProjects.yml` を開く
2. "Run workflow" ボタンをクリック
3. 完了後、README.md が正しく更新されることを確認

### 3.2 README.md の内容確認

```bash
# Featured Projects セクションを確認
grep -A 20 "START_SECTION:featured_projects" README.md
```

**期待される出力例**:

```markdown
<!--START_SECTION:featured_projects-->
<div align="center">

| :---: | :---: |
| [![DEV-LOG-DAILY](https://img.shields.io/badge/DEV--LOG--DAILY-%2300C9FF?style=for-the-badge)](https://github.com/DMARU9/DEV-LOG-DAILY) | ⭐ 0 |
| [![DMARU9](https://img.shields.io/badge/DMARU9-%2392FE9D?style=for-the-badge)](https://github.com/DMARU9/DMARU9) | ⭐ 0 |
| [![DMARU9.github.io](https://img.shields.io/badge/DMARU9.github.io-%23ff69b4?style=for-the-badge)](https://github.com/DMARU9/DMARU9.github.io) | ⭐ 0 |
| [![OBSIDIAN-KNOWLEDGE-COMPILER](https://img.shields.io/badge/OBSIDIAN--KNOWLEDGE--COMPILER-%2300C9FF?style=for-the-badge)](https://github.com/DMARU9/OBSIDIAN-KNOWLEDGE-COMPILER) | ⭐ 0 |

</div>
<!--END_SECTION:featured_projects-->
```

## Step 4: エッジケースの確認

### 4.1 リポジトリ数 0 件時

- テスト用に一時的にリポジトリを非公開にするか、API のレスポンスをシミュレート
- Featured Projects セクションが非表示になることを確認

### 4.2 画像読み込み失敗時

1. ブラウザの開発者ツールでネットワークをオフにする
2. GitHub Analytics セクションの alt テキストが表示されることを確認

## 成功基準

- [ ] SC-001: GitHub Analytics の全画像が 3 秒以内に表示される
- [ ] SC-002: Featured Projects がテーブルとして正しくレンダリングされる
- [ ] SC-003: 全プロジェクトバッジが正しいリポジトリにリンクしている
- [ ] SC-004: ハイフン入りリポジトリ名が正しく表示される
- [ ] SC-005: 画像読み込み失敗時に alt テキストが表示される
