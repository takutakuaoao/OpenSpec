<a id="migrating-to-opsx"></a>
# OPSX への移行

このガイドは、レガシーな OpenSpec ワークフローから OPSX への移行を支援します。移行はスムーズに行えるよう設計されています。既存の作業は保持され、新しいシステムはより高い柔軟性を提供します。

<a id="whats-changing"></a>
## 何が変わるのか？

OPSX は、フェーズ固定の古いワークフローを、流動的でアクションベースのアプローチに置き換えます。主な変化は次のとおりです。

| 観点 | レガシー | OPSX |
|--------|--------|------|
| **コマンド** | `/openspec:proposal`、`/openspec:apply`、`/openspec:archive` | デフォルト: `/opsx:propose`、`/opsx:explore`、`/opsx:apply`、`/opsx:update`、`/opsx:sync`、`/opsx:archive`（拡張ワークフローのコマンドは任意） |
| **ワークフロー** | すべての成果物を一度に作成する | 段階的に、または一度にすべて作成できる（選択可能） |
| **前の段階に戻る** | 扱いにくいフェーズゲート | 自然に行える。いつでもどの成果物でも更新できる |
| **カスタマイズ** | 固定された構造 | スキーマ駆動で、完全にハック可能 |
| **設定** | マーカー付きの `CLAUDE.md` と `project.md` | `openspec/config.yaml` のすっきりした設定 |

**思想の変化:** 作業は直線的ではありません。OPSX は、直線的であるふりをやめます。

---

<a id="before-you-begin"></a>
## 始める前に

<a id="your-existing-work-is-safe"></a>
### 既存の作業は安全です

移行プロセスは、既存のものを保持することを念頭に設計されています。

- **`openspec/changes/` 内のアクティブな変更** — 完全に保持されます。OPSX のコマンドで続行できます。
- **アーカイブされた変更** — 手を加えません。履歴はそのまま残ります。
- **`openspec/specs/` 内のメイン仕様** — 手を加えません。これらが信頼できる唯一の情報源です。
- **CLAUDE.md、AGENTS.md などにあるあなたの内容** — 保持されます。削除されるのは OpenSpec のマーカーブロックだけで、あなたが書いたものはすべて残ります。

<a id="what-gets-removed"></a>
### 削除されるもの

置き換えられる、OpenSpec が管理するファイルだけです。

| 対象 | 理由 |
|------|------|
| レガシーなスラッシュコマンドのディレクトリ/ファイル | 新しいスキルシステムに置き換えられるため |
| `openspec/AGENTS.md` | 時代遅れのワークフロートリガーのため |
| `CLAUDE.md`、`AGENTS.md` などにある OpenSpec のマーカー | もう必要ないため |

**ツールごとのレガシーなコマンドの場所**（例です。お使いのツールでは異なる場合があります）:

- Claude Code: `.claude/commands/openspec/`
- Cursor: `.cursor/commands/openspec-*.md`
- Devin Desktop（旧 Windsurf）: `.windsurf/workflows/openspec-*.md`
- Cline: `.clinerules/workflows/openspec-*.md`
- Roo: `.roo/commands/openspec-*.md`
- GitHub Copilot: `.github/prompts/openspec-*.prompt.md`（IDE 拡張機能のみ。Copilot CLI では非対応）
- Codex: OpenSpec は現在、正規の `.agents/skills/openspec-*` パスを使用します。以前の `.codex/skills` パス配下にある、OpenSpec が管理する `SKILL.md` ファイルは、置き換え先が存在した後にのみ整理されます。カスタムファイルや内容が異なるコピーはそのまま残ります。マーカーのない `.agents` ツリーにすでに OpenSpec のスキルが含まれている場合、OpenSpec はレガシーなディレクトリから推測するのではなく、既存の Codex（`$openspec-*`）または汎用（`/openspec-*`）のレンダリングを維持します。切り替える場合は、`openspec init` で `codex` を明示的に選択してください。レガシーなプロンプトのクリーンアップは、引き続き `$CODEX_HOME/prompts` または `~/.codex/prompts` にある、OpenSpec の許可リストに載ったファイル名のみを対象とします。
- その他（Augment、Continue、Amazon Q など）

移行処理は、設定済みのツールを検出し、そのレガシーなファイルをクリーンアップします。

削除対象のリストは長く見えるかもしれませんが、これらはすべて OpenSpec がもともと作成したファイルです。あなた自身の内容が削除されることはありません。

<a id="what-needs-your-attention"></a>
### あなたの対応が必要なもの

手動での移行が必要なファイルが1つあります。

**`openspec/project.md`** — このファイルには、あなたが書いたプロジェクトのコンテキストが含まれている可能性があるため、自動では削除されません。次の対応が必要です。

1. 内容を確認する
2. 有用なコンテキストを `openspec/config.yaml` に移す（後述のガイダンスを参照）
3. 準備ができたらファイルを削除する

**この変更を行った理由:**

古い `project.md` は受動的でした。エージェントが読むかもしれないし、読まないかもしれず、読んだ内容を忘れるかもしれません。信頼性に一貫性がないことがわかりました。

新しい `config.yaml` のコンテキストは、**すべての OpenSpec の計画リクエストに能動的に注入されます**。つまり、AI が成果物を作成するとき、プロジェクトの規約、技術スタック、ルールが常に存在します。信頼性が高まります。

**トレードオフ:**

コンテキストはすべてのリクエストに注入されるため、簡潔にする必要があります。本当に重要なことに絞ってください。
- 技術スタックと主要な規約
- AI が知っておくべき、自明ではない制約
- これまでよく無視されていたルール

完璧にしようと心配する必要はありません。何が最も効果的かは、私たちもまだ学んでいる途中であり、実験を通じてコンテキスト注入の仕組みを改善していきます。

---

<a id="running-the-migration"></a>
## 移行の実行

`openspec init` と `openspec update` はどちらも、レガシーなファイルを検出し、同じクリーンアップの手順を案内します。状況に合う方を使ってください。

- 新規インストールのデフォルトはプロファイル `core`（`propose`、`explore`、`apply`、`update`、`sync`、`archive`）です。
- 移行したインストールでは、必要に応じて `custom` プロファイルを書き込むことで、以前にインストールしていたワークフローが保持されます。

<a id="using-openspec-init"></a>
### `openspec init` を使う

新しいツールを追加したい場合や、セットアップするツールを再構成したい場合に実行します。

```bash
openspec init
```

init コマンドはレガシーなファイルを検出し、クリーンアップを案内します。

```
Upgrading to the new OpenSpec

OpenSpec now uses agent skills, the emerging standard across coding
agents. This simplifies your setup while keeping everything working
as before.

Files to remove
No user content to preserve:
  • .claude/commands/openspec/
  • openspec/AGENTS.md

Files to update
OpenSpec markers will be removed, your content preserved:
  • CLAUDE.md
  • AGENTS.md

Needs your attention
  • openspec/project.md
    We won't delete this file. It may contain useful project context.

    The new openspec/config.yaml has a "context:" section for planning
    context. This is included in every OpenSpec request and works more
    reliably than the old project.md approach.

    Review project.md, move any useful content to config.yaml's context
    section, then delete the file when ready.

? Upgrade and clean up legacy files? (Y/n)
```

（注: 上記は CLI の実際の出力例のため、英語のまま掲載しています。）

**「はい」と答えたときに起こること:**

1. レガシーなスラッシュコマンドのディレクトリが削除される
2. `CLAUDE.md`、`AGENTS.md` などから OpenSpec のマーカーが取り除かれる（あなたの内容は残る）
3. `openspec/AGENTS.md` が削除される
4. 新しいスキルが `.claude/skills/` にインストールされる
5. デフォルトのスキーマを持つ `openspec/config.yaml` が作成される

<a id="using-openspec-update"></a>
### `openspec update` を使う

移行して、既存のツールを最新バージョンに更新するだけでよい場合に実行します。

```bash
openspec update
```

update コマンドもレガシーな成果物を検出してクリーンアップし、その後、現在のプロファイルと提供方式の設定に合わせて、生成されたスキル/コマンドを更新します。

<a id="non-interactive--ci-environments"></a>
### 非対話型 / CI 環境

スクリプトによる移行の場合:

```bash
openspec init --force --tools claude
```

`--force` フラグはプロンプトをスキップし、クリーンアップを自動的に承認します。

これには、グローバルな Codex プロンプトディレクトリにある、OpenSpec が管理する Codex プロンプトファイルのクリーンアップも含まれます。クリーンアップは、OpenSpec の許可リストに載ったレガシーな Codex プロンプトのファイル名のみを対象とし、置き換え先の `.agents/skills/openspec-*` スキルが存在した後にのみ削除し、その他のファイルはすべて保持します。

---

<a id="migrating-projectmd-to-configyaml"></a>
## project.md を config.yaml に移行する

古い `openspec/project.md` は、プロジェクトのコンテキストのための自由形式の Markdown ファイルでした。新しい `openspec/config.yaml` は構造化されており、そして重要なことに、**すべての計画リクエストに注入される**ため、AI が作業するときに規約が常に存在します。

<a id="before-projectmd"></a>
### 移行前（project.md）

```markdown
# Project Context

This is a TypeScript monorepo using React and Node.js.
We use Jest for testing and follow strict ESLint rules.
Our API is RESTful and documented in docs/api.md.

## Conventions

- All public APIs must maintain backwards compatibility
- New features should include tests
- Use Given/When/Then format for specifications
```

<a id="after-configyaml"></a>
### 移行後（config.yaml）

```yaml
schema: spec-driven

context: |
  Tech stack: TypeScript, React, Node.js
  Testing: Jest with React Testing Library
  API: RESTful, documented in docs/api.md
  We maintain backwards compatibility for all public APIs

rules:
  proposal:
    - Include rollback plan for risky changes
  specs:
    - Use Given/When/Then format for scenarios
    - Reference existing patterns before inventing new ones
  design:
    - Include sequence diagrams for complex flows
```

<a id="key-differences"></a>
### 主な違い

| project.md | config.yaml |
|------------|-------------|
| 自由形式の Markdown | 構造化された YAML |
| ひとかたまりのテキスト | コンテキストと、成果物ごとのルールに分離 |
| いつ使われるか不明確 | コンテキストはすべての成果物に、ルールは対応する成果物にのみ表示される |
| スキーマの選択なし | 明示的な `schema:` フィールドでデフォルトのワークフローを設定 |

<a id="what-to-keep-what-to-drop"></a>
### 残すもの、捨てるもの

移行の際は、取捨選択してください。自問してみましょう。「AI は、*すべての*計画リクエストでこれを必要とするか？」

**`context:` に適しているもの**
- 技術スタック（言語、フレームワーク、データベース）
- 主要なアーキテクチャパターン（モノレポ、マイクロサービスなど）
- 自明ではない制約（「ライブラリ X は〜の理由で使えない」）
- よく無視される重要な規約

**代わりに `rules:` に移すもの**
- 成果物固有の書式（「仕様では Given/When/Then を使う」）
- レビュー基準（「提案にはロールバック計画を含めること」）
- これらは対応する成果物にのみ表示されるため、他のリクエストが軽く保たれます

**完全に省くもの**
- AI がすでに知っている一般的なベストプラクティス
- 要約できる冗長な説明
- 現在の作業に影響しない過去の経緯

<a id="migration-steps"></a>
### 移行手順

1. **config.yaml を作成する**（init でまだ作成されていない場合）:
   ```yaml
   schema: spec-driven
   ```

2. **コンテキストを追加する**（簡潔に。これはすべてのリクエストに入ります）:
   ```yaml
   context: |
     Your project background goes here.
     Focus on what the AI genuinely needs to know.
   ```

3. **成果物ごとのルールを追加する**（任意）:
   ```yaml
   rules:
     proposal:
       - Your proposal-specific guidance
     specs:
       - Your spec-writing rules
   ```

4. 有用なものをすべて移したら、**project.md を削除する**。

**考えすぎないでください。** 必須のものから始めて、反復しましょう。AI が重要なことを見落としていることに気づいたら、追加します。コンテキストが肥大化していると感じたら、削ります。これは生きたドキュメントです。

<a id="need-help-use-this-prompt"></a>
### 助けが必要？ このプロンプトを使ってください

project.md をどう要約すればよいかわからない場合は、AI アシスタントに頼みましょう。

```
I'm migrating from OpenSpec's old project.md to the new config.yaml format.

Here's my current project.md:
[paste your project.md content]

Please help me create a config.yaml with:
1. A concise `context:` section (this gets injected into every planning request, so keep it tight—focus on tech stack, key constraints, and conventions that often get ignored)
2. `rules:` for specific artifacts if any content is artifact-specific (e.g., "use Given/When/Then" belongs in specs rules, not global context)

Leave out anything generic that AI models already know. Be ruthless about brevity.
```

（注: 上記のプロンプトは、アシスタントに正確に伝わるよう、原文の英語のまま掲載しています。）

AI が、何が不可欠で、何を削れるかの見極めを手伝ってくれます。

---

<a id="the-new-commands"></a>
## 新しいコマンド

コマンドの利用可否は、プロファイルによって異なります。

**デフォルト（`core` プロファイル）:**

| コマンド | 目的 |
|---------|---------|
| `/opsx:propose` | 変更を作成し、計画用成果物を1ステップで生成する |
| `/opsx:explore` | 構造にとらわれずにアイデアを考え抜く |
| `/opsx:apply` | tasks.md のタスクを実装する |
| `/opsx:update` | 変更の計画用成果物を修正し、整合性を保つ |
| `/opsx:sync` | デルタ仕様をメイン仕様にマージする |
| `/opsx:archive` | 変更を確定してアーカイブする |

**拡張ワークフロー（カスタム選択）:**

| コマンド | 目的 |
|---------|---------|
| `/opsx:new` | 新しい変更の骨組みを開始する |
| `/opsx:continue` | 次の成果物を作成する（1つずつ） |
| `/opsx:ff` | ファストフォワード — 計画用成果物を一度に作成する |
| `/opsx:verify` | 実装が仕様に一致していることを検証する |
| `/opsx:bulk-archive` | 複数の変更を一度にアーカイブする |
| `/opsx:onboard` | ガイド付きのエンドツーエンドのオンボーディングワークフロー |

拡張コマンドは、`openspec config profile` で有効にしてから `openspec update` を実行します。

<a id="command-mapping-from-legacy"></a>
### レガシーからのコマンド対応表

| レガシー | OPSX での同等のもの |
|--------|-----------------|
| `/openspec:proposal` | `/opsx:propose`（デフォルト）、または `/opsx:new` の後に `/opsx:ff`（拡張） |
| `/openspec:apply` | `/opsx:apply` |
| `/openspec:archive` | `/opsx:archive` |

<a id="new-capabilities"></a>
### 新しい機能

これらの機能は、拡張ワークフローのコマンドセットの一部です。

**きめ細かな成果物の作成:**
```
/opsx:continue
```
依存関係に基づいて、成果物を1つずつ作成します。各ステップをレビューしたいときに使ってください。

**探索モード:**
```
/opsx:explore
```
変更を決める前に、パートナーと一緒にアイデアを考え抜きます。

---

<a id="understanding-the-new-architecture"></a>
## 新しいアーキテクチャを理解する

<a id="from-phase-locked-to-fluid"></a>
### フェーズ固定から流動的へ

レガシーなワークフローは、直線的な進行を強制していました。

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   PLANNING   │ ───► │ IMPLEMENTING │ ───► │   ARCHIVING  │
│    PHASE     │      │    PHASE     │      │    PHASE     │
└──────────────┘      └──────────────┘      └──────────────┘

実装中に、設計が間違っていたと気づいたら？
残念ながら、フェーズゲートがあるため簡単には戻れません。
```

OPSX は、フェーズではなくアクションを使います。

```
         ┌───────────────────────────────────────────────┐
         │           ACTIONS (not phases)                │
         │                                               │
         │     new ◄──► continue ◄──► apply ◄──► archive │
         │      │          │           │             │   │
         │      └──────────┴───────────┴─────────────┘   │
         │                    any order                  │
         └───────────────────────────────────────────────┘
```

<a id="dependency-graph"></a>
### 依存関係グラフ

成果物は有向グラフを形成します。依存関係は関門ではなく、手段です。

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

`/opsx:continue` を実行すると、準備ができているものを確認し、次の成果物を提案します。準備ができている複数の成果物を、任意の順序で作成することもできます。

<a id="skills-vs-commands"></a>
### スキルとコマンド

レガシーなシステムは、ツール固有のコマンドファイルを使っていました。

```
.claude/commands/openspec/
├── proposal.md
├── apply.md
└── archive.md
```

OPSX は、新しく登場しつつある**スキル**標準を使います。

```
.claude/skills/
├── openspec-explore/SKILL.md
├── openspec-new-change/SKILL.md
├── openspec-continue-change/SKILL.md
├── openspec-apply-change/SKILL.md
└── ...
```

スキルは複数の AI コーディングツールで認識され、より豊富なメタデータを提供します。

Codex は OPSX ではスキルのみです。OpenSpec は Codex のカスタムプロンプトファイルを生成しなくなりました。代わりに、生成された `.agents/skills/openspec-*` ディレクトリを使用してください。

---

<a id="continuing-existing-changes"></a>
## 既存の変更を続ける

進行中の変更は、OPSX のコマンドでシームレスに動作します。

**レガシーなワークフローから引き継いだアクティブな変更がある場合は？**

```
/opsx:apply add-my-feature
```

OPSX は既存の成果物を読み込み、中断したところから続行します。

**既存の変更に成果物を追加したい場合は？**

```
/opsx:continue add-my-feature
```

すでに存在するものに基づいて、作成できるものを表示します。

**状況を確認する必要がある場合は？**

```bash
openspec status --change add-my-feature
```

---

<a id="the-new-config-system"></a>
## 新しい設定システム

<a id="configyaml-structure"></a>
### config.yaml の構造

```yaml
# Required: Default schema for new changes
schema: spec-driven

# Optional: Project context (max 50KB)
# Injected into ALL artifact instructions
context: |
  Your project background, tech stack,
  conventions, and constraints.

# Optional: Per-artifact rules
# Only injected into matching artifacts
rules:
  proposal:
    - Include rollback plan
  specs:
    - Use Given/When/Then format
  design:
    - Document fallback strategies
  tasks:
    - Break into 2-hour maximum chunks
```

<a id="schema-resolution"></a>
### スキーマの解決

使用するスキーマを決定するとき、OPSX は次の順序で確認します。

1. **CLI フラグ**: `--schema <name>`（最優先）
2. **変更のメタデータ**: 変更ディレクトリ内の `.openspec.yaml`
3. **プロジェクト設定**: `openspec/config.yaml`
4. **デフォルト**: `spec-driven`

<a id="available-schemas"></a>
### 利用可能なスキーマ

| スキーマ | 成果物 | 適した用途 |
|--------|-----------|----------|
| `spec-driven` | proposal → specs → design → tasks | ほとんどのプロジェクト |

利用可能なすべてのスキーマを一覧表示するには:

```bash
openspec schemas
```

<a id="custom-schemas"></a>
### カスタムスキーマ

独自のワークフローを作成します。

```bash
openspec schema init my-workflow
```

または、既存のものをフォークします。

```bash
openspec schema fork spec-driven my-workflow
```

詳細は [カスタマイズ](customization.md) を参照してください。

---

<a id="troubleshooting"></a>
## トラブルシューティング

<a id="legacy-files-detected-in-non-interactive-mode"></a>
### 「Legacy files detected in non-interactive mode」

CI または非対話型の環境で実行しています。次を使用してください。

```bash
openspec init --force
```

<a id="commands-not-appearing-after-migration"></a>
### 移行後にコマンドが表示されない

IDE を再起動してください。スキルは起動時に検出されます。

<a id="unknown-artifact-id-in-rules"></a>
### 「Unknown artifact ID in rules」

`rules:` のキーが、スキーマの成果物 ID と一致しているか確認してください。

- **spec-driven**: `proposal`、`specs`、`design`、`tasks`

有効な成果物 ID を確認するには、次を実行します。

```bash
openspec schemas --json
```

<a id="config-not-being-applied"></a>
### 設定が適用されない

1. ファイルが `openspec/config.yaml` にあることを確認する（`.yml` ではない）
2. YAML の構文を検証する
3. 設定の変更はすぐに反映されます。再起動は不要です

<a id="projectmd-not-migrated"></a>
### project.md が移行されない

システムは、`project.md` にあなた独自の内容が含まれている可能性があるため、意図的にこれを保持します。手動で確認し、有用な部分を `config.yaml` に移してから、削除してください。

<a id="want-to-see-what-would-be-cleaned-up"></a>
### クリーンアップされる内容を確認したい場合は？

init を実行し、クリーンアップのプロンプトを拒否してください。何も変更されることなく、検出結果の全体の概要を確認できます。

---

<a id="quick-reference"></a>
## クイックリファレンス

<a id="files-after-migration"></a>
### 移行後のファイル

```
project/
├── openspec/
│   ├── specs/                    # 変更なし
│   ├── changes/                  # 変更なし
│   │   └── archive/              # 変更なし
│   └── config.yaml               # 新規: プロジェクト設定
├── .claude/
│   └── skills/                   # 新規: OPSX のスキル
│       ├── openspec-propose/     # デフォルトの core プロファイル
│       ├── openspec-explore/
│       ├── openspec-apply-change/
│       ├── openspec-update-change/
│       ├── openspec-sync-specs/
│       ├── openspec-archive-change/
│       └── ...                   # 拡張プロファイルでは new/continue/ff/etc. が追加される
├── CLAUDE.md                     # OpenSpec のマーカーが削除され、あなたの内容は保持される
└── AGENTS.md                     # OpenSpec のマーカーが削除され、あなたの内容は保持される
```

<a id="whats-gone"></a>
### なくなったもの

- `.claude/commands/openspec/` — `.claude/skills/` に置き換えられました
- `openspec/AGENTS.md` — 時代遅れになりました
- `openspec/project.md` — `config.yaml` に移行してから削除してください
- `CLAUDE.md`、`AGENTS.md` などにある OpenSpec のマーカーブロック

<a id="command-cheatsheet"></a>
### コマンド早見表

```text
/opsx:propose      Start quickly (default core profile)
/opsx:apply        Implement tasks
/opsx:archive      Finish and archive

# Expanded workflow (if enabled):
/opsx:new          Scaffold a change
/opsx:continue     Create next artifact
/opsx:ff           Create planning artifacts
```

---

<a id="getting-help"></a>
## ヘルプを得る

- **Discord**: [discord.gg/YctCnvvshC](https://discord.gg/YctCnvvshC)
- **GitHub Issues**: [github.com/Fission-AI/OpenSpec/issues](https://github.com/Fission-AI/OpenSpec/issues)
- **ドキュメント**: OPSX の完全なリファレンスは [docs/opsx.md](opsx.md)
