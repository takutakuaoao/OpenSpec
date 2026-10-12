<a id="concepts"></a>
# 概念

このガイドでは、OpenSpec の中核となる考え方と、それらがどのように組み合わさるかを説明します。実際の使い方については、[はじめに](getting-started.md) と [ワークフロー](workflows.md) を参照してください。

<a id="philosophy"></a>
## 思想

OpenSpec は、4つの原則を軸に作られています。

```
fluid not rigid         — フェーズゲートなし。意味のあるものから取り組む
iterative not waterfall — 作りながら学び、進めながら洗練させる
easy not complex        — 軽量なセットアップ、最小限の儀式
brownfield-first        — グリーンフィールドだけでなく、既存のコードベースで機能する
```

<a id="why-these-principles-matter"></a>
### これらの原則が重要な理由

**柔軟で、硬直していない（Fluid not rigid）。** 従来の仕様システムは、最初に計画し、次に実装し、それで終わり、というフェーズにあなたを縛りつけます。OpenSpec はより柔軟です。作業にとって意味のある任意の順序で、成果物を作成できます。

**反復的で、ウォーターフォールではない（Iterative not waterfall）。** 要件は変わります。理解は深まります。最初は良さそうに見えたアプローチも、コードベースを見たあとでは通用しないかもしれません。OpenSpec はこの現実を受け入れます。

**簡単で、複雑ではない（Easy not complex）。** 仕様フレームワークの中には、大がかりなセットアップ、硬直した形式、重いプロセスを必要とするものがあります。OpenSpec は邪魔をしません。数秒で初期化でき、すぐに作業を始められ、必要な場合にだけカスタマイズします。

**ブラウンフィールド第一（Brownfield-first）。** ソフトウェアの作業の大半は、ゼロから作ることではなく、既存システムの変更です。OpenSpec のデルタベースのアプローチにより、新しいシステムを記述するだけでなく、既存の振る舞いに対する変更を簡単に仕様化できます。

<a id="the-big-picture"></a>
## 全体像

OpenSpec は、作業を2つの主要な領域に整理します。

```
┌────────────────────────────────────────────────────────────────────┐
│                        openspec/                                   │
│                                                                    │
│   ┌─────────────────────┐      ┌───────────────────────────────┐   │
│   │       specs/        │      │         changes/              │   │
│   │                     │      │                               │   │
│   │  信頼できる          │◄─────│  提案された変更                │   │
│   │  唯一の情報源        │ merge│  1つの変更 = 1つのフォルダ      │   │
│   │  システムの現在の    │      │  成果物とデルタを含む           │   │
│   │  仕組み             │      │                               │   │
│   └─────────────────────┘      └───────────────────────────────┘   │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**仕様（Specs）**は信頼できる唯一の情報源であり、システムが現在どう振る舞うかを記述します。

**変更（Changes）**は提案された修正であり、マージの準備ができるまで別々のフォルダに置かれます。

この分離が鍵です。複数の変更を、衝突することなく並行して進められます。メインの仕様に影響が及ぶ前に、変更をレビューできます。そして変更をアーカイブすると、そのデルタが信頼できる唯一の情報源にきれいにマージされます。

<a id="specs"></a>
## 仕様（Specs）

仕様は、構造化された要件とシナリオを使って、システムの振る舞いを記述します。

<a id="structure"></a>
### 構造

```
openspec/specs/
├── auth/
│   └── spec.md           # 認証の振る舞い
├── payments/
│   └── spec.md           # 決済処理
├── notifications/
│   └── spec.md           # 通知システム
└── ui/
    └── spec.md           # UI の振る舞いとテーマ
```

仕様は、システムにとって意味のある論理的なグルーピングである、ドメインごとに整理します。一般的なパターン:

- **機能領域ごと**: `auth/`、`payments/`、`search/`
- **コンポーネントごと**: `api/`、`frontend/`、`workers/`
- **境界づけられたコンテキストごと**: `ordering/`、`fulfillment/`、`inventory/`

<a id="spec-format"></a>
### 仕様のフォーマット

仕様には要件が含まれ、各要件にはシナリオがあります。

```markdown
# Auth Specification

## Purpose
Authentication and session management for the application.

## Requirements

### Requirement: User Authentication
The system SHALL issue a JWT token upon successful login.

#### Scenario: Valid credentials
- GIVEN a user with valid credentials
- WHEN the user submits login form
- THEN a JWT token is returned
- AND the user is redirected to dashboard

#### Scenario: Invalid credentials
- GIVEN invalid credentials
- WHEN the user submits login form
- THEN an error message is displayed
- AND no token is issued

### Requirement: Session Expiration
The system MUST expire sessions after 30 minutes of inactivity.

#### Scenario: Idle timeout
- GIVEN an authenticated session
- WHEN 30 minutes pass without activity
- THEN the session is invalidated
- AND the user must re-authenticate
```

**主な要素:**

| 要素 | 目的 |
|---------|---------|
| `## Purpose` | この仕様のドメインの概要説明 |
| `### Requirement:` | システムが持たなければならない特定の振る舞い |
| `#### Scenario:` | 要件が実際に機能する様子の具体例 |
| SHALL/MUST/SHOULD | 要件の強さを示す RFC 2119 のキーワード |

<a id="why-structure-specs-this-way"></a>
### 仕様をこの構造にする理由

**要件は「何を」** — 実装を指定せずに、システムが何をすべきかを述べます。

**シナリオは「いつ」** — 検証可能な具体例を提供します。良いシナリオは次の性質を持ちます。
- テスト可能である（自動テストを書ける）
- 正常系とエッジケースの両方をカバーしている
- Given/When/Then、または同様の構造化された形式を使っている

**RFC 2119 のキーワード**（SHALL、MUST、SHOULD、MAY）は意図を伝えます。
- **MUST/SHALL** — 絶対的な要件
- **SHOULD** — 推奨だが、例外はあり得る
- **MAY** — 任意

<a id="what-a-spec-is-and-is-not"></a>
### 仕様とは何か（そして何でないか）

仕様は**振る舞いの契約**であり、実装計画ではありません。

仕様に適した内容:
- ユーザーや下流のシステムが依存する、観測可能な振る舞い
- 入力、出力、エラー条件
- 外部的な制約（セキュリティ、プライバシー、信頼性、互換性）
- テスト可能、または明示的に検証できるシナリオ

仕様では避けるべきこと:
- 内部のクラス名や関数名
- ライブラリやフレームワークの選択
- 実装の手順レベルの詳細
- 詳細な実行計画（これらは `design.md` や `tasks.md` に属する）

簡易テスト:
- 外部から見える振る舞いを変えずに実装を変えられるなら、それはおそらく仕様に属さない。

<a id="keep-it-lightweight-progressive-rigor"></a>
### 軽量に保つ: 段階的な厳密さ

OpenSpec は、官僚的になることを避けることを目指しています。変更を検証可能にするのに十分な、最も軽いレベルを使ってください。

**ライト仕様（デフォルト）:**
- 短く、振る舞いを優先した要件
- 明確なスコープと非目標
- いくつかの具体的な受け入れチェック

**フル仕様（リスクが高い場合）:**
- チーム間やリポジトリ間にまたがる変更
- API/契約の変更、マイグレーション、セキュリティ/プライバシーの懸念
- 曖昧さが高コストの手戻りを招きやすい変更

ほとんどの変更はライトモードで十分です。

<a id="human--agent-collaboration"></a>
### 人間とエージェントの協働

多くのチームでは、人間が探索し、エージェントが成果物を下書きします。想定するループは次のとおりです。

1. 人間が、意図、コンテキスト、制約を提供する。
2. エージェントがこれを、振る舞いを優先した要件とシナリオに変換する。
3. エージェントは、実装の詳細を `spec.md` ではなく `design.md` と `tasks.md` に置く。
4. 実装の前に、検証が構造と明確さを確認する。

これにより、仕様は人間にとって読みやすく、エージェントにとって一貫したものに保たれます。

<a id="changes"></a>
## 変更（Changes）

変更とは、システムに対する提案された修正であり、理解と実装に必要なすべてをまとめたフォルダとしてパッケージ化されます。

<a id="change-structure"></a>
### 変更の構造

```
openspec/changes/add-dark-mode/
├── proposal.md           # 理由と内容
├── design.md             # 方法（技術的アプローチ）
├── tasks.md              # 実装チェックリスト
├── .openspec.yaml        # 変更のメタデータ（任意）: schema, created, skip_specs, retire_capabilities
└── specs/                # デルタ仕様
    └── ui/
        └── spec.md       # ui/spec.md で何が変わるか
```

各変更は自己完結しています。次のものを含みます。
- **成果物** — 意図、設計、タスクを記録したドキュメント
- **デルタ仕様** — 追加、変更、削除される内容の仕様
- **メタデータ** — この特定の変更のための任意の設定

<a id="why-changes-are-folders"></a>
### 変更がフォルダである理由

変更をフォルダとしてパッケージ化することには、いくつかの利点があります。

1. **すべてが一か所に。** 提案、設計、タスク、仕様が1か所にあります。あちこちを探し回る必要はありません。

2. **並行作業。** 複数の変更が、衝突することなく同時に存在できます。`fix-auth-bug` が進行中でも、`add-dark-mode` に取り組めます。

3. **きれいな履歴。** アーカイブされると、変更は完全な文脈を保ったまま `changes/archive/` に移動されます。何が変わったかだけでなく、なぜ変わったかも後から振り返って理解できます。

4. **レビューしやすい。** 変更フォルダはレビューが容易です。開いて、提案を読み、設計を確認し、仕様のデルタを見ます。

<a id="artifacts"></a>
## 成果物（Artifacts）

成果物は、変更の中にあって作業を導くドキュメントです。

<a id="the-artifact-flow"></a>
### 成果物のフロー

```
proposal ──────► specs ──────► design ──────► tasks ──────► implement
    │               │             │              │
   理由            内容           方法          取るべき
 + スコープ         変更          アプローチ       ステップ
```

成果物は互いの上に積み上がります。各成果物が、次の成果物に文脈を提供します。

<a id="artifact-types"></a>
### 成果物の種類

<a id="proposal-proposalmd"></a>
#### 提案（Proposal, `proposal.md`）

提案は、**意図**、**スコープ**、**アプローチ**を高いレベルで記述します。

```markdown
# Proposal: Add Dark Mode

## Intent
Users have requested a dark mode option to reduce eye strain
during nighttime usage and match system preferences.

## Scope
In scope:
- Theme toggle in settings
- System preference detection
- Persist preference in localStorage

Out of scope:
- Custom color themes (future work)
- Per-page theme overrides

## Approach
Use CSS custom properties for theming with a React context
for state management. Detect system preference on first load,
allow manual override.
```

**提案を更新するタイミング:**
- スコープが変わったとき（縮小または拡大）
- 意図が明確になったとき（問題の理解が深まった）
- アプローチが根本的に変わったとき

<a id="specs-delta-specs-in-specs"></a>
#### 仕様（`specs/` 内のデルタ仕様）

デルタ仕様は、現在の仕様に対して**何が変わるか**を記述します。後述の [デルタ仕様](#delta-specs) を参照してください。

<a id="design-designmd"></a>
#### 設計（Design, `design.md`）

設計は、**技術的アプローチ**と**アーキテクチャ上の判断**を記述します。

````markdown
# Design: Add Dark Mode

## Technical Approach
Theme state managed via React Context to avoid prop drilling.
CSS custom properties enable runtime switching without class toggling.

## Architecture Decisions

### Decision: Context over Redux
Using React Context for theme state because:
- Simple binary state (light/dark)
- No complex state transitions
- Avoids adding Redux dependency

### Decision: CSS Custom Properties
Using CSS variables instead of CSS-in-JS because:
- Works with existing stylesheet
- No runtime overhead
- Browser-native solution

## Data Flow
```
ThemeProvider (context)
       │
       ▼
ThemeToggle ◄──► localStorage
       │
       ▼
CSS Variables (applied to :root)
```

## File Changes
- `src/contexts/ThemeContext.tsx` (new)
- `src/components/ThemeToggle.tsx` (new)
- `src/styles/globals.css` (modified)
````

**設計を更新するタイミング:**
- 実装によって、そのアプローチが通用しないとわかったとき
- より良い解決策が見つかったとき
- 依存関係や制約が変わったとき

<a id="tasks-tasksmd"></a>
#### タスク（Tasks, `tasks.md`）

タスクは**実装チェックリスト**であり、チェックボックス付きの具体的なステップです。

```markdown
# Tasks

## 1. Theme Infrastructure
- [ ] 1.1 Create ThemeContext with light/dark state
- [ ] 1.2 Add CSS custom properties for colors
- [ ] 1.3 Implement localStorage persistence
- [ ] 1.4 Add system preference detection

## 2. UI Components
- [ ] 2.1 Create ThemeToggle component
- [ ] 2.2 Add toggle to settings page
- [ ] 2.3 Update Header to include quick toggle

## 3. Styling
- [ ] 3.1 Define dark theme color palette
- [ ] 3.2 Update components to use CSS variables
- [ ] 3.3 Test contrast ratios for accessibility
```

**タスクのベストプラクティス:**
- 関連するタスクを見出しの下にグループ化する
- 階層的な番号付け（1.1、1.2 など）を使う
- 1回のセッションで完了できる程度にタスクを小さく保つ
- 各タスクをどう検証するか（テスト、コマンド、観測可能な結果）を明記する
- 各グループの作業に必要なテストとドキュメントは、最後にまとめて対応するグループではなく、そのグループ内で行う
- 完了したらタスクにチェックを付ける

<a id="delta-specs"></a>
## デルタ仕様（Delta Specs）

デルタ仕様は、OpenSpec をブラウンフィールドの開発で機能させる重要な概念です。仕様全体を再度記述するのではなく、**何が変わるか**を記述します。

<a id="the-format"></a>
### フォーマット

```markdown
# Delta for Auth

## ADDED Requirements

### Requirement: Two-Factor Authentication
The system MUST support TOTP-based two-factor authentication.

#### Scenario: 2FA enrollment
- GIVEN a user without 2FA enabled
- WHEN the user enables 2FA in settings
- THEN a QR code is displayed for authenticator app setup
- AND the user must verify with a code before activation

#### Scenario: 2FA login
- GIVEN a user with 2FA enabled
- WHEN the user submits valid credentials
- THEN an OTP challenge is presented
- AND login completes only after valid OTP

## MODIFIED Requirements

### Requirement: Session Expiration
The system MUST expire sessions after 15 minutes of inactivity.
(Previously: 30 minutes)

#### Scenario: Idle timeout
- GIVEN an authenticated session
- WHEN 15 minutes pass without activity
- THEN the session is invalidated

## REMOVED Requirements

### Requirement: Remember Me
(Deprecated in favor of 2FA. Users should re-authenticate each session.)
```

<a id="delta-sections"></a>
### デルタのセクション

| セクション | 意味 | アーカイブ時の動作 |
|---------|---------|------------------------|
| `## ADDED Requirements` | 新しい振る舞い | メインの仕様に追加される |
| `## MODIFIED Requirements` | 変更された振る舞い | 既存の要件を置き換える |
| `## REMOVED Requirements` | 廃止される振る舞い | メインの仕様から削除される。変更が `retire_capabilities: true` を宣言している場合、最後の要件を削除するとその機能は廃止され、仕様ファイルも削除される |
| `## Purpose` | まったく新しい機能が何のためのものか | 作成されるメイン仕様の Purpose の初期値になる。仕様がすでに存在する場合は無視される |

<a id="why-deltas-instead-of-full-specs"></a>
### 完全な仕様ではなくデルタを使う理由

**明確さ。** デルタは、何が変わるのかを正確に示します。完全な仕様を読む場合は、現在のバージョンとの差分を頭の中で比較しなければなりません。

**競合の回避。** 2つの変更が、異なる要件を変更している限り、競合することなく同じ仕様ファイルに触れることができます。

**レビューの効率。** レビュアーは、変更されていない文脈ではなく、変更そのものを見ます。重要な部分に集中できます。

**ブラウンフィールドへの適合。** ほとんどの作業は既存の振る舞いを変更するものです。デルタは、変更を後付けではなく、第一級のものとして扱います。

<a id="schemas"></a>
## スキーマ（Schemas）

スキーマは、ワークフローの成果物の種類と、それらの依存関係を定義します。

<a id="how-schemas-work"></a>
### スキーマの仕組み

```yaml
# openspec/schemas/spec-driven/schema.yaml
name: spec-driven
artifacts:
  - id: proposal
    generates: proposal.md
    requires: []              # No dependencies, can create first

  - id: specs
    generates: specs/**/*.md
    requires: [proposal]      # Needs proposal before creating

  - id: design
    generates: design.md
    requires: [proposal]      # Can create in parallel with specs

  - id: tasks
    generates: tasks.md
    requires: [specs, design] # Needs both specs and design first
```

**成果物は依存関係グラフを形成します:**

```
                    proposal
                   (root node)
                       │
         ┌─────────────┴─────────────┐
         │                           │
         ▼                           ▼
      specs                       design
   (requires:                  (requires:
    proposal)                   proposal)
         │                           │
         └─────────────┬─────────────┘
                       │
                       ▼
                    tasks
                (requires:
                specs, design)
```

**依存関係は手段であり、関門ではありません。** 何を作成できるかを示すものであり、次に何を作成しなければならないかを示すものではありません。不要であれば設計を省略できます。仕様は設計の前でも後でも作成できます。どちらも proposal にのみ依存しているからです。

<a id="built-in-schemas"></a>
### 組み込みスキーマ

**spec-driven**（デフォルト）

仕様駆動開発の標準ワークフロー:

```
proposal → specs → design → tasks → implement
```

最適な用途: 実装の前に仕様について合意したい、ほとんどの機能開発。

<a id="custom-schemas"></a>
### カスタムスキーマ

チームのワークフロー向けにカスタムスキーマを作成します。

```bash
# Create from scratch
openspec schema init research-first

# Or fork an existing one
openspec schema fork spec-driven research-first
```

**カスタムスキーマの例:**

```yaml
# openspec/schemas/research-first/schema.yaml
name: research-first
artifacts:
  - id: research
    generates: research.md
    requires: []           # Do research first

  - id: proposal
    generates: proposal.md
    requires: [research]   # Proposal informed by research

  - id: tasks
    generates: tasks.md
    requires: [proposal]   # Skip specs/design, go straight to tasks
```

カスタムスキーマの作成と使用方法の詳細は、[カスタマイズ](customization.md) を参照してください。

<a id="archive"></a>
## アーカイブ（Archive）

アーカイブは、変更のデルタ仕様をメインの仕様にマージし、履歴のために変更を保存することで、変更を完了します。

<a id="what-happens-when-you-archive"></a>
### アーカイブ時に起こること

```
アーカイブ前:

openspec/
├── specs/
│   └── auth/
│       └── spec.md ◄────────────────┐
└── changes/                         │
    └── add-2fa/                     │
        ├── proposal.md              │
        ├── design.md                │ merge
        ├── tasks.md                 │
        └── specs/                   │
            └── auth/                │
                └── spec.md ─────────┘


アーカイブ後:

openspec/
├── specs/
│   └── auth/
│       └── spec.md        # 2FA の要件が含まれるようになった
└── changes/
    └── archive/
        └── 2025-01-24-add-2fa/    # 履歴のために保存
            ├── proposal.md
            ├── design.md
            ├── tasks.md
            └── specs/
                └── auth/
                    └── spec.md
```

<a id="the-archive-process"></a>
### アーカイブのプロセス

1. **デルタをマージする。** 各デルタ仕様のセクション（ADDED/MODIFIED/REMOVED）が、対応するメインの仕様に適用されます。

2. **アーカイブに移動する。** 変更フォルダは、時系列順に並ぶよう日付のプレフィックスが付けられて `changes/archive/` に移動されます。

3. **文脈を保存する。** すべての成果物がアーカイブ内にそのまま残ります。変更がなぜ行われたのかを、いつでも振り返って理解できます。

<a id="why-archive-matters"></a>
### アーカイブが重要な理由

**クリーンな状態。** アクティブな変更（`changes/`）には、進行中の作業だけが表示されます。完了した作業は邪魔にならない場所に移動されます。

**監査証跡。** アーカイブは、すべての変更の完全な文脈を保存します。何が変わったかだけでなく、理由を説明する提案、方法を説明する設計、行われた作業を示すタスクも含まれます。

**仕様の進化。** 変更がアーカイブされるにつれて、仕様は有機的に成長します。アーカイブのたびにデルタがマージされ、時間とともに包括的な仕様が築かれます。

<a id="how-it-all-fits-together"></a>
## 全体の組み合わせ

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                              OPENSPEC FLOW                                   │
│                                                                              │
│   ┌────────────────┐                                                         │
│   │  1. START      │  /opsx:propose (core) or /opsx:new (expanded)           │
│   │     CHANGE     │                                                         │
│   └───────┬────────┘                                                         │
│           │                                                                  │
│           ▼                                                                  │
│   ┌────────────────┐                                                         │
│   │  2. CREATE     │  /opsx:ff or /opsx:continue (expanded workflow)         │
│   │     ARTIFACTS  │  Creates proposal → specs → design → tasks              │
│   │                │  (based on schema dependencies)                         │
│   └───────┬────────┘                                                         │
│           │                                                                  │
│           ▼                                                                  │
│   ┌────────────────┐                                                         │
│   │  3. IMPLEMENT  │  /opsx:apply                                            │
│   │     TASKS      │  Work through tasks, checking them off                  │
│   │                │◄──── Update artifacts as you learn                      │
│   └───────┬────────┘                                                         │
│           │                                                                  │
│           ▼                                                                  │
│   ┌────────────────┐                                                         │
│   │  4. VERIFY     │  /opsx:verify (optional)                                │
│   │     WORK       │  Check implementation matches specs                     │
│   └───────┬────────┘                                                         │
│           │                                                                  │
│           ▼                                                                  │
│   ┌────────────────┐     ┌──────────────────────────────────────────────┐    │
│   │  5. ARCHIVE    │────►│  Delta specs merge into main specs           │    │
│   │     CHANGE     │     │  Change folder moves to archive/             │    │
│   └────────────────┘     │  Specs are now the updated source of truth   │    │
│                          └──────────────────────────────────────────────┘    │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

**好循環:**

1. 仕様が現在の振る舞いを記述する
2. 変更が（デルタとして）修正を提案する
3. 実装が変更を現実のものにする
4. アーカイブがデルタを仕様にマージする
5. 仕様が新しい振る舞いを記述するようになる
6. 次の変更が、更新された仕様の上に築かれる

<a id="glossary"></a>
## 用語集

| 用語 | 定義 |
|------|------------|
| **成果物（Artifact）** | 変更内のドキュメント（proposal、design、tasks、またはデルタ仕様） |
| **アーカイブ（Archive）** | 変更を完了し、そのデルタをメインの仕様にマージするプロセス |
| **変更（Change）** | システムに対する提案された修正で、成果物を含むフォルダとしてパッケージ化されたもの |
| **デルタ仕様（Delta spec）** | 現在の仕様に対する変更（ADDED/MODIFIED/REMOVED）を記述する仕様 |
| **ドメイン（Domain）** | 仕様の論理的なグルーピング（例: `auth/`、`payments/`） |
| **要件（Requirement）** | システムが持たなければならない特定の振る舞い |
| **シナリオ（Scenario）** | 要件の具体例で、通常は Given/When/Then 形式 |
| **スキーマ（Schema）** | 成果物の種類とその依存関係の定義 |
| **仕様（Spec）** | システムの振る舞いを記述する仕様書で、要件とシナリオを含む |
| **信頼できる唯一の情報源（Source of truth）** | 現在合意されている振る舞いを含む `openspec/specs/` ディレクトリ |

<a id="next-steps"></a>
## 次のステップ

- [はじめに](getting-started.md) - 実践的な最初のステップ
- [ワークフロー](workflows.md) - よくあるパターンと、それぞれをいつ使うか
- [コマンド](commands.md) - 完全なコマンドリファレンス
- [カスタマイズ](customization.md) - カスタムスキーマの作成とプロジェクトの設定
