<a id="opsx-workflow"></a>
# OPSX ワークフロー

> フィードバックは [Discord](https://discord.gg/YctCnvvshC) で歓迎します。

<a id="what-is-it"></a>
## これは何か？

OPSX は、OpenSpec の標準ワークフローになりました。

これは、OpenSpec の変更のための**流動的で反復的なワークフロー**です。硬直したフェーズはもうなく、いつでも実行できるアクションだけがあります。

<a id="why-this-exists"></a>
## なぜ存在するのか

レガシーな OpenSpec のワークフローは機能しますが、**閉じられています**。

- **指示がハードコードされている** — TypeScript の中に埋め込まれていて、変更できない
- **オール・オア・ナッシング** — 1つの大きなコマンドがすべてを作成し、個々の部分をテストできない
- **固定された構造** — 全員に同じワークフローで、カスタマイズできない
- **ブラックボックス** — AI の出力が悪くても、プロンプトを調整できない

**OPSX はこれを開放します。** 誰もが次のことをできるようになります。

1. **指示を試す** — テンプレートを編集して、AI がより良くなるかを確認する
2. **きめ細かくテストする** — 各成果物の指示を個別に検証する
3. **ワークフローをカスタマイズする** — 独自の成果物と依存関係を定義する
4. **素早く反復する** — テンプレートを変更し、すぐにテストでき、再ビルドは不要

```
Legacy workflow:                      OPSX:
┌────────────────────────┐           ┌────────────────────────┐
│  Hardcoded in package  │           │  schema.yaml           │◄── You edit this
│  (can't change)        │           │  templates/*.md        │◄── Or this
│        ↓               │           │        ↓               │
│  Wait for new release  │           │  Instant effect        │
│        ↓               │           │        ↓               │
│  Hope it's better      │           │  Test it yourself      │
└────────────────────────┘           └────────────────────────┘
```

**これはすべての人のためのものです:**
- **チーム** — 実際の働き方に合ったワークフローを作成する
- **パワーユーザー** — プロンプトを調整して、自分のコードベースでより良い AI の出力を得る
- **OpenSpec のコントリビューター** — リリースを待たずに新しいアプローチを試す

何が最も効果的かは、私たちもまだ学んでいる途中です。OPSX は、私たちが一緒に学ぶことを可能にします。

<a id="the-user-experience"></a>
## ユーザー体験

**直線的なワークフローの問題:**
「計画フェーズ」にいて、次に「実装フェーズ」にいて、それから「完了」。しかし、実際の作業はそのようには進みません。何かを実装して、設計が間違っていたと気づき、仕様を更新する必要が生じ、そして実装を続ける。直線的なフェーズは、実際の作業の進み方と相容れません。

**OPSX のアプローチ:**
- **フェーズではなくアクション** — 作成、実装、更新、アーカイブ — いつでも、どれでも実行できる
- **依存関係は手段である** — 何が可能かを示すものであり、次に何が必要かを示すものではない

```
  proposal ──→ specs ──→ design ──→ tasks ──→ implement
```

<a id="setup"></a>
## セットアップ

```bash
# Make sure you have openspec installed — skills are automatically generated
openspec init
```

これにより、AI コーディングアシスタントが自動検出する `.claude/skills/`（または同等の場所）にスキルが作成されます。

デフォルトでは、OpenSpec は `core` ワークフロープロファイル（`propose`、`explore`、`apply`、`update`、`sync`、`archive`）を使用します。拡張ワークフローのコマンド（`new`、`continue`、`ff`、`verify`、`bulk-archive`、`onboard`）が必要な場合は、`openspec config profile` で設定し、`openspec update` で適用します。

セットアップ中に、**プロジェクト設定**（`openspec/config.yaml`）を作成するかどうかを尋ねられます。これは任意ですが、推奨します。

<a id="project-configuration"></a>
## プロジェクト設定

プロジェクト設定では、デフォルトを設定し、プロジェクト固有のコンテキストをすべての成果物に注入できます。

<a id="creating-config"></a>
### 設定の作成

設定は `openspec init` の際に作成されます。手動で作成することもできます。

```yaml
# openspec/config.yaml
schema: spec-driven

context: |
  Tech stack: TypeScript, React, Node.js
  API conventions: RESTful, JSON responses
  Testing: Vitest for unit tests, Playwright for e2e
  Style: ESLint with Prettier, strict TypeScript

rules:
  proposal:
    - Include rollback plan
    - Identify affected teams
  specs:
    - Use Given/When/Then format for scenarios
  design:
    - Include sequence diagrams for complex flows
```

<a id="config-fields"></a>
### 設定フィールド

| フィールド | 型 | 説明 |
|-------|------|-------------|
| `schema` | string | 新しい変更のデフォルトスキーマ（例: `spec-driven`） |
| `context` | string | すべての成果物の指示に注入されるプロジェクトのコンテキスト |
| `rules` | object | 成果物 ID をキーとする、成果物ごとのルール |

<a id="how-it-works"></a>
### 仕組み

**スキーマの優先順位**（高いものから低いものの順）:
1. CLI フラグ（`--schema <name>`）
2. 変更のメタデータ（変更ディレクトリ内の `.openspec.yaml`）
3. プロジェクト設定（`openspec/config.yaml`）
4. デフォルト（`spec-driven`）

**コンテキストの注入:**
- コンテキストは、すべての成果物の指示の先頭に追加される
- `<context>...</context>` タグで囲まれる
- AI がプロジェクトの規約を理解するのを助ける

**ルールの注入:**
- ルールは、一致する成果物にのみ注入される
- `<rules>...</rules>` タグで囲まれる
- コンテキストの後、テンプレートの前に表示される

<a id="artifact-ids-by-schema"></a>
### スキーマごとの成果物 ID

**spec-driven**（デフォルト）:
- `proposal` — 変更の提案
- `specs` — 仕様
- `design` — 技術設計
- `tasks` — 実装タスク

<a id="config-validation"></a>
### 設定の検証

- `rules` 内の不明な成果物 ID は警告を生成する
- スキーマ名は、利用可能なスキーマに対して検証される
- コンテキストには 50KB のサイズ制限がある
- 不正な YAML は、行番号付きで報告される

<a id="troubleshooting"></a>
### トラブルシューティング

**「Unknown artifact ID in rules: X」**
- 成果物 ID がスキーマと一致しているか確認する（上記の一覧を参照）
- `openspec schemas --json` を実行して、各スキーマの成果物 ID を確認する

**設定が適用されない:**
- ファイルが `openspec/config.yaml` にあることを確認する（`.yml` ではない）
- バリデータで YAML の構文を確認する
- 設定の変更はすぐに反映される（再起動は不要）

**コンテキストが大きすぎる:**
- コンテキストは 50KB に制限されている
- 要約するか、代わりに外部ドキュメントにリンクする

<a id="commands"></a>
## コマンド

| コマンド | 動作 |
|---------|--------------|
| `/opsx:propose` | 変更を作成し、計画用成果物を1ステップで生成する（デフォルトのクイックパス） |
| `/opsx:explore` | アイデアを考え抜き、問題を調査し、要件を明確にする |
| `/opsx:new` | 新しい変更の骨組みを開始する（拡張ワークフロー） |
| `/opsx:continue` | 次の成果物を作成する（拡張ワークフロー） |
| `/opsx:ff` | 計画用成果物をファストフォワードする（拡張ワークフロー） |
| `/opsx:apply` | タスクを実装し、必要に応じて成果物を更新する |
| `/opsx:update` | 変更の計画用成果物を修正し、整合性を保つ |
| `/opsx:verify` | 成果物に照らして実装を検証する（拡張ワークフロー） |
| `/opsx:sync` | デルタ仕様をメイン仕様にマージする（任意） |
| `/opsx:archive` | 完了したらアーカイブする |
| `/opsx:bulk-archive` | 完了した複数の変更をアーカイブする（拡張ワークフロー） |
| `/opsx:onboard` | エンドツーエンドの変更のガイド付きウォークスルー（拡張ワークフロー） |

<a id="usage"></a>
## 使い方

<a id="explore-an-idea"></a>
### アイデアを探索する
```
/opsx:explore
```
アイデアを考え抜き、問題を調査し、選択肢を比較します。構造は不要で、思考パートナーとして使えます。洞察がまとまったら、`/opsx:propose`（デフォルト）または `/opsx:new`/`/opsx:ff`（拡張）に移行します。

<a id="start-a-new-change"></a>
### 新しい変更を始める
```
/opsx:propose
```
変更を作成し、実装の前に必要な計画用成果物を生成します。

拡張ワークフローを有効にしている場合は、代わりに次を使用できます。

```text
/opsx:new        # scaffold only
/opsx:continue   # create one artifact at a time
/opsx:ff         # create all planning artifacts at once
```

<a id="create-artifacts"></a>
### 成果物を作成する
```
/opsx:continue
```
依存関係に基づいて作成可能なものを表示し、成果物を1つ作成します。繰り返し使うことで、変更を段階的に組み立てます。

```
/opsx:ff add-dark-mode
```
すべての計画用成果物を一度に作成します。何を作るかが明確に見えているときに使います。

<a id="implement-the-fluid-part"></a>
### 実装する（流動的な部分）
```
/opsx:apply
```
タスクを順に進め、完了したものにチェックを付けます。複数の変更を並行して扱っている場合は `/opsx:apply <name>` を実行できます。そうでない場合は、会話から推測し、判断できなければ選択を促します。

<a id="updating-a-change"></a>
### 変更を更新する
```
/opsx:update add-dark-mode - we're storing the theme in a cookie now
```
変更の既存の計画用成果物を修正し、どの方向にも整合性を保ちます（設計の編集が提案にさかのぼって波及することもあります）。コードを編集することはありません。すべての編集は、事前にあなたに確認されます。新しい成果物を始めることなく、欠落しているファイルをどう扱うかについては、[update のリファレンス](commands.md#opsxupdate) を参照してください。

変更がすでに実装されている場合は、コードが修正後の計画に追いつくよう、`/opsx:apply` を推奨します。修正によって変更の*意図*が変わる場合は、代わりに新しく始めてください。[更新するか、やり直すか](#when-to-update-vs-start-fresh) を参照してください。

<a id="sync-delta-specs"></a>
### デルタ仕様を同期する
```text
/opsx:sync
```
現在の変更のデルタ仕様を、アーカイブせずにメインの `openspec/specs/` にマージします。変更はアクティブなままです。デルタ全体を適用します。`## REMOVED` の下の要件はメイン仕様から削除され、名前が変更されたものはその場でタイトルが変更され、デルタで言及されていない内容はそのまま残されます。同期は任意です。まだ同期していない場合は、アーカイブが先に同期するかどうかを尋ねます。アーカイブの前にメイン仕様を更新したいとき、並行する変更が、この変更が追加したばかりの仕様の上に構築する必要があるとき、またはアーカイブの前にマージ後のメイン仕様をレビューしたいときに使います。

<a id="finish-up"></a>
### 仕上げる
```
/opsx:archive   # Move to archive when done (prompts to sync specs if needed)
```

<a id="when-to-update-vs-start-fresh"></a>
## 更新するか、やり直すか

実装の前であれば、提案や仕様はいつでも編集できます。しかし、洗練が「これは別の作業だ」に変わるのはいつでしょうか？

<a id="what-a-proposal-captures"></a>
### 提案が記述するもの

提案は、3つのことを定義します。
1. **意図** — どんな問題を解決しようとしているのか？
2. **スコープ** — 範囲内と範囲外は何か？
3. **アプローチ** — どのように解決するのか？

問いは、どれがどのくらい変わったか、ということです。

<a id="update-the-existing-change-when"></a>
### 既存の変更を更新するとき:

**意図は同じで、実行方法を洗練させる**
- 考慮していなかったエッジケースを発見した
- アプローチには調整が必要だが、目標は変わらない
- 実装によって、設計が少しずれていたとわかった

**スコープが縮小する**
- 全体のスコープが大きすぎると気づき、まず MVP をリリースしたい
- 「ダークモードを追加する」→「ダークモードの切り替えを追加する（システム設定は v2 で）」

**学びに基づく修正**
- コードベースが、想定していた構造になっていなかった
- 依存関係が期待どおりに動作しない
- 「CSS 変数を使う」→「代わりに Tailwind の dark: プレフィックスを使う」

<a id="start-a-new-change-when"></a>
### 新しい変更を始めるとき:

**意図が根本的に変わった**
- 問題そのものが今は違う
- 「ダークモードを追加する」→「カスタムの色、フォント、余白を備えた包括的なテーマシステムを追加する」

**スコープが膨れ上がった**
- 変更が大きく育ち、本質的に別の作業になった
- 更新を重ねると、元の提案が見る影もなくなる
- 「ログインのバグを直す」→「認証システムを書き直す」

**元の変更を完了できる**
- 元の変更を「完了」にできる
- 新しい作業は単独で成り立ち、洗練ではない
- 「ダークモードの MVP を追加する」を完了 → アーカイブ → 新しい変更「ダークモードを強化する」

<a id="the-heuristics"></a>
### 判断の目安

```
                        ┌─────────────────────────────────────┐
                        │     Is this the same work?          │
                        └──────────────┬──────────────────────┘
                                       │
                    ┌──────────────────┼──────────────────┐
                    │                  │                  │
                    ▼                  ▼                  ▼
             Same intent?      >50% overlap?      Can original
             Same problem?     Same scope?        be "done" without
                    │                  │          these changes?
                    │                  │                  │
          ┌────────┴────────┐  ┌──────┴──────┐   ┌───────┴───────┐
          │                 │  │             │   │               │
         YES               NO YES           NO  NO              YES
          │                 │  │             │   │               │
          ▼                 ▼  ▼             ▼   ▼               ▼
       UPDATE            NEW  UPDATE       NEW  UPDATE          NEW
```

| テスト | 更新 | 新しい変更 |
|------|--------|------------|
| **同一性** | 「同じもの、洗練された」 | 「別の作業」 |
| **スコープの重複** | 50%超が重複 | 50%未満が重複 |
| **完了** | 変更なしでは「完了」にできない | 元の変更を完了でき、新しい作業は単独で成り立つ |
| **ストーリー** | 更新の連なりが一貫したストーリーを語る | パッチを当てると、わかりやすくなるどころか混乱する |

<a id="the-principle"></a>
### 原則

> **更新は文脈を保つ。新しい変更は明確さをもたらす。**
>
> 考えの履歴に価値があるときは、更新を選ぶ。
> パッチを当てるより、一から始めるほうが明確なときは、新しい変更を選ぶ。

git のブランチのようなものだと考えてください。
- 同じ機能に取り組んでいる間は、コミットし続ける
- 本当に新しい作業のときは、新しいブランチを始める
- 部分的な機能をマージして、フェーズ2のために一から始めることもある

<a id="whats-different"></a>
## 何が違うのか？

| | レガシー（`/openspec:proposal`） | OPSX（`/opsx:*`） |
|---|---|---|
| **構造** | 1つの大きな提案ドキュメント | 依存関係を持つ個別の成果物 |
| **ワークフロー** | 直線的なフェーズ: 計画 → 実装 → アーカイブ | 流動的なアクション — いつでも何でも実行できる |
| **反復** | 前の段階に戻りにくい | 学びに応じて成果物を更新する |
| **カスタマイズ** | 固定された構造 | スキーマ駆動（独自の成果物を定義できる） |

**重要な洞察:** 作業は直線的ではありません。OPSX は、直線的であるふりをやめます。

<a id="architecture-deep-dive"></a>
## アーキテクチャ詳細解説

このセクションでは、OPSX が内部でどのように動作し、レガシーなワークフローとどう比較されるかを説明します。
このセクションの例は拡張コマンドセット（`new`、`continue` など）を使用しています。デフォルトの `core` ユーザーは、同じ流れを `propose → apply → sync → archive` に置き換えて考えてください。

<a id="philosophy-phases-vs-actions"></a>
### 思想: フェーズとアクション

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         LEGACY WORKFLOW                                      │
│                    (Phase-Locked, All-or-Nothing)                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌──────────────┐      ┌──────────────┐      ┌──────────────┐             │
│   │   PLANNING   │ ───► │ IMPLEMENTING │ ───► │   ARCHIVING  │             │
│   │    PHASE     │      │    PHASE     │      │    PHASE     │             │
│   └──────────────┘      └──────────────┘      └──────────────┘             │
│         │                     │                     │                       │
│         ▼                     ▼                     ▼                       │
│   /openspec:proposal   /openspec:apply      /openspec:archive              │
│                                                                             │
│   • Creates ALL artifacts at once                                          │
│   • Can't go back to update specs during implementation                    │
│   • Phase gates enforce linear progression                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────────────────┐
│                            OPSX WORKFLOW                                     │
│                      (Fluid Actions, Iterative)                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│              ┌────────────────────────────────────────────┐                 │
│              │           ACTIONS (not phases)             │                 │
│              │                                            │                 │
│              │   new ◄──► continue ◄──► apply ◄──► archive │                 │
│              │    │          │           │           │    │                 │
│              │    └──────────┴───────────┴───────────┘    │                 │
│              │              any order                     │                 │
│              └────────────────────────────────────────────┘                 │
│                                                                             │
│   • Create artifacts one at a time OR fast-forward                         │
│   • Update specs/design/tasks during implementation                        │
│   • Dependencies enable progress, phases don't exist                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

<a id="component-architecture"></a>
### コンポーネントのアーキテクチャ

**レガシーなワークフロー**は、TypeScript にハードコードされたテンプレートを使用します。

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      LEGACY WORKFLOW COMPONENTS                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Hardcoded Templates (TypeScript strings)                                  │
│                    │                                                        │
│                    ▼                                                        │
│   Tool-specific configurators/adapters                                      │
│                    │                                                        │
│                    ▼                                                        │
│   Generated Command Files (.claude/commands/openspec/*.md)                  │
│                                                                             │
│   • Fixed structure, no artifact awareness                                  │
│   • Change requires code modification + rebuild                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**OPSX** は、外部のスキーマと依存関係グラフエンジンを使用します。

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         OPSX COMPONENTS                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   Schema Definitions (YAML)                                                 │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  name: spec-driven                                                  │   │
│   │  artifacts:                                                         │   │
│   │    - id: proposal                                                   │   │
│   │      generates: proposal.md                                         │   │
│   │      requires: []              ◄── Dependencies                     │   │
│   │    - id: specs                                                      │   │
│   │      generates: specs/**/*.md  ◄── Glob patterns                    │   │
│   │      requires: [proposal]      ◄── Enables after proposal           │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                    │                                                        │
│                    ▼                                                        │
│   Artifact Graph Engine                                                     │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  • Topological sort (dependency ordering)                           │   │
│   │  • State detection (filesystem existence)                           │   │
│   │  • Rich instruction generation (templates + context)                │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                    │                                                        │
│                    ▼                                                        │
│   Skill Files (.claude/skills/openspec-*/SKILL.md)                          │
│                                                                             │
│   • Cross-editor compatible (Claude Code, Cursor, Devin)                    │
│   • Skills query CLI for structured data                                    │
│   • Fully customizable via schema files                                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

<a id="dependency-graph-model"></a>
### 依存関係グラフのモデル

成果物は有向非巡回グラフ（DAG）を形成します。依存関係は関門ではなく、**手段**です。

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
                                  │
                                  ▼
                          ┌──────────────┐
                          │ APPLY PHASE  │
                          │ (requires:   │
                          │  tasks)      │
                          └──────────────┘
```

**状態遷移:**

```
   BLOCKED ────────────────► READY ────────────────► DONE
      │                        │                       │
   Missing                  All deps               File exists
   dependencies             are DONE               on filesystem
```

BLOCKED = 依存先が不足、READY = すべての依存先が DONE、DONE = ファイルシステム上にファイルが存在する。

<a id="information-flow"></a>
### 情報の流れ

**レガシーなワークフロー** — エージェントは静的な指示を受け取ります。

```
  User: "/openspec:proposal"
           │
           ▼
  ┌─────────────────────────────────────────┐
  │  Static instructions:                   │
  │  • Create proposal.md                   │
  │  • Create tasks.md                      │
  │  • Create design.md                     │
  │  • Create delta spec files              │
  │                                         │
  │  No awareness of what exists or         │
  │  dependencies between artifacts         │
  └─────────────────────────────────────────┘
           │
           ▼
  Agent creates ALL artifacts in one go
```

**OPSX** — エージェントは、豊富なコンテキストを照会します。

```
  User: "/opsx:continue"
           │
           ▼
  ┌──────────────────────────────────────────────────────────────────────────┐
  │  Step 1: Query current state                                             │
  │  ┌────────────────────────────────────────────────────────────────────┐  │
  │  │  $ openspec status --change "add-auth" --json                      │  │
  │  │                                                                    │  │
  │  │  {                                                                 │  │
  │  │    "artifacts": [                                                  │  │
  │  │      {"id": "proposal", "status": "done"},                         │  │
  │  │      {"id": "specs", "status": "ready"},      ◄── First ready      │  │
  │  │      {"id": "design", "status": "ready"},                          │  │
  │  │      {"id": "tasks", "status": "blocked",                          │  │
  │  │       "missingDeps": ["specs", "design"]}                          │  │
  │  │    ]                                                               │  │
  │  │  }                                                                 │  │
  │  └────────────────────────────────────────────────────────────────────┘  │
  │                                                                          │
  │  Step 2: Get rich instructions for ready artifact                        │
  │  ┌────────────────────────────────────────────────────────────────────┐  │
  │  │  $ openspec instructions specs --change "add-auth" --json          │  │
  │  │                                                                    │  │
  │  │  {                                                                 │  │
  │  │    "template": "# Specification\n\n## ADDED Requirements...",      │  │
  │  │    "dependencies": [{"id": "proposal", "path": "...", "done": true}│  │
  │  │    "unlocks": ["tasks"]                                            │  │
  │  │  }                                                                 │  │
  │  └────────────────────────────────────────────────────────────────────┘  │
  │                                                                          │
  │  Step 3: Read dependencies → Create ONE artifact → Show what's unlocked  │
  └──────────────────────────────────────────────────────────────────────────┘
```

<a id="iteration-model"></a>
### 反復のモデル

**レガシーなワークフロー** — 反復しにくい:

```
  ┌─────────┐     ┌─────────┐     ┌─────────┐
  │/proposal│ ──► │ /apply  │ ──► │/archive │
  └─────────┘     └─────────┘     └─────────┘
       │               │
       │               ├── "Wait, the design is wrong"
       │               │
       │               ├── Options:
       │               │   • Edit files manually (breaks context)
       │               │   • Abandon and start over
       │               │   • Push through and fix later
       │               │
       │               └── No official "go back" mechanism
       │
       └── Creates ALL artifacts at once
```

**OPSX** — 自然な反復:

```
  /opsx:new ───► /opsx:continue ───► /opsx:apply ───► /opsx:archive
      │                │                  │
      │                │                  ├── "The design is wrong"
      │                │                  │
      │                │                  ▼
      │                │            Just edit design.md
      │                │            and continue!
      │                │                  │
      │                │                  ▼
      │                │         /opsx:apply picks up
      │                │         where you left off
      │                │
      │                └── Creates ONE artifact, shows what's unlocked
      │
      └── Scaffolds change, waits for direction
```

<a id="custom-schemas"></a>
### カスタムスキーマ

スキーマ管理コマンドを使って、カスタムワークフローを作成します。

```bash
# Create a new schema from scratch (interactive)
openspec schema init my-workflow

# Or fork an existing schema as a starting point
openspec schema fork spec-driven my-workflow

# Validate your schema structure
openspec schema validate my-workflow

# See where a schema resolves from (useful for debugging)
openspec schema which my-workflow
```

スキーマは、`openspec/schemas/`（プロジェクトローカルでバージョン管理される）または `~/.local/share/openspec/schemas/`（ユーザーグローバル）に保存されます。

**スキーマの構造:**
```
openspec/schemas/research-first/
├── schema.yaml
└── templates/
    ├── research.md
    ├── proposal.md
    └── tasks.md
```

**schema.yaml の例:**
```yaml
name: research-first
artifacts:
  - id: research        # Added before proposal
    generates: research.md
    requires: []

  - id: proposal
    generates: proposal.md
    requires: [research]  # Now depends on research

  - id: tasks
    generates: tasks.md
    requires: [proposal]
```

**依存関係グラフ:**
```
   research ──► proposal ──► tasks
```

<a id="summary"></a>
### まとめ

| 観点 | レガシー | OPSX |
|--------|----------|------|
| **テンプレート** | ハードコードされた TypeScript | 外部の YAML + Markdown |
| **依存関係** | なし（すべて一度に） | トポロジカルソートを伴う DAG |
| **状態** | フェーズベースのメンタルモデル | ファイルシステム上の存在 |
| **カスタマイズ** | ソースを編集し、再ビルドする | schema.yaml を作成する |
| **反復** | フェーズ固定 | 流動的で、何でも編集できる |
| **エディタ対応** | ツール固有のコンフィギュレーター/アダプター | 単一のスキルディレクトリ |

<a id="schemas"></a>
## スキーマ

スキーマは、どの成果物が存在し、それらがどう依存するかを定義します。現在利用可能なもの:

- **spec-driven**（デフォルト）: proposal → specs → design → tasks

```bash
# List available schemas
openspec schemas

# See all schemas with their resolution sources
openspec schema which --all

# Create a new schema interactively
openspec schema init my-workflow

# Fork an existing schema for customization
openspec schema fork spec-driven my-workflow

# Validate schema structure before use
openspec schema validate my-workflow
```

<a id="tips"></a>
## ヒント

- 変更を決める前に、`/opsx:explore` でアイデアを考え抜く
- 何がしたいかがわかっているときは `/opsx:ff`、探索しているときは `/opsx:continue`
- `/opsx:apply` の間に何かがおかしければ、成果物を修正してから続ける
- タスクの進捗は、`tasks.md` のチェックボックスで追跡される
- 状態はいつでも確認できる: `openspec status --change "name"`

<a id="feedback"></a>
## フィードバック

まだ粗削りです。それは意図的です。何が効果的かを学んでいる最中だからです。

バグを見つけましたか？ アイデアがありますか？ [Discord](https://discord.gg/YctCnvvshC) に参加するか、[GitHub](https://github.com/Fission-AI/openspec/issues) で issue を作成してください。
