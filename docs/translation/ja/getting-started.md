<a id="getting-started"></a>
# はじめに

このガイドでは、OpenSpec をインストールして初期化した後の動作の仕組みを説明します。インストール手順については、[メインの README](../../../README.md#quick-start) または [インストールガイド](installation.md) を参照してください。ドキュメント全体が初めての方は、[ドキュメントのホーム](README.md) に全体の地図があります。

> **これらのコマンドはどこに入力するの？** 場所は2つあり、これらを取り違えるのが、初期に最もよくあるつまずきです。
>
> - `openspec ...` コマンド（`openspec init` など）は、**ターミナル**で実行します。
> - `/opsx:...` コマンド（`/opsx:propose` など）は、**AI アシスタントのチャット**で実行します。コードを書くよう頼むのと同じ入力欄です。
>
> 開始すべき別の「インタラクティブモード」はありません。チャットにスラッシュコマンドを入力するだけで、アシスタントが引き継ぎます。詳しい説明: [コマンドの仕組み](how-commands-work.md)。

<a id="your-first-five-minutes"></a>
## 最初の5分間

ループ全体を、各ステップが行われる場所のラベル付きで示します。

```text
TERMINAL   $ npm install -g @fission-ai/openspec@latest
TERMINAL   $ cd your-project && openspec init
AI CHAT      /opsx:explore                    （任意: 先に考えを整理する）
AI CHAT      /opsx:propose add-dark-mode      （AI が計画を下書きし、あなたがレビューする）
AI CHAT      /opsx:apply                      （AI が実装する）
AI CHAT      /opsx:archive                    （仕様が更新され、変更が保管される）
```

セットアップのためのターミナル操作が2つあり、あとはチャットが中心になります。このガイドの残りでは、各ステップの内容と、目にするものを詳しく説明します。

**ターミナルでの作業を自分でやりたくない？** [セットアップ用プロンプト](installation.md#install-with-your-ai-assistant) をアシスタントに貼り付ければ、2行とも処理し、作成されたものを報告してくれます。

> **まだ何を作るか決まっていない？ `/opsx:explore` から始めましょう。** これは、リスクなしで使える思考パートナーで、コードベースを読み、選択肢を比較し、曖昧なアイデアを、コードを書く前に具体的な計画へと磨き上げます。全体像がはっきりしたら、`/opsx:propose` に引き継ぎます。曖昧なままだと自信満々に間違ったものを作ってしまう AI と付き合ううえで、これは最良の習慣です。[探索ガイド](explore.md) を参照してください。

<a id="how-it-works"></a>
## 仕組み

OpenSpec は、あなたと AI コーディングアシスタントが、コードを書く前に何を作るかを合意するのを助けます。

**デフォルトのクイックパス（core プロファイル）:**

```text
/opsx:explore ──► /opsx:propose ──► /opsx:apply ──► /opsx:sync ──► /opsx:archive
   （任意）
```

何をするか考えているときは `/opsx:explore` から始め、すでに決まっているときは直接 `/opsx:propose` に進みます。探索はデフォルトのプロファイルに含まれているため、必要なときにいつでも使えます。

**拡張パス（ワークフローをカスタム選択）:**

```text
/opsx:new ──► /opsx:ff or /opsx:continue ──► /opsx:apply ──► /opsx:verify ──► /opsx:archive
```

デフォルトのグローバルプロファイルは `core` で、`propose`、`explore`、`apply`、`update`、`sync`、`archive` が含まれます。`openspec config profile` を実行してから `openspec update` を実行すると、拡張ワークフローのコマンドを有効にできます。

<a id="what-openspec-creates"></a>
## OpenSpec が作成するもの

`openspec init` を実行すると、プロジェクトは次の構造になります。

```
openspec/
├── specs/              # 信頼できる唯一の情報源（システムの振る舞い）
│   └── <domain>/
│       └── spec.md
├── changes/            # 提案された更新（変更ごとに1フォルダ）
│   └── <change-name>/
│       ├── proposal.md
│       ├── design.md
│       ├── tasks.md
│       └── specs/      # デルタ仕様（何が変わるか）
│           └── <domain>/
│               └── spec.md
└── config.yaml         # プロジェクト設定（任意）
```

**2つの重要なディレクトリ:**

- **`specs/`** - 信頼できる唯一の情報源。これらの仕様は、システムが現在どう振る舞うかを記述します。ドメインごとに整理されます（例: `specs/auth/`、`specs/payments/`）。

- **`changes/`** - 提案された変更。各変更は、関連するすべての成果物を含む専用のフォルダを持ちます。変更が完了すると、その仕様はメインの `specs/` ディレクトリにマージされます。

<a id="understanding-artifacts"></a>
## 成果物を理解する

各変更フォルダには、作業を導く成果物が含まれています。

| 成果物 | 目的 |
|----------|---------|
| `proposal.md` | 「なぜ」と「何を」- 意図、スコープ、アプローチを記述する |
| `specs/` | ADDED/MODIFIED/REMOVED の要件を示すデルタ仕様 |
| `design.md` | 「どのように」- 技術的アプローチとアーキテクチャ上の判断 |
| `tasks.md` | チェックボックス付きの実装チェックリスト |

**成果物は互いの上に積み上がります:**

```
proposal ──► specs ──► design ──► tasks ──► implement
   ▲           ▲          ▲                    │
   └───────────┴──────────┴────────────────────┘
            学びに応じて更新する
```

実装中にさらに学んだことがあれば、いつでも戻って以前の成果物を洗練できます。

<a id="how-delta-specs-work"></a>
## デルタ仕様の仕組み

デルタ仕様は OpenSpec の中心的な概念です。現在の仕様に対して何が変わるかを示します。

<a id="the-format"></a>
### フォーマット

デルタ仕様は、変更の種類をセクションで示します。

```markdown
# Delta for Auth

## ADDED Requirements

### Requirement: Two-Factor Authentication
The system MUST require a second factor during login.

#### Scenario: OTP required
- GIVEN a user with 2FA enabled
- WHEN the user submits valid credentials
- THEN an OTP challenge is presented

## MODIFIED Requirements

### Requirement: Session Timeout
The system SHALL expire sessions after 30 minutes of inactivity.
(Previously: 60 minutes)

#### Scenario: Idle timeout
- GIVEN an authenticated session
- WHEN 30 minutes pass without activity
- THEN the session is invalidated

## REMOVED Requirements

### Requirement: Remember Me
(Deprecated in favor of 2FA)
```

<a id="what-happens-on-archive"></a>
### アーカイブ時に起こること

変更をアーカイブすると、次のようになります。

1. **ADDED** の要件が、メインの仕様に追加される
2. **MODIFIED** の要件が、既存のバージョンを置き換える
3. **REMOVED** の要件が、メインの仕様から削除される

変更フォルダは、監査履歴として `openspec/changes/archive/` に移動されます。

<a id="example-your-first-change"></a>
## 例: 最初の変更

アプリケーションにダークモードを追加する流れを見てみましょう。

<a id="1-start-the-change-default"></a>
### 1. 変更を開始する（デフォルト）

```text
You: /opsx:propose add-dark-mode

AI:  openspec/changes/add-dark-mode/ を作成しました
     ✓ proposal.md — 取り組む理由と、変更内容
     ✓ specs/       — 要件とシナリオ
     ✓ design.md    — 技術的アプローチ
     ✓ tasks.md     — 実装チェックリスト
     実装の準備ができました！
```

拡張ワークフローのプロファイルを有効にしている場合は、2段階で行うこともできます。`/opsx:new` の後に `/opsx:ff`（または段階的に進める `/opsx:continue`）を実行します。

<a id="2-what-gets-created"></a>
### 2. 作成されるもの

**proposal.md** - 意図を記述します。

```markdown
# Proposal: Add Dark Mode

## Intent
Users have requested a dark mode option to reduce eye strain
during nighttime usage.

## Scope
- Add theme toggle in settings
- Support system preference detection
- Persist preference in localStorage

## Approach
Use CSS custom properties for theming with a React context
for state management.
```

**specs/ui/spec.md** - 新しい要件を示すデルタ:

```markdown
# Delta for UI

## ADDED Requirements

### Requirement: Theme Selection
The system SHALL allow users to choose between light and dark themes.

#### Scenario: Manual toggle
- GIVEN a user on any page
- WHEN the user clicks the theme toggle
- THEN the theme switches immediately
- AND the preference persists across sessions

#### Scenario: System preference
- GIVEN a user with no saved preference
- WHEN the application loads
- THEN the system's preferred color scheme is used
```

**tasks.md** - 実装チェックリスト:

```markdown
# Tasks

## 1. Theme Infrastructure
- [ ] 1.1 Create ThemeContext with light/dark state
- [ ] 1.2 Add CSS custom properties for colors
- [ ] 1.3 Implement localStorage persistence

## 2. UI Components
- [ ] 2.1 Create ThemeToggle component
- [ ] 2.2 Add toggle to settings page
- [ ] 2.3 Update Header to include quick toggle

## 3. Styling
- [ ] 3.1 Define dark theme color palette
- [ ] 3.2 Update components to use CSS variables
```

<a id="3-implement"></a>
### 3. 実装する

```
You: /opsx:apply

AI:  タスクを進めています...
     ✓ 1.1 light/dark の状態を持つ ThemeContext を作成
     ✓ 1.2 globals.css に CSS カスタムプロパティを追加
     ✓ 1.3 localStorage への永続化を実装
     ✓ 2.1 ThemeToggle コンポーネントを作成
     ...
     すべてのタスクが完了しました！
```

実装中に設計の調整が必要だとわかったら、成果物を更新して続行するだけです。

<a id="4-archive"></a>
### 4. アーカイブする

```
You: /opsx:archive

AI:  add-dark-mode をアーカイブしています...
     ✓ 仕様を openspec/specs/ui/spec.md にマージしました
     ✓ openspec/changes/archive/2025-01-24-add-dark-mode/ に移動しました
     完了です！次の機能に進む準備ができました。
```

デルタ仕様はメインの仕様の一部となり、システムがどう動作するかを記述するようになりました。

<a id="verifying-and-reviewing"></a>
## 検証とレビュー

CLI を使って、変更の状況を確認できます。

```bash
# List active changes
openspec list

# View change details
openspec show add-dark-mode

# Validate spec formatting
openspec validate add-dark-mode

# Interactive dashboard
openspec view
```

<a id="next-steps"></a>
## 次のステップ

- [まず探索する](explore.md) - 決める前に `/opsx:explore` でアイデアを整理する
- [変更のレビュー](reviewing-changes.md) - コードの前に、AI が下書きした計画で何を確認すべきか
- [良い仕様の書き方](writing-specs.md) - 優れた要件とシナリオの姿
- [既存プロジェクトでの OpenSpec の利用](existing-projects.md) - 大規模なブラウンフィールドのコードベースで始める
- [変更の編集と反復](editing-changes.md) - 成果物の更新、前の段階への戻り、手動編集の整合
- [コア概念の概要](overview.md) - 全体の考え方を1ページで
- [例とレシピ](examples.md) - 最初から最後までの実際の変更
- [ワークフロー](workflows.md) - よくあるパターンと、各コマンドを使うべき場面
- [コマンド](commands.md) - すべてのスラッシュコマンドの完全なリファレンス
- [概念](concepts.md) - 仕様、変更、スキーマのより深い理解
- [カスタマイズ](customization.md) - OpenSpec を自分好みに調整する
- [ストア](stores-beta/user-guide.md) - 計画がリポジトリやチームにまたがる場合は、専用のリポジトリで管理する（ベータ）
- [FAQ](faq.md) と [トラブルシューティング](troubleshooting.md) - 行き詰まったとき
