# Research: Profile README Fixes v2 — GitHub Analytics & Featured Projects

**Created**: 2026-08-01

## 調査概要

`github-readme-stats.vercel.app` の 503 停止問題と、Featured Projects のテーブル表示問題について調査しました。

## 調査結果

### 1. github-readme-stats の 503 停止

**問題**: `github-readme-stats.vercel.app` が Vercel 無料枠の上限到達で停止

**確認方法**:

```bash
curl -s -o /dev/null -w "%{http_code}" "https://github-readme-stats.vercel.app/api?username=DMARU9"
# 出力: 503
```

**エラーメッセージ**: "This Deployment is paused by the owner." + `DEPLOYMENT_PAUSED`

**原因**: Vercel の Hobby プラン（無料）では月間ビルド数に制限があり、超过了でデプロイが一時停止される

**影響範囲**: Stats カードと Top Languages カードの両方が表示されない

### 2. 代替サービスの調査

**確認したサービス**:

| サービス                       | URL                                      | ステータス | 備考                       |
| ------------------------------ | ---------------------------------------- | ---------- | -------------------------- |
| github-readme-stats (公式)     | github-readme-stats.vercel.app           | 503        | 公式インスタンス（停止中） |
| github-readme-stats (フォーク) | github-readme-stats-ruby.vercel.app      | 200        | 稼働中の公開フォーク ✅    |
| github-readme-streak-stats     | github-readme-streak-stats.herokuapp.com | 200        | Streak 統計（稼働中）      |
| github-profile-summary-cards   | github-profile-summary-cards.vercel.app  | 200        | Profile Summary（稼働中）  |
| shields.io                     | img.shields.io                           | 200        | バッジ生成（稼働中）       |

**代替フォークの検索結果**:

- 複数の公開フォークインスタンスをテスト
- `github-readme-stats-ruby.vercel.app` が api と top-langs の両エンドポイントで200 OKを確認
- `github-readme-stats-git-master.vercel.app` は404（デプロイ未存在）
- その他のフォーク（wheat, rosy, cyan, amber, bronze, frost, jade, topaz, indigo）も404

**決定**: `github-readme-stats-ruby.vercel.app` に差し替え

### 3. Featured Projects のテーブル表示問題

**問題**: Markdown テーブルが正しくレンダリングされない

**原因分析**:

1. カラム数の不一致: header は 4列、データ行は 3列や 1列
2. 1プロジェクトを 2行で表現（バッジ行 + スター行）
3. shields.io の `link=` パラメータが GitHub の `<img>` タグでは無効

**shields.io のハイフン区切り問題**:

- `OBSIDIAN-KNOWLEDGE-COMPILER` が `OBSIDIAN: KNOWLEDGE` に分割される
- 解決策: `--` でエスケープ（`OBSIDIAN--KNOWLEDGE--COMPILER`）

### 4. 404 page not found の調査

**現状**: 現時点で再現せず

**可能性**:

- `github-readme-stats` の 503 エラー画像を誤認した可能性
- shields.io の一時的なエラー
- GitHub の Camo プロキシのキャッシュ問題

**対応**: アルテキストを具体的に設定し、画像読み込み失敗時の UX を改善

## 結論

1. `github-readme-stats` の代替フォークに差し替え
2. Featured Projects のテーブル構造を 2カラムに再設計
3. shields.io バッジのハイフンエスケープを実装
4. alt テキストを全画像に追加
