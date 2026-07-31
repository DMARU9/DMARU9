# Tasks: Profile README Fixes & Improvements

**Input**: Design documents from `/specs/001-profile-readme-fixes/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: テストタスクは生成しません（仕様書で明示的に要求されていないため）

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: プロジェクトの検証ツールをセットアップし、動的セクションのマーカーを追加

- [ ] T001 [P] Create `.markdownlint.json` at repository root to configure markdownlint rules (disable line length rule MD013, allow inline HTML MD033) per plan.md constraints
- [ ] T002 [P] Add `<!--START_SECTION:achievements-->` and `<!--END_SECTION:achievements-->` section markers to `README.md` in the GitHub Achievements section, with fallback message `> ℹ️ No achievements data available.`
- [ ] T003 [P] Add `<!--START_SECTION:featured_projects-->` and `<!--END_SECTION:featured_projects-->` section markers to `README.md` in the Featured Projects section, replacing the current "Coming Soon" placeholder with fallback message `> 🏗️ No public projects yet. Stay tuned!`

---

## Phase 2: User Story 1 - About Me Format Improvement (Priority: P1) 🎯 MVP

**Goal**: Kubernetes YAML スタイルを維持しつつ、不要フィールドを削除してコピーユーザビリティを改善

**Independent Test**: README.md の About Me セクションをブラウザで確認し、`api_version`, `kind`, `namespace`, `labels` が存在せず、`name`, `stacks`, `status` が残っていることを検証

### Implementation for User Story 1

- [ ] T004 [US1] Remove unnecessary metadata fields (`api_version`, `kind`, `namespace`, `labels`) from the About Me YAML code block in `README.md`, keeping only `name` and `spec` (containing `location`, `stacks`, `status`)

**Checkpoint**: At this point, User Story 1 should be fully functional — About Me section is clean and copy-friendly

---

## Phase 3: User Story 2 - GitHub Analytics Stars Display (Priority: P1)

**Goal**: Stars バッジが 0 の場合でも正しく表示されるよう修正

**Independent Test**: README.md の GitHub Analytics セクションで、Stars バッジが 0 の場合でも `0 Stars` と表示されることを検証

### Implementation for User Story 2

- [ ] T005 [US2] Verify and fix the stars badge in `README.md` GitHub Analytics section. Current badge uses `https://img.shields.io/github/stars/DMARU9?style=for-the-badge&logo=github&label=STARS&color=ff69b4` — confirm this shields.io endpoint displays "0" correctly (shields.io returns a badge even for 0 stars). If it does not, switch to a static badge: `https://img.shields.io/badge/⭐_Stars-0-ff69b4?style=for-the-badge&logo=github`

**Checkpoint**: At this point, User Story 2 should be functional — stars badge displays accurately including 0

---

## Phase 4: User Story 3 - GitHub Analytics Top Languages Fix (Priority: P1)

**Goal**: Top Languages の表示問題を修正し、言語分布を正しく表示

**Independent Test**: README.md の GitHub Analytics セクションで、Top Languages バッジが言語分布を正しく表示することを検証

### Implementation for User Story 3

- [ ] T006 [US3] Fix the Top Languages badge in `README.md` GitHub Analytics section. Current URL uses `github-readme-stats.vercel.app/api/top-langs/`. Troubleshooting steps: (1) Verify the URL loads correctly by opening it directly in a browser, (2) Add `hide=&show=` parameters to control which languages are displayed, (3) Ensure `layout=compact` and theme/color params match the existing color scheme, (4) If the API is consistently broken, consider alternative: `github-readme-stats.vercel.app/api/top-langs?username=DMARU9&layout=compact&hide_border=true&title_color=00C9FF&text_color=c9d1d9&bg_color=0D1117&theme=transparent`. Add a fallback message for when no language data is available

**Checkpoint**: At this point, User Story 3 should be functional — Top Languages displays correctly

---

## Phase 5: User Story 4 - GitHub Achievements Display (Priority: P2)

**Goal**: GitHub GraphQL API で Achievements を取得し、shields.io バッジとして表示

**Independent Test**: `gh workflow run UpdateAchievements.yml` を手動実行し、README.md の Achievements セクションに実際の GitHub Achievements が表示されることを検証

### Implementation for User Story 4

- [ ] T007 [US4] Create `.github/workflows/UpdateAchievements.yml` — new GitHub Actions workflow that: (1) triggers on schedule (every 24 hours) and manual dispatch, (2) uses `GITHUB_TOKEN` for GraphQL API authentication, (3) calls GitHub GraphQL API to fetch `user.achievements` for login `DMARU9`, (4) transforms achievement nodes into shields.io badge URLs using the pattern `https://img.shields.io/badge/{name}-{description}-00C9FF?style=for-the-badge`, (5) updates README.md between `<!--START_SECTION:achievements-->` and `<!--END_SECTION:achievements-->` markers, (6) includes fallback message `> ℹ️ No achievements data available.` when no achievements found, (7) commits and pushes changes with `concurrency` group to prevent parallel runs, (8) requires `contents: write` permission
- [ ] T008 [US4] Update the Achievements section in `README.md` to replace the current manual shields.io badges (`🔰_Joined`, `📂_Public_Repos`, `⚡_Status`) and the github-profile-trophy fallback note with the dynamic section markers and a clean fallback state

**Checkpoint**: At this point, User Story 4 should be functional — Achievements display actual data via GraphQL API

---

## Phase 6: User Story 5 - Dynamic Public Projects Display (Priority: P2)

**Goal**: Featured Projects を stars 順（同数なら更新日順）で最大 6 件自動表示

**Independent Test**: `gh workflow run UpdateFeaturedProjects.yml` を手動実行し、README.md の Featured Projects セクションにリポジトリが stars 順で表示されることを検証

### Implementation for User Story 5

- [ ] T009 [US5] Create `.github/workflows/UpdateFeaturedProjects.yml` — new GitHub Actions workflow that: (1) triggers on schedule (every 6 hours) and manual dispatch, (2) calls GitHub REST API `GET /users/DMARU9/repos` to fetch public repos, (3) filters out forks and archived repos, (4) sorts by `stargazers_count` descending, then by `pushed_at` descending for ties, (5) selects top 6 repos, (6) generates shields.io badges per repo using pattern `https://img.shields.io/badge/{name}-{description}-{color}?style=for-the-badge&link={url}` with colors cycling through `#00C9FF`, `#92FE9D`, `#ff69b4`, (7) generates star count labels below each badge, (8) updates README.md between `<!--START_SECTION:featured_projects-->` and `<!--END_SECTION:featured_projects-->` markers with a table layout, (9) includes fallback message `> 🏗️ No public projects yet. Stay tuned!` when no repos found, (10) commits and pushes changes with `concurrency` group, (11) requires `contents: write` permission
- [ ] T010 [US5] Update the Featured Projects section in `README.md` to replace the current "Coming Soon" placeholder content with the dynamic section markers and a clean fallback state

**Checkpoint**: At this point, User Story 5 should be functional — Featured Projects auto-updates with repos sorted by stars

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: 全体の品質担保と仕様書の検証

- [ ] T011 Run markdownlint on `README.md` to verify all markdown changes are valid. Fix any linting errors
- [ ] T012 Run link check (lychee or equivalent) on `README.md` to verify all badge URLs and links are valid
- [ ] T013 Verify all README.md section markers are correctly placed and matched (`<!--START_SECTION:...-->` / `<!--END_SECTION:...-->`)
- [ ] T014 Run quickstart.md validation scenarios to confirm all 6 test scenarios pass

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — can start immediately
- **US1 About Me (Phase 2)**: No dependencies — can start immediately (parallel with Phase 1)
- **US2 Stars (Phase 3)**: No dependencies — can start immediately (parallel with Phase 1)
- **US3 Top Languages (Phase 4)**: No dependencies — can start immediately (parallel with Phase 1)
- **US4 Achievements (Phase 5)**: Depends on T002 (section markers in README) from Phase 1
- **US5 Featured Projects (Phase 6)**: Depends on T003 (section markers in README) from Phase 1
- **Polish (Phase 7)**: Depends on all user stories being complete

### User Story Dependencies

- **US1 About Me (P1)**: Independent — no dependencies on other stories
- **US2 Stars (P1)**: Independent — no dependencies on other stories
- **US3 Top Languages (P1)**: Independent — no dependencies on other stories
- **US4 Achievements (P2)**: Depends on Phase 1 (section markers)
- **US5 Featured Projects (P2)**: Depends on Phase 1 (section markers)

### Within Each User Story

- README.md edits should be done in sequence (avoid merge conflicts)
- Workflow creation is independent of README edits
- Each story should be committed separately for clean git history

### Parallel Opportunities

- T001, T002, T003 (Phase 1) can all run in parallel
- T004 (US1), T005 (US2), T006 (US3) can all run in parallel (different sections of README.md)
- T007 (US4 workflow) and T009 (US5 workflow) can run in parallel (different files)
- T008 (US4 README) and T010 (US5 README) can run in parallel (different sections)
- All Polish tasks (T011-T014) can run in parallel

---

## Parallel Execution Examples

### User Story 1 (About Me) — Single Task, No Parallelism Needed

```
T004 → [US1 Complete]
```

### User Stories 2 + 3 (Stars + Top Languages) — Independent, Parallelizable

```
T005 ─┐
       ├→ [US2 + US3 Complete]
T006 ─┘
```

### User Stories 4 + 5 (Achievements + Featured Projects) — Workflow + README per story

```
T007 ─┐  T009 ─┐
       ├→       ├→ [US4 + US5 Complete]
T008 ─┘  T010 ─┘
```

### All P1 Stories in Parallel (After Phase 1)

```
T001 ─┐
T002 ─┤
T003 ─┘
       │
       ▼
T004 ─┐
T005 ─┤
T006 ─┘
       │
       ▼
[All P1 Stories Complete — MVP]
```

---

## Implementation Strategy

### MVP Scope (User Stories 1, 2, 3)

The MVP delivers all P1 user stories which are direct README.md edits:

1. **US1**: Clean About Me YAML (T004)
2. **US2**: Fix stars badge (T005)
3. **US3**: Fix Top Languages (T006)

These can be completed in a single commit and provide immediate value.

### Incremental Delivery

1. **Sprint 1 (MVP)**: Phase 1 + Phase 2 + Phase 3 + Phase 4 → README fixes
2. **Sprint 2**: Phase 5 + Phase 6 → GitHub Actions automation
3. **Sprint 3**: Phase 7 → Polish & validation

### Commit Strategy

- Phase 1: `chore: add markdownlint config and section markers`
- Phase 2: `fix: clean up About Me YAML format`
- Phase 3: `fix: ensure stars badge displays 0 correctly`
- Phase 4: `fix: repair Top Languages display`
- Phase 5: `feat: add UpdateAchievements workflow with GraphQL API`
- Phase 6: `feat: add UpdateFeaturedProjects workflow with REST API`
- Phase 7: `chore: validate markdown and links`
