# Tasks: Profile README Fixes v2 — GitHub Analytics & Featured Projects

**Input**: Design documents from `/specs/002-profile-readme-fixes-v2/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: テストタスクは含みません。仕様にテスト要件が明記されていないため。視覚確認は各ストーリーの Independent Test でカバー。

**Organization**: ユーザーストーリーごとにグループ化。各ストーリーは独立して実装・テスト可能。

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 並列実行可能（異なるファイル、依存なし）
- **[Story]**: 所属するユーザーストーリー（例: US1, US2, US3）
- ファイルパスを明記

## Phase 1: Setup (共有インフラ)

**Purpose**: ブランチ作成と前提確認

- [x] T001 既存の README.md と .github/workflows/UpdateFeaturedProjects.yml の現在の状態をバックアップ確認 — [#19](https://github.com/DMARU9/DMARU9/issues/19)

---

## Phase 2: Foundational (ブロッキング前提条件)

**Purpose**: 代替サービスの稼働確認。このフェーズ完了前はユーザーストーリー作業不可

**⚠️ CRITICAL**: 代替サービスが稼働していることが全タスクの前提

- [ ] T002 [P] `github-readme-stats` の公開フォークインスタンスが稼働していることを確認（HTTP 200 応答）。確認結果をresearch.mdに記録 — [#20](https://github.com/DMARU9/DMARU9/issues/20)
- [ ] T003 [P] `github-readme-streak-stats.herokuapp.com` が稼働していることを確認（HTTP 200 応答） — [#21](https://github.com/DMARU9/DMARU9/issues/21)
- [ ] T004 [P] `github-profile-summary-cards.vercel.app` が稼働していることを確認（HTTP 200 応答） — [#22](https://github.com/DMARU9/DMARU9/issues/22)

**Checkpoint**: 全サービスが稼働中。ユーザーストーリー作業を開始可能

---

## Phase 3: User Story 1 - GitHub Analytics 統計画像の正常表示 (Priority: P1) 🎯 MVP

**Goal**: Stats と Top Languages の画像が公開フォークインスタンスから正しく表示される

**Independent Test**: GitHub プロフィールページを表示し、Stats, Top Languages が正常にレンダリングされることを確認

### Implementation for User Story 1

- [ ] T005 [US1] README.md の Stats カードURLを `github-readme-stats.vercel.app` → 稼働中の公開フォークインスタンスに差し替え。alt テキスト `![Stats]` → `![GitHub Stats for DMARU9: stars, commits, PRs, issues]` を追加 — [#23](https://github.com/DMARU9/DMARU9/issues/23)
- [ ] T006 [US1] README.md の Top Languages カードURLを `github-readme-stats.vercel.app` → 稼働中の公開フォークインスタンスに差し替え。alt テキスト `![Languages]` → `![Top Languages for DMARU9]` を追加 — [#24](https://github.com/DMARU9/DMARU9/issues/24)
- [ ] T007 [US1] Streak カードの alt テキストを `![Streak]` → `![GitHub Streak for DMARU9]` に更新（サービスは稼働中のためURL変更なし） — [#25](https://github.com/DMARU9/DMARU9/issues/25)
- [ ] T008 [US1] Profile Summary カードの alt テキストを具体的な説明に更新（サービスは稼働中のためURL変更なし） — [#26](https://github.com/DMARU9/DMARU9/issues/26)

**Checkpoint**: GitHub Analytics セクションの全画像が正常に表示される

---

## Phase 4: User Story 2 - Featured Projects テーブル表示の修正 (Priority: P1)

**Goal**: Featured Projects が正しい2カラムMarkdownテーブルとして表示され、バッジがクリック可能に

**Independent Test**: GitHub プロフィールページで Featured Projects がテーブル形式で表示され、バッジクリックでリポジトリに遷移することを確認

### Implementation for User Story 2

- [ ] T009 [P] [US2] .github/workflows/UpdateFeaturedProjects.yml の jq スクリプトを更新: `sort_by(-.stargazers_count)` → `sort_by(-.stargazers_count, -.pushed_at)` に変更（更新日順のサブソート追加） — [#27](https://github.com/DMARU9/DMARU9/issues/27)
- [ ] T010 [P] [US2] .github/workflows/UpdateFeaturedProjects.yml の jq スクリプトを更新: `.[0:6]` → `.[0:4]` に変更（最大表示数を4件に制限） — [#28](https://github.com/DMARU9/DMARU9/issues/28)
- [ ] T011 [US2] .github/workflows/UpdateFeaturedProjects.yml のバッジ生成ロジックを全面改修: 説明文をバッジに含めない、`link=` パラメータを削除しMarkdownリンク記法で実装、1プロジェクト=1行のテーブル形式に変更 — [#29](https://github.com/DMARU9/DMARU9/issues/29)
- [ ] T012 [US2] .github/workflows/UpdateFeaturedProjects.yml のテーブル構造を更新: header `| Project | Stars |`、alignment `| :---: | :---: |`、各行 `| [![NAME](badge-url)](repo-url) | ⭐ N |` — [#30](https://github.com/DMARU9/DMARU9/issues/30)
- [ ] T013 [US2] .github/workflows/UpdateFeaturedProjects.yml の空リポジトリ対応を実装: `REPO_COUNT=0` の場合、セクション全体を非表示 — [#31](https://github.com/DMARU9/DMARU9/issues/31)
- [ ] T014 [US2] README.md の Featured Projects セクションの現在の壊れたテーブルを削除（GitHub Actions が次回実行時に正しいテーブルを自動生成するため） — [#32](https://github.com/DMARU9/DMARU9/issues/32)

**Checkpoint**: Featured Projects が正しいテーブル形式で表示され、バッジがクリック可能

---

## Phase 5: User Story 3 - バッジ label/message の正しい区切り処理 (Priority: P2)

**Goal**: リポジトリ名のハイフンが正しくエスケープされ、バッジに正しく表示される

**Independent Test**: shields.io バッジURLを直接ブラウザで開き、リポジトリ名が正しく表示されることを確認

### Implementation for User Story 3

- [ ] T015 [P] [US3] .github/workflows/UpdateFeaturedProjects.yml にリポジトリ名のハイフンエスケープ処理を追加: `-` → `--` 変換ロジックを実装 — [#33](https://github.com/DMARU9/DMARU9/issues/33)
- [ ] T016 [P] [US3] README.md の手動修正済みバッジ（もしあれば）のハイフンエスケープを確認・修正 — [#34](https://github.com/DMARU9/DMARU9/issues/34)

**Checkpoint**: ハイフン入りリポジトリ名のバッジが正しく表示される

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: 全ストーリー完了後の最終検証とドキュメント更新

- [ ] T017 quickstart.md の検証ガイドに従い、全ステップを実行して動作確認 — [#35](https://github.com/DMARU9/DMARU9/issues/35)
- [ ] T018 [P] Agent Context を更新: `.github/copilot-instructions.md` の `<!-- SPECKIT START -->` 間の参照を新しい plan に更新（既に完了済みの場合はスキップ） — [#36](https://github.com/DMARU9/DMARU9/issues/36)
- [ ] T019 全変更を Conventional Commits 形式でコミット（`fix: ...` または `docs: ...`） — [#37](https://github.com/DMARU9/DMARU9/issues/37)
- [ ] T020 README.md を GitHub で表示し、全セクションが正しくレンダリングされることを最終確認 — [#38](https://github.com/DMARU9/DMARU9/issues/38)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: 依存なし - すぐに開始可能
- **Foundational (Phase 2)**: Setup 完了後に開始可能 - 全ユーザーストーリーをブロック
- **User Stories (Phase 3+)**: Foundational 完了後に開始可能
  - Phase 3 (US1) と Phase 4 (US2) は並列実行可能
  - Phase 5 (US3) は Phase 4 の実装と並列可能
- **Polish (Phase 6)**: 全ユーザーストーリー完了後に実行

### User Story Dependencies

- **User Story 1 (P1)**: Foundational 完了後に開始可能。他のストーリーに依存なし
- **User Story 2 (P1)**: Foundational 完了後に開始可能。他のストーリーに依存なし
- **User Story 3 (P2)**: Foundational 完了後に開始可能。US2 の実装と並列可能（同じファイルを変更するが異なる箇所）

### Parallel Opportunities

- T002, T003, T004 (サービス確認) は並列実行可能
- T005, T006, T007, T008 (alt テキスト更新) は並列実行可能
- T009, T010 (jq スクリプト更新) は並列実行可能
- T011, T012, T013, T014 (ワークフロー改修) は順番に実行（同じファイル内）
- T015, T016 (ハイフンエスケープ) は並列実行可能
- T017, T018 (検証・ドキュメント) は並列実行可能

---

## Parallel Example: User Story 1 + User Story 2

```
Timeline:
  T002 ─┐
  T003 ─┤ (並列: サービス確認)
  T004 ─┘
       ↓
  ┌─── US1 ────┐  ┌─── US2 ────────────────────────┐
  │ T005 ─────┐│  │ T009 ─────────┐                 │
  │ T006 ────┐││  │ T010 ────────┐│                 │
  │ T007 ──┐ │││  │ T011 ──────┐ ││                 │
  │ T008 ─┐│ │││  │ T012 ────┐ │ ││                 │
  │        ││ │││  │ T013 ──┐ │ │ ││                 │
  │        ││ │││  │ T014 ─┐│ │ │ ││                 │
  └────────┘│ │││  └───────┘│ │ │ ││                 │
           └──┘││           └──┘ │ ││                 │
              └─┘               └──┘ │                 │
                                     └─────────────────┘
                                          ↓
                                    ┌─── Polish ───┐
                                    │ T017 ────┐   │
                                    │ T018 ──┐ │   │
                                    │ T019 ─┐│ │   │
                                    │ T020 ┐││ │   │
                                    └──────┘┘┘ ┘   │
                                           └───────┘
```
