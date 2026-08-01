# Feature Specification: Profile README Fixes v2 — GitHub Analytics & Featured Projects

**Feature Branch**: `002-profile-readme-fixes-v2`

**Created**: 2026-08-01

**Status**: Draft

**Input**: User description: "GitHub Analyticsのstarsとtop languageの表示がおかしい。Featured Projectsはテーブル表示したいがそのような表示になっていない。Featured Projectsに404 page not foundが4つ出ている。"

## User Scenarios & Testing _(mandatory)_

### User Story 1 - GitHub Analytics 統計画像の正常表示 (Priority: P1)

GitHub Analytics セクションの Stats（スター数やコミット数などの統計）と Top Languages（使用言語の分布）の画像が、外部サービス `github-readme-stats.vercel.app` の 503 停止により表示されなくなっている。代替の稼働中サービスに差し替え、プロフィール訪問者が常に正しい統計情報を確認できるようにする。

**Why this priority**: GitHub Analytics はプロフィールの中心的な情報セクションであり、統計画像が表示されないとプロフィール全体の信頼性が損なわれる。訪問者が最初に目にする重要エリアのため最優先。

**Independent Test**: GitHub プロフィールページを表示し、Stats, Top Languages, Streak, Profile Summary の全画像が正常にレンダリングされることを目視確認する。

**Acceptance Scenarios**:

1. **Given** プロフィールページを開いたとき、**When** GitHub Analytics セクションを表示すると、**Then** Stats カードにスター数・コミット数・PR数・Issue数が正しく表示される
2. **Given** プロフィールページを開いたとき、**When** GitHub Analytics セクションを表示すると、**Then** Top Languages カードに使用言語の分布が正しく表示される
3. **Given** プロフィールページを開いたとき、**When** GitHub Analytics セクションを表示すると、**Then** Streak カードに連続コントリビューション日数が表示される
4. **Given** プロフィールページを開いたとき、**When** Profile Summary セクションを表示すると、**Then** リポジトリ別言語・最多コミット言語・生産的 time 帯のサマリーが表示される
5. **Given** 外部サービスが一時的にダウンした場合、**When** 画像が取得できないとき、**Then** 壊れた画像アイコンではなく、alt テキストによるフォールバック表示がされる

---

### User Story 2 - Featured Projects テーブル表示の修正 (Priority: P1)

Featured Projects セクションが Markdown テーブルとして正しくレンダリングされず、プレーンテキストのまま表示されている。カラム数を統一し、プロジェクト一覧が整形されたテーブルとして表示されるようにする。また、各プロジェクトバッジをクリック可能にして該当リポジトリに遷移できるようにする。

**Why this priority**: Featured Projects はユーザーの代表的な成果物を紹介するセクションであり、テーブルとして正しく整形されていないと訪問者にプロジェクトの情報が伝わらない。バッジがクリックできないのもUX上の問題。

**Independent Test**: GitHub プロフィールページを表示し、Featured Projects セクションが正しいテーブル形式で表示され、各プロジェクトバッジをクリックすると該当リポジトリに遷移することを確認する。

**Acceptance Scenarios**:

1. **Given** プロフィールページを開いたとき、**When** Featured Projects セクションを表示すると、**Then** プロジェクト一覧がテーブル形式で表示される（プレーンテキストではない）
2. **Given** Featured Projects テーブルが表示されているとき、**When** プロジェクトバッジをクリックすると、**Then** 該当する GitHub リポジトリページに遷移する
3. **Given** プロジェクトのスター数が 0 のとき、**When** テーブルを表示すると、**Then** 「⭐ 0」と表示される（非表示にならない）
4. **Given** プロジェクト数が増減したとき、**When** GitHub Actions によって README が更新されると、**Then** テーブルの行数が自動的に調整される（最大4件）

---

### User Story 3 - バッジ label/message の正しい区切り処理 (Priority: P2)

shields.io のバッジURLでは、`-`（ハイフン）が label / message / color の区切り文字として解釈される。リポジトリ名にハイフンを含む場合（例: `OBSIDIAN-KNOWLEDGE-COMPILER`）、`--`（ダブルハイフン）でエスケープする必要がある。現在の実装ではエスケープされておらず、意図しない区切りが発生しているため修正する。

**Why this priority**: バッジの表示崩れは見た目上の問題であり、機能的にはテーブル表示修正（P1）ほど緊急ではないが、正しいプロジェクト名が表示されないのは訪問者に誤解を与える。

**Independent Test**: shields.io バッジURLを直接ブラウザで開き、正しいリポジトリ名が label として表示されることを確認する。

**Acceptance Scenarios**:

1. **Given** リポジトリ名にハイフンが含まれる場合（例: `OBSIDIAN-KNOWLEDGE-COMPILER`）、**When** バッジが生成されると、**Then** label は `OBSIDIAN-KNOWLEDGE-COMPILER` 全体が表示される（`OBSIDIAN: KNOWLEDGE` と分割されない）
2. **Given** リポジトリ名にハイフンが含まれない場合、**When** バッジが生成されると、**Then** 既存の表示が維持される（デグレードしない）

---

### Edge Cases

- 外部統計サービス（`github-readme-streak-stats` や `github-profile-summary-cards` も含む）が停止する可能性がある場合 → フォールバック用サービスは用意せず、alt テキストで表示を維持する
- リポジトリ数が 0 件の場合 → セクション全体を非表示にする
- リポジトリ名や説明に特殊文字（絵文字、HTML タグなど）が含まれる場合 → 説明文はバッジに含めないため問題なし。リポジトリ名は `--` でエスケープして対応
- プロジェクト数が最大表示数（4件）を超えた場合 → 上位4件のみ表示（ソート順で自動的に絞り込まれる）
- GitHub の Camo プロキシが画像をキャッシュしている場合 → サービス復旧後は数時間で自動的にキャッシュが更新されるため、特別な対応は不要

## Requirements _(mandatory)_

### Functional Requirements

- **FR-001**: GitHub Analytics の Stats カードは、稼働中の統計サービスから取得した画像を使用しなければならない（`github-readme-stats.vercel.app` は使用不可）
- **FR-002**: GitHub Analytics の Top Languages カードは、稼働中の統計サービスから取得した画像を使用しなければならない
- **FR-003**: 使用する統計サービスは少なくともスター数・コミット数・使用言語分布を正確に表示できるものでなければならない
- **FR-004**: Featured Projects セクションは、カラム数が統一された Markdown テーブルとしてレンダリングされなければならない（プロジェクト名バッジ + スター数の2カラム構成。説明文は含めない）
  - **FR-004a**: Featured Projects のリポジトリはスター数降順でソートし、スター数が同じ場合は更新日降順でソートしなければならない
  - **FR-004b**: Featured Projects に表示するリポジトリは最大 4件とする
  - **FR-004c**: パブリックリポジトリが0件の場合、Featured Projects セクション全体（見出し・テーブル・改行）を非表示にする
- **FR-005**: 各プロジェクトバッジはクリック可能で、該当 GitHub リポジトリに遷移しなければならない
- **FR-006**: shields.io バッジURL内のリポジトリ名にハイフンが含まれる場合、`--` でエスケープしなければならない（説明文はバッジに含めないため、説明文のエスケープは不要）
- **FR-007**: shields.io バッジURLに `link=` パラメータを含めてはならない（GitHub Markdown では無効なため）
- **FR-008**: バッジのクリックリンクは Markdown の `[![alt](image-url)](target-url)` 記法で実装しなければならない
- **FR-009**: 全セクションの画像には適切な alt テキストを設定し、画像読み込み失敗時に内容が推測可能でなければならない
- **FR-010**: 既存の正常に動作しているセクション（About Me, Tech Stack, Current Focus など）は変更してはならない（デグレード防止）

### Key Entities

- **GitHub Stats カード**: ユーザーの累積GitHub統計（スター、コミット、PR、Issueなど）を可視化する画像
- **Top Languages カード**: ユーザーのパブリックリポジトリにおけるプログラミング言語の使用比率を可視化する画像
- **Featured Project バッジ**: リポジトリ名のみを表示する shields.io バッジ画像（説明文は含めない）。クリックでリポジトリに遷移する
- **Markdown テーブル行**: 1プロジェクトを1行で表現する。各カラムにはプロジェクト名バッジ（クリック可能リンク付き）とスター数（`⭐ N`）を配置する。全行でカラム数（2列）が一致する必要がある

## Success Criteria _(mandatory)_

### Measurable Outcomes

- **SC-001**: GitHub プロフィールページを開いたとき、GitHub Analytics セクションの全画像（Stats, Top Languages, Streak, Profile Summary）が正常に表示される（プレースホルダーや壊れた画像アイコンが存在しない）
- **SC-002**: Featured Projects セクションが正しい Markdown テーブルとしてレンダリングされる（プレーンテキスト表示が 0 件になる）
- **SC-003**: 全プロジェクトバッジをクリックした際、正しいリポジトリページに遷移する（遷移成功率 100%）
- **SC-004**: ハイフンを含むリポジトリ名のバッジが正しい区切りで表示される（表示崩れが 0 件になる）
- **SC-005**: 画像読み込み失敗時、alt テキストにより訪問者が情報を推測できる（`![Stats](url)` のような空の alt が存在しない）

## Clarifications

### Session 2026-08-01

- Q: `github-readme-stats` の代替サービスはどの方法で復旧しますか？ → A: 別の公開インスタンス（フォーク）に差し替え
- Q: Featured Projects テーブルのカラム構成は？ → A: ~~プロジェクト名（バッジ）+ 説明文 + スター数の 3カラム構成~~ → Q8 により上書き: リポジトリ名バッジ + スター数の 2カラム構成（説明文は含めない）
- Q: Featured Projects テーブルの行構成は？ → A: 1プロジェクト = 1行
- Q: 統計サービスが再び停止した場合のフォールバックは？ → A: alt テキストのみ表示（フォールバック用サービスなし）
- Q: Featured Projects のソート順は？ → A: スター数降順（同数なら更新日降順）
- Q: Featured Projects の最大表示数は？ → A: 最大 4件
- Q: パブリックリポジトリが0件の場合の表示は？ → A: セクション全体を非表示
- Q: リポジトリ名・説明の特殊文字処理は？ → A: 説明文はバッジに含めず、リポジトリ名バッジ + スター数のみ

---

## Assumptions

- `github-readme-streak-stats.herokuapp.com` および `github-profile-summary-cards.vercel.app` は現在稼働中であり、当面利用可能である
- `github-readme-stats` の代替として、同じく広く使われている `github-readme-stats` の公開フォークインスタンスに差し替える（自前デプロイではない）
- Featured Projects セクションは GitHub Actions によって自動生成される前提で、手動更新は不要
- shields.io のバッジはリポジトリ名のみを label として使用し、説明文は含めない（特殊文字問題の回避）
- shields.io の `link=` パラメータは SVG 内部の `<a>` タグに作用するが、GitHub の `<img>` タグによるレンダリングでは無効であるため、Markdown リンク記法で代替する
- テーブルのカラム数は2列（プロジェクト名バッジ・スター数）に固定し、1プロジェクト = 1行で表示する。最大表示数は4件
- ハイフンを含むリポジトリ名のエスケープ処理は GitHub Actions のワークフロー内で実施する
