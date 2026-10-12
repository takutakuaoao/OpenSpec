<a id="supported-tools"></a>
# 対応ツール

OpenSpec は、多くの AI コーディングアシスタントで動作します。`openspec init` を実行すると、OpenSpec は、有効なプロファイル/ワークフローの選択と提供方式（delivery mode）を使って、選択したツールを設定します。

<a id="how-it-works"></a>
## 仕組み

選択した各ツールに対して、OpenSpec は次をインストールできます。

1. **スキル**（提供方式にスキルが含まれる場合）: `.../skills/openspec-*/SKILL.md`
2. **コマンド**（提供方式にコマンドが含まれる場合）: ツール固有の `opsx-*` コマンドファイル

Codex はスキルのみです。提供方式が `commands` に設定されていても、OpenSpec は Codex 用に `.agents/skills/openspec-*/SKILL.md` をインストールし、Codex のカスタムプロンプトファイルは生成しません。以前の `.codex/skills` パス配下にある、OpenSpec が管理する既存のスキルは、置き換え先が書き込まれた後に整理されます。カスタムファイルや内容が異なるファイルは保持されます。

デフォルトでは、OpenSpec は `core` プロファイルを使用し、次が含まれます。
- `propose`
- `explore`
- `apply`
- `update`
- `sync`
- `archive`

拡張ワークフロー（`new`、`continue`、`ff`、`verify`、`bulk-archive`、`onboard`）は、`openspec config profile` で有効にしてから `openspec update` を実行することで利用できます。

<a id="how-to-invoke"></a>
## 呼び出し方

これらのドキュメントでは、正式な名前として `/opsx:propose` を使用していますが、各ツールは、
OpenSpec が書き込んだファイルを読み込む方法に従って表記します。下の
[ツールのディレクトリリファレンス](#tool-directory-reference) でお使いのツールのコマンドパスを確認し、ここでその形式に対応するものを見つけてください。

| OpenSpec が書き込むコマンドファイル | 入力方法 | ツール |
|------------------------------|----------|-------|
| `.../commands/opsx/<id>.*` — `opsx/` フォルダが名前空間になる | `/opsx:<id>` | Claude Code, CodeBuddy, Crush, Gemini CLI, Lingma, Qoder, ZCode |
| `.../opsx-<id>.*` — ファイル名がそのままコマンドになる | `/opsx-<id>` | コマンドファイルが生成される、その他のすべてのツール（Amazon Q と Devin を除く） |
| `.devin/workflows/opsx-<id>.md` — Devin の2つのエージェントのうち1つだけが読み込む | Devin Desktop では `/opsx-<id>`、Devin Local では `/openspec-<skill>` | Devin Desktop\*\*\*\* |
| `.amazonq/prompts/opsx-<id>.md` — コマンドではなくプロンプト | `@opsx-<id>` | Amazon Q Developer |
| なし — スキルのみ | `/openspec-<skill>` | CodeArts, ForgeCode, Hermes, MiniMax Code, Mistral Vibe, Zed Agent, 共有 `.agents` |
| なし — Kimi Code | `/skill:openspec-<skill>` | Kimi Code |
| なし — Codex CLI | `$openspec-<skill>` | Codex（[`/openspec-<skill>` は認識されません](https://github.com/openai/codex/issues/11817)） |

そのため、`/opsx:propose` は、Cursor では `/opsx-propose`、Amazon Q では `@opsx-propose`、
Codex では `$openspec-propose` になります。

2つの要素は独立して変化するため、行がまとめられません。

- **名前。** 行1と2の違いは、ファイルがコマンドをどう命名するかだけであり、
  `opsx-<id>` / `opsx:<id>` という語幹は、コマンドファイルが生成されるすべてのツールで同じです。
- **ラッパー。** Amazon Q は、ファイルを `@` で呼び出すプロンプトライブラリに読み込みます。
  スキルのみのツールはコマンドファイルを一切生成しないため、最後の3行は*スキル*名を使用します。
  スキル名は [生成されるスキル名](#generated-skill-names) に一覧があり、コマンド ID と1対1では
  対応しません（`/opsx:apply` は `openspec-apply-change` スキルです）。

上記のコマンドパスのパターンは、意図的に拡張子を問わない形（`.*`）にしています。
拡張子はツール側のもの（Gemini CLI は `.toml`、Continue は `.prompt`、
Kiro と GitHub Copilot は `.prompt.md`）であり、一部のツールでは、ピッカーに拡張子付きの名前が表示されます。
拡張子ではなく、ディレクトリの構造に注目してください。

OpenSpec が生成するファイルと、セットアップ後に表示される「Getting started」のヒントには、
選択したツールに適した形式がすでに使われています。そのため、最も手っ取り早いのは
ヒントを読むことです。

<a id="tool-directory-reference"></a>
## ツールのディレクトリリファレンス

| ツール (ID) | スキルのパスパターン | コマンドのパスパターン |
|-----------|---------------------|----------------------|
| Amazon Q Developer (`amazon-q`) | `.amazonq/skills/openspec-*/SKILL.md` | `.amazonq/prompts/opsx-<id>.md` |
| Antigravity (`antigravity`) | `.agent/skills/openspec-*/SKILL.md` | `.agent/workflows/opsx-<id>.md` |
| Auggie (`auggie`) | `.augment/skills/openspec-*/SKILL.md` | `.augment/commands/opsx-<id>.md` |
| IBM Bob Shell (`bob`) | `.bob/skills/openspec-*/SKILL.md` | `.bob/commands/opsx-<id>.md` |
| Claude Code (`claude`) | `.claude/skills/openspec-*/SKILL.md` | `.claude/commands/opsx/<id>.md` |
| Cline (`cline`) | `.cline/skills/openspec-*/SKILL.md` | `.clinerules/workflows/opsx-<id>.md` |
| Command Code (`command-code`) | `.commandcode/skills/openspec-*/SKILL.md` | `.commandcode/commands/opsx-<id>.md` |
| CodeArts (`codeartsagent`) | `.codeartsdoer/skills/openspec-*/SKILL.md` | 生成されない（コマンドアダプターなし。スキルベースの `/openspec-*` 呼び出しを使用） |
| CodeBuddy (`codebuddy`) | `.codebuddy/skills/openspec-*/SKILL.md` | `.codebuddy/commands/opsx/<id>.md` |
| Codex (`codex`) | `.agents/skills/openspec-*/SKILL.md` | 生成されない（スキルのみ。`$openspec-*` を使用） |
| Devin Desktop（旧 Windsurf） (`devin`) | `.devin/skills/openspec-*/SKILL.md` | `.devin/workflows/opsx-<id>.md`\*\*\*\* |
| ForgeCode (`forgecode`) | `.forge/skills/openspec-*/SKILL.md` | 生成されない（コマンドアダプターなし。スキルベースの `/openspec-*` 呼び出しを使用） |
| Continue (`continue`) | `.continue/skills/openspec-*/SKILL.md` | `.continue/prompts/opsx-<id>.prompt` |
| CoStrict (`costrict`) | `.cospec/skills/openspec-*/SKILL.md` | `.cospec/openspec/commands/opsx-<id>.md` |
| Crush (`crush`) | `.crush/skills/openspec-*/SKILL.md` | `.crush/commands/opsx/<id>.md` |
| Cursor (`cursor`) | `.cursor/skills/openspec-*/SKILL.md` | `.cursor/commands/opsx-<id>.md` |
| Factory Droid (`factory`) | `.factory/skills/openspec-*/SKILL.md` | `.factory/commands/opsx-<id>.md` |
| Gemini CLI (`gemini`) | `.gemini/skills/openspec-*/SKILL.md` | `.gemini/commands/opsx/<id>.toml` |
| GitHub Copilot (`github-copilot`) | `.github/skills/openspec-*/SKILL.md` | `.github/prompts/opsx-<id>.prompt.md`\*\* |
| Hermes Agent (`hermes`) | `.hermes/skills/openspec-*/SKILL.md`\*\*\* | 生成されない（コマンドアダプターなし。スキルベースの `/openspec-*` 呼び出しを使用） |
| iFlow (`iflow`) | `.iflow/skills/openspec-*/SKILL.md` | `.iflow/commands/opsx-<id>.md` |
| Junie (`junie`) | `.junie/skills/openspec-*/SKILL.md` | `.junie/commands/opsx-<id>.md` |
| Kilo Code (`kilocode`) | `.kilocode/skills/openspec-*/SKILL.md` | `.kilo/command/opsx-<id>.md` |
| Kimi Code (`kimi`) | `.kimi-code/skills/openspec-*/SKILL.md` | 生成されない（コマンドアダプターなし。スキルベースの `/skill:openspec-*` 呼び出しを使用） |
| Kiro (`kiro`) | `.kiro/skills/openspec-*/SKILL.md` | `.kiro/prompts/opsx-<id>.prompt.md` |
| Lingma (`lingma`) | `.lingma/skills/openspec-*/SKILL.md` | `.lingma/commands/opsx/<id>.md` |
| MiniMax Code (`minimax-code`) | `~/.minimax/skills/openspec-*/SKILL.md` | 生成されない（コマンドアダプターなし。MiniMax Code のスキルを使用） |
| Mistral Vibe (`vibe`) | `.vibe/skills/openspec-*/SKILL.md` | 生成されない（コマンドアダプターなし。スキルベースの `/openspec-*` 呼び出しを使用） |
| Oh My Pi (`oh-my-pi`) | `.omp/skills/openspec-*/SKILL.md` | `.omp/commands/opsx-<id>.md` |
| OpenCode (`opencode`) | `.opencode/skills/openspec-*/SKILL.md` | `.opencode/commands/opsx-<id>.md` |
| Pi (`pi`) | `.pi/skills/openspec-*/SKILL.md` | `.pi/prompts/opsx-<id>.md` |
| SourceCraft Code Assistant for VS Code (`codeassistant`) | `.codeassistant/skills/openspec-*/SKILL.md` | `.codeassistant/commands/opsx-<id>.md` |
| Qoder (`qoder`) | `.qoder/skills/openspec-*/SKILL.md` | `.qoder/commands/opsx/<id>.md` |
| Qwen Code (`qwen`) | `.qwen/skills/openspec-*/SKILL.md` | `.qwen/commands/opsx-<id>.md` |
| [Rovo Dev CLI](https://support.atlassian.com/rovo/docs/use-rovo-dev-cli/) (`rovodev`) | `.rovodev/skills/openspec-*/SKILL.md` | 生成されない。Rovo にはスラッシュコマンドの仕組みがなく、スキルを自動的に、またはプロンプト（例: 「openspec-propose スキルを使って」）で照合します。`/skills` はスキルの管理のみを行います。生成されるコンテンツは、スキルを名前で参照し、`/openspec-*` コマンドとして参照することはありません。 |
| [Zoo Code](https://github.com/Zoo-Code-Org/Zoo-Code) (`roocode`) | `.roo/skills/openspec-*/SKILL.md` | `.roo/commands/opsx-<id>.md` |
| Trae (`trae`) | `.trae/skills/openspec-*/SKILL.md` | `.trae/commands/opsx-<id>.md` |
| [Zed Agent](https://zed.dev/docs/ai/skills) (`zed`) | `.agents/skills/openspec-*/SKILL.md` | 生成されない（スキルのみ。`/openspec-*` または `@openspec-*` を使用） |
| ZCode (`zcode`) | `.zcode/skills/openspec-*/SKILL.md` | `.zcode/commands/opsx/<id>.md` |
| 共有 `.agents` スキル (`agents`) | `.agents/skills/openspec-*/SKILL.md` | 生成されない（コマンドアダプターなし。スキルベースの `/openspec-*` 呼び出しを使用） |

\*\* GitHub Copilot のプロンプトファイルは、IDE 拡張機能（VS Code、JetBrains、Visual Studio）では、カスタムスラッシュコマンドとして認識されます。Copilot CLI は現在、`.github/prompts/*.prompt.md` を直接利用しません。`github-copilot` を選択すると、GitHub がホストする**クラウドコーディングエージェント**もセットアップできます。以下の [GitHub Copilot クラウドコーディングエージェント](#github-copilot-cloud-coding-agent) を参照してください。

\*\*\* Hermes は、デフォルトでは `~/.hermes/skills/` からスキルを読み込みます。プロジェクトローカルの OpenSpec スキルを使うには、プロジェクトの `.hermes/skills/` ディレクトリを `~/.hermes/config.yaml` の `skills.external_dirs` に追加してください。そうすると Hermes は、`/openspec-propose` のような、ユーザー向けのスラッシュ呼び出しでスキルを公開します。

\*\*\*\* Windsurf は 2026年6月2日に [Devin Desktop に名称変更](https://docs.devin.ai/desktop/devin-desktop-faq) され、設定ディレクトリも移動しました。`.devin/` が推奨される読み取り + 書き込みの場所であり、`.windsurf/` はレガシーな読み取り専用のフォールバックです。OpenSpec はこの名称変更に従っており、ツール ID は `devin` ですが、`--tools windsurf` も引き続きこれに解決されるため、既存のセットアップスクリプトはそのまま動作します。OpenSpec のファイルが `.windsurf/` に残っているプロジェクトでは、次回の `openspec update` で移動が提案されます。拒否した場合はそのまま残り、あなたが自分で書いたファイルに手が加えられることはありません。ワークフローはファイル名で呼び出されるため、`.devin/workflows/opsx-apply.md` は `/opsx-apply` になります。[Devin Local エージェントはワークフローに対応しておらず](https://docs.devin.ai/desktop/devin-local)、スキルのみに対応しています。また `.windsurf/` はまったく読み込みません。そのため OpenSpec は、Devin のスキルを書き込む際には、その本文と getting-started のヒントを、両方のエージェントで動作する `/openspec-*` のスキル呼び出しにしています。コマンドのみの提供方式では、スキルは書き込まれず、どちらも `/opsx-*` にフォールバックします。

SourceCraft Code Assistant への対応は、その VS Code 拡張機能が対象です。その[カスタムコマンド](https://sourcecraft.dev/portal/docs/en/code-assistant/operations/agent/slash-commands)と[スキル](https://sourcecraft.dev/portal/docs/ru/code-assistant/operations/agent/skills)は、VS Code でのみ利用できます。この連携は、SourceCraft の Web 版や JetBrains を設定しません。

スキルのみの提供方式では、Code Assistant に、アイデアを添えて `openspec-propose` スキルを使うよう依頼してください。スキルはリクエストの照合によって有効化され、OpenSpec はこのツール用の `/openspec-*` コマンドを生成しません。

MiniMax Code は、グローバルなスキルのみの連携です。OpenSpec が書き込むのは、
`~/.minimax/skills/` 配下の `openspec-*` ディレクトリのみで、リポジトリローカルの
`.minimax` や `.mavis` ディレクトリは作成しません。コマンドのみの提供方式では、
既存のグローバルな MiniMax Code のスキルには手を加えないため、あるプロジェクトの
提供方式の設定が、別のプロジェクトが使用しているスキルを削除することはありません。

<a id="github-copilot-cloud-coding-agent"></a>
### GitHub Copilot クラウドコーディングエージェント

GitHub の [Copilot コーディングエージェント](https://docs.github.com/en/copilot/using-github-copilot/coding-agent) は、エディタ内の Copilot とは別に、GitHub 上の GitHub Actions 環境で実行されます。OpenSpec は、次の2つのファイルを生成することで、このエージェントが OpenSpec CLI を使えるようセットアップできます。

- `.github/workflows/copilot-setup-steps.yml` — エージェントの環境に `@fission-ai/openspec` をインストールする
- `.github/agents/openspec.agent.md` — OpenSpec の操作方法をエージェントに伝える

これは GitHub Actions のワークフローをリポジトリに書き込むため、**オプトイン**です。

| 方法 | 動作 |
|-----|----------|
| `openspec init`（対話型） | クラウド用ファイルをセットアップするか尋ねる。デフォルトは**いいえ**。 |
| `openspec init --copilot-cloud` | プロンプトなしでセットアップする（スクリプト/CI 向け）。 |
| `openspec init --no-copilot-cloud` | プロンプトなしでスキップし、以前に生成されたファイルがあれば削除する。 |
| `openspec update` | 確認しない。オプトインしている場合（またはプロジェクトにすでにファイルがある場合）のみ、ファイルを更新する。オプトアウトしている場合は、OpenSpec が管理するクラウド用ファイルを削除する。 |

選択は `openspec/config.yaml` に `githubCopilot.cloudAgent: true|false` として保存されるため、非対話型の更新でもそれが尊重されます。OpenSpec は、自身が内容を生成したファイルだけを書き込みまたは削除します。`copilot-setup-steps.yml` や `openspec.agent.md` をカスタマイズしている場合や、すでに自分のものがある場合は、そのまま残されます（`init`/`update` がその旨を知らせます）。

<a id="when-to-pick-the-shared-agents-target"></a>
### 共有 `.agents` ターゲットを選ぶタイミング

`agents` はベンダー中立のオプションです。ツール固有のディレクトリではなく、多くのエージェントツールが
読み込む共有ルートである `.agents/skills/` にスキルを書き込みます。

| 状況 | 選択するもの |
|-----------|------|
| お使いのツールが上の表に専用の行を持っている | そのツール自身の ID — スラッシュコマンドに対応している場合はそれを含め、そのツールの連携が得られる |
| 1つのリポジトリで複数のエージェントが、すべて `.agents/skills` を読み込む | `agents` — ツールごとに1つではなく、1つのスキルツリーで済む |
| お使いのツールがまだ一覧にないが、`.agents/skills` を読み込む | `agents` |

ツール固有の ID と一緒に選択しても問題ありません。通常、それぞれが自身のルートに
書き込みます。Codex と Zed Agent は例外で、同じ正規の `.agents` ルートを使用するためです。
Codex が Zed または `agents` と一緒に選択された場合、OpenSpec は Codex 主導の1つのツリーを維持します。
その引き継ぎには、Codex 向けの `$openspec-*` と、他のエージェント向けの
`/openspec-*` の両方が含まれるため、`--tools all` や既存のマルチエージェントの
セットアップは、2つの書き込み元が同じファイルを上書きし合うことなく、そのまま機能します。
OpenSpec は、プロジェクトに `.agents/skills/` ディレクトリが存在する場合にも、これを自動的に提案します。
`.agents/` だけでは不十分です。ツールはこのルートをルールやサブエージェントの定義にも使うためです。
`.agents` は `.agent` ではないことに注意してください。単数形のディレクトリは Antigravity のものです。

知っておくべき点が2つあります。

- **スキルのみ。** コマンドアダプターは存在しないため、`opsx-*` コマンドファイルは
  書き込まれません。コマンドを含む提供方式では、`openspec init` は `agents` を
  `Commands skipped for: … (no adapter)` の下に報告するツールとして一覧に挙げます。
  ワークフローはスキル名で呼び出します。`.agents/skills` を読み込むほとんどの
  アシスタントでは `/openspec-propose` と表記され、これは OpenSpec のセットアップヒントが
  表示する形式です。このターゲットはベンダー中立なので、アシスタントが別の形式を使う場合は、
  そのアシスタント自身のドキュメントを確認してください。
- **`AGENTS.md` は作成も編集もされません。** 対象は `.agents/` ディレクトリです。
  ルートの `AGENTS.md` に古いバージョンの OpenSpec のマーカーブロックが残っている場合、
  `openspec update` がそれを取り除きます。[移行ガイド](migration-guide.md) を参照してください。

ここでの Zed への対応は、組み込みの Zed Agent を対象としています。Zed の External Agents と
Terminal Threads は、それぞれ独自の連携を使用します。Agent Skills には
[Zed v1.4.2](https://github.com/zed-industries/zed/releases/tag/v1.4.2) 以降が必要です。
プロジェクトローカルのスキルは、[信頼を付与](https://zed.dev/docs/worktree-trust)するまで、
信頼されていないワークツリーでは利用できません。

`.agents/skills/` は Codex、Zed Agent、ベンダー中立ターゲットで共有されるため、
OpenSpec がそこで何を管理するかを知っておく価値があります。
OpenSpec は、選択したワークフローの `openspec-*` スキルディレクトリと、その共有ツリーを
Codex、Zed Agent、ベンダー中立ターゲットのどれがレンダリングしたかを記録する
`.openspec-target` マーカーのみを、書き込み、更新、削除します。そのディレクトリ内の
それ以外のものには手を加えません。`openspec-*` という名前とマーカーは OpenSpec のものとして扱ってください。
それらの内部の編集は、他のすべてのツールと同様に、次回の `openspec update` で置き換えられます。

マーカー導入前のプロジェクトでは、OpenSpec は、管理対象のスキルの参照から所有者を推測します。
`$openspec-*` は Codex を、`/openspec-*` はベンダー中立ターゲットを意味します。
レガシーな `.codex/skills` と並んで存在する汎用の正規ツリーは、古いデュアルターゲットのインストールとして扱われ、
互換性のある共有ツリーに統合されます。

`openspec update` も、この所有関係を尊重します。プロジェクトが `.agents` をベンダー中立ターゲットとして
所有していて、残っている Codex のインストールが、散在するプロンプトファイルからのみ検出された場合、
update は確立された `agents` ツリーをそのまま残し、Codex の構文で書き換えることはしません。
また、それらのレガシーなプロンプトファイルは削除せずに保持します。共有ツリーを Codex に
引き渡すには、`openspec init --tools codex` を明示的に実行してください。

<a id="non-interactive-setup"></a>
## 非対話型セットアップ

CI/CD やスクリプトによるセットアップには、`--tools`（および任意で `--profile`）を使用します。

```bash
# Configure specific tools
openspec init --tools claude,cursor

# Configure all supported tools
openspec init --tools all

# Skip tool configuration
openspec init --tools none

# Override profile for this init run
openspec init --profile core
```

**利用可能なツール ID（`--tools`）** — `windsurf` も `devin` のエイリアスとして受け付けられます: `amazon-q`, `antigravity`, `auggie`, `bob`, `claude`, `cline`, `command-code`, `codeartsagent`, `codex`, `devin`, `forgecode`, `codebuddy`, `continue`, `costrict`, `crush`, `cursor`, `factory`, `gemini`, `github-copilot`, `hermes`, `iflow`, `junie`, `kilocode`, `kimi`, `kiro`, `lingma`, `minimax-code`, `vibe`, `oh-my-pi`, `opencode`, `pi`, `qoder`, `qwen`, `roocode`, `codeassistant`, `trae`, `zed`, `zcode`, `agents`

<a id="workflow-dependent-installation"></a>
## ワークフローに依存するインストール

OpenSpec は、選択したワークフローに基づいて、ワークフローの成果物をインストールします。

- **コアプロファイル（デフォルト）:** `propose`, `explore`, `apply`, `update`, `sync`, `archive`
- **カスタム選択:** すべてのワークフロー ID の任意のサブセット:
  `propose`, `explore`, `new`, `continue`, `apply`, `update`, `ff`, `sync`, `archive`, `bulk-archive`, `verify`, `onboard`

つまり、スキルやコマンドの数は、固定ではなく、プロファイルと提供方式に依存します。

<a id="generated-skill-names"></a>
## 生成されるスキル名

プロファイル/ワークフローの設定で選択された場合、OpenSpec は次のスキルを生成します。

- `openspec-propose`
- `openspec-explore`
- `openspec-new-change`
- `openspec-continue-change`
- `openspec-apply-change`
- `openspec-update-change`
- `openspec-ff-change`
- `openspec-sync-specs`
- `openspec-archive-change`
- `openspec-bulk-archive-change`
- `openspec-verify-change`
- `openspec-onboard`

コマンドの動作については [コマンド](commands.md) を、`init`/`update` のオプションについては [CLI](cli.md) を参照してください。

<a id="related"></a>
## 関連項目

- [CLI リファレンス](cli.md) — ターミナルコマンド
- [コマンド](commands.md) — スラッシュコマンドとスキル
- [はじめに](getting-started.md) — 初回のセットアップ
