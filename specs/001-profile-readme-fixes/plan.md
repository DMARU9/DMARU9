# Implementation Plan: Profile README Fixes & Improvements

**Branch**: `001-profile-readme-fixes` | **Date**: 2026-07-31 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-profile-readme-fixes/spec.md`

## Summary

GitHub プロフィール README の品質向上を実施します。主な変更内容：

1. About Me セクションの Kubernetes YAML スタイルを維持しつつ、不要フィールドを削除してコピーユーザビリティを改善
2. GitHub Analytics の stars バッジが 0 の場合でも正しく表示されるよう修正
3. Top Languages の表示問題を修正
4. GitHub Achievements を GitHub GraphQL API で取得し、shields.io バッジとして表示する GitHub Actions ワークフローを新設
5. Featured Projects を stars 順（同数なら更新日順）で最大 6 件自動表示する GitHub Actions ワークフローを新設

## Technical Context

**Language/Version**: Markdown, YAML (GitHub Actions), Shell (GitHub Actions runner)

**Primary Dependencies**:

- GitHub API (REST + GraphQL)
- shields.io (バッジ生成)
- github-readme-stats (統計バッジ)
- github-profile-summary-cards (プロフィールサマリー)
- GitHub Actions (自動化)

**Storage**: N/A（ファイルベース）

**Testing**:

- Markdownlint (Markdown バリデーション)
- Lychee (リンクチェック)
- 手動ブラウザ確認

**Target Platform**: GitHub.com (Web)

**Project Type**: Profile Repository (GitHub プロフィールリポジトリ)

**Performance Goals**:

- README セクションの読み込み: 3 秒以内
- データの更新頻度: 24 時間以内

**Constraints**:

- 既存のカラースキーム (#00C9FF, #92FE9D) を維持
- terminal/hacker アイゼティックを維持
- 機能の肥大化を避ける (Constitution: Simplicity)

**Scale/Scope**:

- 変更対象: README.md, GitHub Actions workflows
- 最大表示プロジェクト数: 6 件

## Constitution Check

| Principle                         | Status  | Notes                                  |
| --------------------------------- | ------- | -------------------------------------- |
| I. Profile Quality (品質第一)     | ✅ Pass | README の品質向上が本機能の主目的      |
| II. Git Validation (Git検証)      | ✅ Pass | pre-commit hooks で検証                |
| III. Code Quality (コード品質)    | ✅ Pass | markdownlint, prettier で検証          |
| IV. Automation First (自動化優先) | ✅ Pass | GitHub Actions で自動化                |
| V. Simplicity (シンプルさ)        | ✅ Pass | プロフィールリポジトリとして適切な範囲 |

## Project Structure

### Documentation (this feature)

```text
specs/001-profile-readme-fixes/
├── spec.md              # 仕様書
├── plan.md              # このファイル
├── research.md          # Phase 0 出力
├── data-model.md        # Phase 1 出力
├── quickstart.md        # Phase 1 出力
├── contracts/           # Phase 1 出力
└── tasks.md             # Phase 2 出力
```

### Source Code (repository root)

```text
DMARU9/
├── README.md                              # メインプロフィール README
├── .github/
│   └── workflows/
│       ├── UpdateReadme.yml               # 既存: 最近のアクティビティ更新
│       ├── GenerateContributionSnake.yml  # 既存: コントリビューショーンヘビ
│       ├── LinkCheck.yml                  # 既存: リンクチェック
│       ├── UpdateFeaturedProjects.yml     # 新規: Featured Projects 自動更新
│       └── UpdateAchievements.yml         # 新規: Achievements 自動更新
└── .specify/
    └── memory/
        └── constitution.md                # プロジェクト規範
```

**Structure Decision**: プロフィールリポジトリであるため、ソースコードは主に README.md と GitHub Actions workflows で構成されます。新しいワークフローは既存の `.github/workflows/` ディレクトリに追加します。

## Complexity Tracking

> Constitution Check に違反はなし。複雑性の正当化は不要。
