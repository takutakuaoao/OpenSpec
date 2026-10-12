<a id="customization"></a>
# カスタマイズ

OpenSpec には、3つのレベルのカスタマイズがあります。

| レベル | できること | 適した用途 |
|-------|--------------|----------|
| **プロジェクト設定** | デフォルトを設定し、コンテキストやルールを注入する | ほとんどのチーム |
| **カスタムスキーマ** | 独自のワークフロー成果物を定義する | 独自のプロセスを持つチーム |
| **グローバルオーバーライド** | すべてのプロジェクトでスキーマを共有する | パワーユーザー |

---

<a id="project-configuration"></a>
## プロジェクト設定

`openspec/config.yaml` ファイルは、チーム向けに OpenSpec をカスタマイズする最も簡単な方法です。次のことができます。

- **デフォルトのスキーマを設定する** - コマンドごとに `--schema` を指定する必要がなくなります
- **プロジェクトのコンテキストを注入する** - AI が技術スタックや規約などを把握します
- **成果物ごとのルールを追加する** - 特定の成果物向けのカスタムルール
- **操作ごとのガイダンスを追加する** - apply とアーカイブの作業に関する、助言的な好み
- **連携に関する選択を記憶する** - たとえば、[GitHub Copilot クラウドコーディングエージェント](supported-tools.md#github-copilot-cloud-coding-agent) のオプトイン

<a id="quick-setup"></a>
### クイックセットアップ

```bash
openspec init
```

これにより、対話形式で設定を作成できます。手動で作成することもできます。

```yaml
# openspec/config.yaml
schema: spec-driven

context: |
  Tech stack: TypeScript, React, Node.js, PostgreSQL
  API style: RESTful, documented in docs/api.md
  Testing: Jest + React Testing Library
  We value backwards compatibility for all public APIs

rules:
  proposal:
    - Include rollback plan
    - Identify affected teams
  specs:
    - Use Given/When/Then format
    - Reference existing patterns before inventing new ones

operations:
  apply:
    guidance:
      - Run focused tests before the full suite
  archive:
    guidance:
      - Keep the completion summary concise

# Set by `openspec init` when you choose (or decline) the GitHub Copilot
# cloud coding agent; controls whether `init`/`update` generate its files.
githubCopilot:
  cloudAgent: false
```

<a id="how-it-works"></a>
### 仕組み

**デフォルトのスキーマ:**

```bash
# Without config
openspec new change my-feature --schema spec-driven

# With config - schema is automatic
openspec new change my-feature
```

**コンテキストとルールの注入:**

成果物を生成するとき、コンテキストとルールが AI のプロンプトに注入されます。

```xml
<context>
Tech stack: TypeScript, React, Node.js, PostgreSQL
...
</context>

<rules>
- Include rollback plan
- Identify affected teams
</rules>

<template>
[Schema's built-in template]
</template>
```

- **コンテキスト**は、すべての成果物に表示されます
- **ルール**は、一致する成果物にのみ表示されます

**操作ガイダンス:**

`operations.apply.guidance` と `operations.archive.guidance` は、エージェントがそれらの
操作をどのように行うべきかについての、助言的な指示の省略可能な配列です。これらは
`rules` とは別のものです。操作ガイダンスは成果物の内容を制約せず、
成果物のルールが操作ガイダンスとして読み替えられることもありません。

apply とアーカイブは、実行時にこれらの入力を取得します。

```bash
openspec instructions apply --change my-feature --json
openspec instructions archive --change my-feature --json
```

どちらのインターフェースも、現在のプロジェクトの `context` と、対応する
`operationGuidance` を、それぞれ独立した省略可能なフィールドとして返します。呼び出しのたびに、
解決されたルートから最新のスナップショットを読み取ります。`--store <id>` が選択されている場合、
変更、コンテキスト、ガイダンスのすべてが、現在のリポジトリではなく、そのストアから取得されます。
アーカイブの instructions コマンドは読み取り専用です。デルタ仕様の検査やマージ、
メイン仕様の書き込み、変更の移動、静的なアーカイブワークフローの実行は行いません。

プロジェクトのコンテキストは、プロンプトレベルで必須の入力です。生成されるワークフローはこれを読み取り、
関連するプロジェクトの事実、規約、制約を適用します。操作ガイダンスは、
省略可能な追加のアドバイスです。ワークフローはすべての項目を検討し、組み込みのワークフローと
両立する、該当する項目に従います。

どちらのフィールドも、CLI が制御する状態、解決されたパス、組み込みの
ステップ、ユーザーの明示的な選択、成果物のルールとは別のものとして扱われます。ワークフローは、
制御する値を保持したまま、コンテキストの競合を報告します。該当しない、または
競合するガイダンスには従わず、その理由を説明します。どちらのフィールドも強制可能なチェックではなく、
ワークフローは、ユーザーが別途その内容を求めない限り、そのテキストを実装ファイル、仕様、変更の
成果物、要約にコピーしません。

**アーカイブと仕様同期の入力の安全性:**

アーカイブ、一括アーカイブ、単独の同期は、`openspec status --json` の
`artifactPaths.specs.existingOutputPaths` を、デルタ仕様の唯一の
入力元として使用します。`specs` 成果物のないスキーマや、具体的な出力リストが空である変更には、
同期するものがありません。他の成果物からデルタ仕様が推測されることはありません。

意味的なマージがメイン仕様を書き込む前に、ワークフローは現在の
`openspec instructions specs --change <name> --json` の出力を取得します。返される
`specs` のルールは、そのマージで生成されるメイン仕様のみを制約します。単一のアーカイブは
そのスナップショットをインラインの同期に渡し、単独の同期は直接それを取得し、
一括アーカイブは最初の仕様の書き込みの前に、必要なすべてのスナップショットを取得します。
ゼロ以外の終了コード、または無効な JSON のアーカイブ/仕様 instructions の応答は、
空の入力ではなく、取得の失敗です。ワークフローは、影響を受ける仕様の書き込みまたは変更の
移動の前に停止します（一括アーカイブの場合は、バッチの書き込みや移動の前です）。

この設定は、アーカイブの実行フェーズ、ユーザーへのプロンプト、
ファイルシステム操作、意味的なマージの責任範囲、`openspec archive` コマンドの直接実行、
および成果物の `rules` の構造と出力を変更しません。

<a id="schema-resolution-order"></a>
### スキーマの解決順序

OpenSpec がスキーマを必要とするとき、次の順序で確認します。

1. CLI フラグ: `--schema <name>`
2. 変更のメタデータ（変更フォルダ内の `.openspec.yaml`）
3. プロジェクト設定（`openspec/config.yaml`）
4. デフォルト（`spec-driven`）

---

<a id="custom-schemas"></a>
## カスタムスキーマ

プロジェクト設定では足りない場合、完全にカスタムのワークフローを持つ独自のスキーマを作成できます。カスタムスキーマは、プロジェクトの `openspec/schemas/` ディレクトリに置かれ、コードと一緒にバージョン管理されます。

```text
your-project/
├── openspec/
│   ├── config.yaml        # プロジェクト設定
│   ├── schemas/           # カスタムスキーマはここに置く
│   │   └── my-workflow/
│   │       ├── schema.yaml
│   │       └── templates/
│   └── changes/           # あなたの変更
└── src/
```

<a id="fork-an-existing-schema"></a>
### 既存のスキーマをフォークする

カスタマイズする最速の方法は、組み込みのスキーマをフォークすることです。

```bash
openspec schema fork spec-driven my-workflow
```

これにより、`spec-driven` スキーマ全体が `openspec/schemas/my-workflow/` にコピーされ、自由に編集できます。

**得られるもの:**

```text
openspec/schemas/my-workflow/
├── schema.yaml           # ワークフローの定義
└── templates/
    ├── proposal.md       # proposal 成果物のテンプレート
    ├── spec.md           # specs のテンプレート
    ├── design.md         # design のテンプレート
    └── tasks.md          # tasks のテンプレート
```

あとは `schema.yaml` を編集してワークフローを変更するか、テンプレートを編集して AI が生成する内容を変更します。

<a id="create-a-schema-from-scratch"></a>
### スキーマをゼロから作成する

まったく新しいワークフローを作る場合:

```bash
# Interactive
openspec schema init research-first

# Non-interactive
openspec schema init rapid \
  --description "Rapid iteration workflow" \
  --artifacts "proposal,tasks" \
  --default
```

<a id="schema-structure"></a>
### スキーマの構造

スキーマは、ワークフロー内の成果物と、それらが互いにどう依存するかを定義します。

```yaml
# openspec/schemas/my-workflow/schema.yaml
name: my-workflow
version: 1
description: My team's custom workflow

artifacts:
  - id: proposal
    generates: proposal.md
    description: Initial proposal document
    template: proposal.md
    instruction: |
      Create a proposal that explains WHY this change is needed.
      Focus on the problem, not the solution.
    requires: []

  - id: design
    generates: design.md
    description: Technical design
    template: design.md
    instruction: |
      Create a design document explaining HOW to implement.
    requires:
      - proposal    # Can't create design until proposal exists

  - id: tasks
    generates: tasks.md
    description: Implementation checklist
    template: tasks.md
    requires:
      - design

apply:
  requires: [tasks]
  tracks: tasks.md
```

**主なフィールド:**

| フィールド | 目的 |
|-------|---------|
| `id` | 一意の識別子。コマンドやルールで使用される |
| `generates` | 出力ファイル名（`specs/**/*.md` のような glob に対応） |
| `template` | `templates/` ディレクトリ内のテンプレートファイル |
| `instruction` | この成果物を作成するための AI への指示 |
| `requires` | 依存関係 - どの成果物が先に存在している必要があるか |

成果物は、書かせたい順序でリストしてください。`requires` は何が可能かを決め、
`artifacts:` リストの順序は、複数の成果物が同時に準備できているときに、
どれが先になるかを決めます。

<a id="templates"></a>
### テンプレート

テンプレートは、AI を導く Markdown ファイルです。その成果物を作成するときに、プロンプトに注入されます。

```markdown
<!-- templates/proposal.md -->
## Why

<!-- Explain the motivation for this change. What problem does this solve? -->

## What Changes

<!-- Describe what will change. Be specific about new capabilities or modifications. -->

## Impact

<!-- Affected code, APIs, dependencies, systems -->
```

テンプレートには、次のものを含められます。
- AI が埋めるべきセクション見出し
- AI へのガイダンスを記した HTML コメント
- 期待される構造を示す書式例

<a id="validate-your-schema"></a>
### スキーマを検証する

カスタムスキーマを使う前に、検証します。

```bash
openspec schema validate my-workflow
```

これは次をチェックします。
- `schema.yaml` の構文が正しいこと
- 参照されているすべてのテンプレートが存在すること
- 循環依存がないこと
- 成果物の ID が有効であること

<a id="use-your-custom-schema"></a>
### カスタムスキーマを使う

作成したら、次のようにスキーマを使用します。

```bash
# Specify on command
openspec new change feature --schema my-workflow

# Or set as default in config.yaml
schema: my-workflow
```

<a id="debug-schema-resolution"></a>
### スキーマの解決をデバッグする

どのスキーマが使われているかわからない場合は、次のコマンドで確認します。

```bash
# See where a specific schema resolves from
openspec schema which my-workflow

# List all available schemas
openspec schema which --all
```

出力には、プロジェクト、ユーザーディレクトリ、パッケージのどこから来ているかが表示されます。

```text
Schema: my-workflow
Source: project
Path: /path/to/project/openspec/schemas/my-workflow
```

---

> **注:** OpenSpec は、プロジェクト間で共有するためのユーザーレベルのスキーマも `~/.local/share/openspec/schemas/` でサポートしていますが、コードと一緒にバージョン管理できるため、`openspec/schemas/` のプロジェクトレベルのスキーマを推奨します。

---

<a id="examples"></a>
## 例

<a id="rapid-iteration-workflow"></a>
### 高速反復ワークフロー

素早く反復するための最小限のワークフロー:

```yaml
# openspec/schemas/rapid/schema.yaml
name: rapid
version: 1
description: Fast iteration with minimal overhead

artifacts:
  - id: proposal
    generates: proposal.md
    description: Quick proposal
    template: proposal.md
    instruction: |
      Create a brief proposal for this change.
      Focus on what and why, skip detailed specs.
    requires: []

  - id: tasks
    generates: tasks.md
    description: Implementation checklist
    template: tasks.md
    requires: [proposal]

apply:
  requires: [tasks]
  tracks: tasks.md
```

<a id="adding-a-review-artifact"></a>
### レビュー成果物を追加する

デフォルトをフォークして、レビューのステップを追加します。

```bash
openspec schema fork spec-driven with-review
```

その後、`schema.yaml` を編集して次を追加します。

```yaml
  - id: review
    generates: review.md
    description: Pre-implementation review checklist
    template: review.md
    instruction: |
      Create a review checklist based on the design.
      Include security, performance, and testing considerations.
    requires:
      - design

  - id: tasks
    # ... existing tasks config ...
    requires:
      - specs
      - design
      - review    # Now tasks require review too
```

---

<a id="community-schemas"></a>
## コミュニティスキーマ

OpenSpec は、独立したリポジトリを通じて配布される、コミュニティがメンテナンスするスキーマにも対応しています。これらは、OpenSpec を他のツールやシステムと統合する、方針の明確なワークフローを提供します。spec-kit の [github/spec-kit のコミュニティ拡張カタログ](https://github.com/github/spec-kit/tree/main/extensions) と似た仕組みです。

コミュニティスキーマは OpenSpec 本体には組み込まれていません。それぞれ独自のリポジトリで、独自のリリースサイクルで管理されています。使用するには、スキーマのバンドルをプロジェクトの `openspec/schemas/<schema-name>/` ディレクトリにコピーしてください（各リポジトリの README にインストール手順があります）。

| スキーマ | メンテナー | リポジトリ | 説明 |
|--------|-----------|-----------|-------------|
| `intent-driven` | @harikrishnan83 | [intent-driven-dev/openspec-schemas](https://github.com/intent-driven-dev/openspec-schemas/tree/main/openspec/schemas/intent-driven) | 実装前に、変更の意図、観測可能な振る舞い、技術設計、永続的なアーキテクチャ上の判断を記録します。変更ローカルな ADR レビューマニフェストを追加し、該当する長期的な判断を、不変で置き換え可能な ADR として書き出します。 |
| `superpowers-bridge` | @JiangWay | [JiangWay/openspec-schemas](https://github.com/JiangWay/openspec-schemas/tree/main/superpowers-bridge) | OpenSpec の成果物ガバナンスを、[obra/superpowers](https://github.com/obra/superpowers) の実行スキル（ブレインストーミング、計画作成、サブエージェントによる TDD、コードレビュー、仕上げ）と統合します。Superpowers が標準ではカバーしていない部分を補う、証拠重視の `retrospective` 成果物を追加します。 |
| `nanopm` | @nmrtn | [nmrtn/nanopm](https://github.com/nmrtn/nanopm/tree/main/openspec-schema) | PM ファーストのワークフロー。[nanopm](https://github.com/nmrtn/nanopm) の計画パイプライン（監査 → 戦略 → ロードマップ → PRD）を実装の上流で実行します。プロダクトの計画と、OpenSpec の仕様駆動のエンジニアリングワークフローをつなぎます。成果物は `.nanopm/` が存在すればそこから読み込まれ、proposal は監査を、design は戦略を、tasks は PRD の分解を元にします。 |
| `e2e-runbooks` | @Lukk17 | [Lukk17/openspec-schemas](https://github.com/Lukk17/openspec-schemas/tree/master/openspec/schemas/e2e-runbooks) | 機能（capability）単位のエンドツーエンドテストのランブック。各機能に、不変の仕様、不変のタスクテンプレート、実行ごとのタイムスタンプ付き実行記録が1つずつ付きます。アサーションは観測可能な振る舞い（HTTP ステータス、レスポンスボディ、永続化された状態。ログの部分文字列は不可）のみで、各実行では開始/終了の UTC 時刻、所要時間、LLM のトークン消費量の最善推定値が記録されます。 |
| `anvil` | @jikkujoyce | [jikkujoyce/openspec-schemas](https://github.com/jikkujoyce/openspec-schemas/tree/main/schemas/anvil) | TDD の規律と敵対的レビューのステップを備えた、仕様駆動のワークフロー。フロー: `proposal` → `specs` → `design` → `review` → `test-plan` → `tasks` → `apply` → `verify`。`review` は、新しいコンテキストを持つ読み取り専用のレビュアー（可能であれば別のモデル）によって書かれ、`test-plan`、`tasks`、`apply` をゲートするようエージェントに伝える `VERDICT:` 行を出力します。OpenSpec は成果物の存在しか確認しないため、ゲートは独自の CI やフックで強制してください。`test-plan` はすべての仕様シナリオを名前付きのテストに対応づけ、`verify` が監査する red/green の台帳を兼ねます。 |

> コミュニティスキーマを提供したいですか？ リポジトリへのリンクを添えて issue を作成するか、この表に行を追加する PR を送ってください。

---

<a id="see-also"></a>
## 関連項目

- [CLI リファレンス: スキーマコマンド](cli.md#schema-commands) - コマンドの完全なドキュメント
