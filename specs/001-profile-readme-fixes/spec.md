# Feature Specification: Profile README Fixes & Improvements

**Feature Branch**: `001-profile-readme-fixes`

**Created**: 2026-07-31

**Status**: Ready

**Input**: User description: "以下の仕様作成をお願いします。・about meはこの記載方法で正しいのでしょうか？コピーできるように記載している方法で正しいのですか？そうであれば修正は不要ですが、そうでないのであれば、修正してください。・GitHub Analyticsについてですが、starsとTop Languagesが正しく出ていないです。starsは0であるため出力されていないのかと思います。0にも対応しましょう。Top Languagesはなぜか分かりません。・ Achievementsについても正しく修正してください。・あとはパブリックプロジェクトが増えた場合にそれに対応するようにしてください。"

## User Scenarios & Testing _(mandatory)_

### User Story 1 - About Me Format Improvement (Priority: P1)

Improve the current Kubernetes YAML-style About Me format for better copy usability while keeping the unique visual style. Remove unnecessary metadata fields (`api_version`, `kind`, `namespace`, `labels`) and keep only core profile information (`name`, `stacks`, `status`). This makes the section cleaner and the content easier to copy without irrelevant YAML scaffolding.

**Why this priority**: The About Me section is the first thing visitors see. The Kubernetes YAML style is a distinctive design choice that should be preserved, but the extra metadata fields make it cluttered and hard to copy useful information.

**Independent Test**: Can be fully tested by viewing the README on GitHub and verifying the About Me section renders correctly, is easy to read, and allows clean text copying without unnecessary YAML metadata.

**Acceptance Scenarios**:

1. **Given** the README is viewed on GitHub, **When** the About Me section is displayed, **Then** it should retain the YAML-in-code-block visual style
2. **Given** a user wants to copy information from the About Me section, **When** they select and copy text, **Then** the copied content should contain only useful profile information without metadata like api_version, kind, namespace, labels
3. **Given** the About Me format is evaluated, **When** compared to the original, **Then** it should have fewer unnecessary fields while preserving the core information (name, stacks, status)

---

### User Story 2 - GitHub Analytics Stars Display (Priority: P1)

Ensure the GitHub Analytics stars badge properly displays even when the star count is 0, providing accurate representation of the user's GitHub profile metrics.

**Why this priority**: Stars are a key metric for GitHub profiles and should always be displayed accurately, even when the count is 0. Currently, the badge may be hidden or show incorrect information when stars are 0.

**Independent Test**: Can be fully tested by viewing the README and verifying the stars badge shows "0" or "No stars yet" instead of being hidden or showing broken images.

**Acceptance Scenarios**:

1. **Given** the user has 0 stars, **When** the GitHub Analytics section is displayed, **Then** the stars badge should show "0" or "No stars yet"
2. **Given** the user has stars, **When** the GitHub Analytics section is displayed, **Then** the stars badge should show the correct count
3. **Given** the stars API is unavailable, **When** the GitHub Analytics section is displayed, **Then** a fallback message should be shown instead of broken images

---

### User Story 3 - GitHub Analytics Top Languages Fix (Priority: P1)

Fix the Top Languages display in GitHub Analytics to properly show the user's programming language distribution based on their public repositories.

**Why this priority**: Top Languages provides insight into the user's technical expertise and should accurately reflect their coding activity. Currently, it's not displaying correctly.

**Independent Test**: Can be fully tested by viewing the README and verifying the Top Languages section shows accurate language distribution.

**Acceptance Scenarios**:

1. **Given** the user has public repositories with language data, **When** the Top Languages section is displayed, **Then** it should show the correct language distribution
2. **Given** the user has no public repositories with language data, **When** the Top Languages section is displayed, **Then** an appropriate fallback message should be shown
3. **Given** the Top Languages API is unavailable, **When** the GitHub Analytics section is displayed, **Then** a fallback message should be shown instead of broken images

---

### User Story 4 - GitHub Achievements Display (Priority: P2)

Display actual GitHub Achievements using the GitHub GraphQL API to fetch achievement data, then render them as shields.io badges. This replaces the current manually-created shields.io badges that are not linked to real achievements.

**Why this priority**: Achievements provide social proof and recognition of the user's contributions to the GitHub community. Currently, it uses manually crafted shields.io badges that don't reflect actual GitHub achievements.

**Independent Test**: Can be fully tested by running the GitHub Action and verifying the Achievements section displays actual GitHub achievements as badges, or shows a fallback if the user has no achievements.

**Acceptance Scenarios**:

1. **Given** the user has GitHub achievements, **When** the Achievements section is displayed, **Then** it should show the actual achievements fetched via GraphQL API as shields.io badges
2. **Given** the user has no GitHub achievements, **When** the Achievements section is displayed, **Then** an appropriate "No achievements yet" message should be shown
3. **Given** the GraphQL API is unavailable, **When** the Achievements section is displayed, **Then** a fallback message should be shown instead of broken badges

---

### User Story 5 - Dynamic Public Projects Display (Priority: P2)

Automatically display public projects in the Featured Projects section sorted by stars count (descending), with a maximum of 6 repositories. When all repositories have the same star count, fall back to most recently updated first. The display should update via GitHub Actions when new repositories are created.

**Why this priority**: Featured Projects showcase the user's work and should automatically reflect their current public repository portfolio. Currently, it shows "Coming Soon" placeholders.

**Independent Test**: Can be fully tested by viewing the README and verifying the Featured Projects section shows the correct repos sorted by stars, and verifies the fallback to update-date sorting when stars are equal.

**Acceptance Scenarios**:

1. **Given** the user has public repositories with different star counts, **When** the Featured Projects section is displayed, **Then** repositories should be sorted by stars descending
2. **Given** all public repositories have the same star count, **When** the Featured Projects section is displayed, **Then** repositories should be sorted by last updated date (most recent first)
3. **Given** the user creates a new public repository, **When** the Featured Projects section is updated (via scheduled GitHub Action), **Then** the new repository should appear if it qualifies

---

### Edge Cases

- What happens when GitHub API services are temporarily unavailable?
- How does the system handle repositories with no language data?
- What happens when the user has more than the maximum displayable number of projects?
- How does the system handle special characters in repository names or descriptions?
- What happens when a repository is archived or has special status?

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: System MUST verify and correct the About Me format to follow GitHub profile README best practices
- **FR-002**: System MUST ensure the stars badge displays accurately even when the count is 0
- **FR-003**: System MUST properly display Top Languages based on the user's public repository language data
- **FR-004**: System MUST display GitHub Achievements by fetching data via GitHub GraphQL API and rendering as shields.io badges
- **FR-005**: System MUST automatically update the Featured Projects section when new public repositories are created
- **FR-006**: System MUST provide fallback messages when API services are unavailable
- **FR-007**: System MUST handle edge cases like repositories with no language data gracefully
- **FR-008**: System MUST maintain consistent styling and formatting across all sections

### Non-Functional Requirements

- **NFR-001**: Profile README sections MUST load within 3 seconds under normal network conditions
- **NFR-002**: Fallback messages MUST be clear and informative when services are unavailable
- **NFR-003**: All displayed data MUST be accurate and up-to-date within 24 hours
- **NFR-004**: The README MUST be responsive and display correctly on both desktop and mobile devices

## Clarifications

### Session 2026-07-31

- Q: About Me のフォーマットはどうする？ → A: 現在の Kubernetes YAML スタイルを維持しつつ、コピーユーザビリティを改善する
- Q: Featured Projects の表示方式は？ → A: stars 数順に上位 N 件を表示。全リポジトリの stars が同じ場合は更新日順で直近のものを表示
- Q: Featured Projects の最大表示件数は？ → A: 6 件（GitHub pinned と同等）
- Q: GitHub Achievements の表示方法は？ → A: GitHub GraphQL API で Achievements を取得し、shields.io バッジとして表示する
- Q: About Me の改善内容は？ → A: api_version、kind、namespace、labels を削除し、コア情報（name、stacks、status）のみ残す

## Assumptions

- The user has a GitHub account with public repositories
- GitHub API services are generally available and reliable
- The user wants an automated solution that requires minimal manual updates
- The current README structure should be preserved where possible
- The user prefers visual consistency with the existing color scheme and styling
- The user wants to maintain the terminal/hacker aesthetic of the profile

## Out of Scope

- Private repository information display
- Real-time updates (hourly or more frequent)
- Custom styling beyond the existing color scheme
- Integration with non-GitHub services
- Advanced analytics or metrics beyond standard GitHub profile data
- Changes to the contribution snake or activity sections

## Dependencies

- GitHub API for repository and profile data
- Third-party services for README stats and visualizations (github-readme-stats, github-profile-summary-cards)
- GitHub Actions for automated updates
- Network connectivity for API calls and image loading

## Success Criteria

- All sections of the GitHub profile README display correctly and accurately
- The About Me section follows standard formatting conventions and is easy to read/copy
- GitHub Analytics shows accurate stars (including 0) and correct Top Languages
- Achievements section properly displays GitHub achievements or appropriate alternatives
- Featured Projects automatically updates when new public repositories are created
- Fallback messages appear when services are unavailable
- The README maintains consistent styling and professional appearance
- The profile remains visually appealing and informative
