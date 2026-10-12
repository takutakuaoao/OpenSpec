<a id="cli-reference"></a>
# CLI リファレンス

OpenSpec CLI（`openspec`）は、プロジェクトのセットアップ、検証、ステータスの確認、管理のためのターミナルコマンドを提供します。これらのコマンドは、[コマンド](commands.md) に記載されている AI のスラッシュコマンド（`/opsx:propose` など）を補完するものです。

<a id="summary"></a>
## 概要

| カテゴリ | コマンド | 目的 |
|----------|----------|---------|
| **セットアップ** | `init`, `update` | プロジェクトで OpenSpec を初期化・更新する |
| **ストア（独立した OpenSpec リポジトリ）** | `store setup`, `store register`, `store unregister`, `store remove`, `store list`, `store doctor` | ストア（登録した独立した OpenSpec リポジトリ）を管理する |
| **ヘルス** | `doctor` | 解決されたルートの関係性の健全性を報告する |
| **作業コンテキスト** | `context` | 作業セット（ルート + 参照しているストア）を組み立てる |
| **個人用ワークセット** | `workset create`, `workset list`, `workset open`, `workset remove` | 個人用のローカルな作業ビューを保持し、ツールで開く |
| **閲覧** | `list`, `view`, `show` | 変更と仕様を探索する |
| **検証** | `validate` | 変更と仕様の問題をチェックする |
| **ライフサイクル** | `archive` | 完了した変更を確定する |
| **ワークフロー** | `new change`, `status`, `instructions`, `templates`, `schemas` | 成果物駆動のワークフローを支援する |
| **スキーマ** | `schema init`, `schema fork`, `schema validate`, `schema which` | カスタムワークフローを作成・管理する |
| **設定** | `config` | 設定を表示・変更する |
| **ユーティリティ** | `feedback`, `completion` | フィードバックとシェル連携 |

---

<a id="human-vs-agent-commands"></a>
## 人間用コマンドとエージェント用コマンド

ほとんどの CLI コマンドは、ターミナルでの**人間による利用**を想定して設計されています。一部のコマンドは、JSON 出力による**エージェント/スクリプトからの利用**にも対応しています。

<a id="human-only-commands"></a>
### 人間専用のコマンド

これらのコマンドは対話型で、ターミナルでの利用を想定しています。

| コマンド | 目的 |
|---------|---------|
| `openspec init` | プロジェクトを初期化する（対話式プロンプト） |
| `openspec view` | インタラクティブなダッシュボード |
| `openspec workset open <name>` | 保存したワークセットを開く（エディタウィンドウまたはターミナルのエージェントセッション） |
| `openspec config edit` | エディタで設定を開く |
| `openspec feedback` | GitHub 経由でフィードバックを送信する |
| `openspec completion install` | シェル補完をインストールする |

<a id="agent-compatible-commands"></a>
### エージェント互換のコマンド

これらのコマンドは、AI エージェントやスクリプトがプログラムから利用できるよう、`--json` 出力に対応しています。

| コマンド | 人間による利用 | エージェントによる利用 |
|---------|-----------|-----------|
| `openspec list` | 変更/仕様を閲覧する | 構造化データ用の `--json` |
| `openspec show <item>` | 内容を読む | 解析用の `--json` |
| `openspec validate` | 問題をチェックする | 一括検証用の `--all --json` |
| `openspec status` | 成果物の進捗を確認する | 構造化されたステータス用の `--json` |
| `openspec instructions` | 次のステップを取得する | エージェント向け指示用の `--json` |
| `openspec templates` | テンプレートのパスを調べる | パス解決用の `--json` |
| `openspec schemas` | 利用可能なスキーマを一覧表示する | スキーマの検出用の `--json`。登録済みのルートを選択するには `--store <id>` |
| `openspec store setup <id>` | ローカルストアを作成して登録する | 構造化されたセットアップ出力用に、明示的な入力と併用する `--json` |
| `openspec store register <path>` | 既存のストアを登録する | 構造化された登録出力用の `--json` |
| `openspec store unregister <id>` | ローカルストアの登録を解除する | 構造化されたクリーンアップ出力用の `--json` |
| `openspec store remove <id>` | 登録済みのローカルストアのフォルダを削除する | 非対話型の削除用の `--yes --json` |
| `openspec store list` | 登録済みのストアを閲覧する | 構造化された登録情報用の `--json` |
| `openspec store doctor` | ローカルストアのセットアップをチェックする | 構造化された診断用の `--json` |
| `openspec new change <id>` | リポジトリローカルの変更の骨組みを作成する | `--json`。さらに、登録済みのストアを OpenSpec のルートとして使用する `--store <id>` |
| `openspec workset create [name]` | 個人用の作業ビューを構成する | 非対話型の構成用の `--member <path> --json` |
| `openspec workset list` | 保存したワークセットを閲覧する | 構造化されたビュー用の `--json` |
| `openspec workset remove <name>` | 保存したビューを削除する | 非対話型の削除用の `--yes --json` |

---

<a id="global-options"></a>
## グローバルオプション

これらのオプションは、すべてのコマンドで使用できます。

| オプション | 説明 |
|--------|-------------|
| `--version`, `-V` | バージョン番号を表示する |
| `--no-color` | カラー出力を無効にする |
| `--help`, `-h` | コマンドのヘルプを表示する |

---

<a id="setup-commands"></a>
## セットアップコマンド

<a id="openspec-init"></a>
### `openspec init`

プロジェクトで OpenSpec を初期化します。フォルダ構造を作成し、AI ツールとの連携を設定します。

デフォルトの動作では、グローバル設定のデフォルト（プロファイル `core`、提供方式 `both`、ワークフロー `propose, explore, apply, update, sync, archive`）が使用されます。

```
openspec init [path] [options]
```

`--language <language>` を使うと、新しいプロジェクトの `openspec/config.yaml` に
言語の指示を追加できます。既存のプロジェクトでは、OpenSpec がプロジェクト固有のガイダンスを
上書きしないよう、設定の `context` フィールドを編集してください。

**引数:**

| 引数 | 必須 | 説明 |
|----------|----------|-------------|
| `path` | いいえ | 対象ディレクトリ（デフォルト: カレントディレクトリ） |

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--tools <list>` | AI ツールを非対話型で設定する。`all`、`none`、またはカンマ区切りのリストを使用 |
| `--language <language>` | 新しい設定を作成するときに、この言語で成果物を書く |
| `--force` | 確認なしでレガシーなファイルを自動的にクリーンアップする |
| `--profile <profile>` | この init の実行に限り、グローバルプロファイルを上書きする（`core` または `custom`） |
| `--no-animation` | アニメーションではなく、静的なウェルカム画面を表示する |
| `--copilot-cloud` | プロンプトなしで GitHub Copilot の[クラウドコーディングエージェント用ファイル](supported-tools.md#github-copilot-cloud-coding-agent)をセットアップする |
| `--no-copilot-cloud` | プロンプトなしで GitHub Copilot のクラウドコーディングエージェント用ファイルをスキップする |

`--profile custom` は、グローバル設定（`openspec config profile`）で現在選択されているワークフローを使用します。

ウェルカムアニメーションは、環境変数 `OPENSPEC_NO_ANIMATION` が設定されている場合（空を含む任意の値）、`NO_COLOR` が空でない値に設定されている場合、または OS のモーション低減設定が有効な場合（macOS の「視差効果を減らす」、GNOME のアニメーション無効化）にもスキップされます。

**対応しているツール ID（`--tools`）** — `windsurf` も `devin` のエイリアスとして受け付けられます: `amazon-q`, `antigravity`, `auggie`, `bob`, `claude`, `cline`, `command-code`, `codeartsagent`, `codex`, `devin`, `forgecode`, `codebuddy`, `continue`, `costrict`, `crush`, `cursor`, `factory`, `gemini`, `github-copilot`, `hermes`, `iflow`, `junie`, `kilocode`, `kimi`, `kiro`, `lingma`, `minimax-code`, `vibe`, `oh-my-pi`, `opencode`, `pi`, `codeassistant`, `qoder`, `qwen`, `rovodev`, `roocode`, `trae`, `zed`, `zcode`, `agents`

> このリストは、`src/core/config.ts` の `AI_TOOLS` と対応しています。各ツールのスキルとコマンドのパスについては、[対応ツール](supported-tools.md) を参照してください。

**例:**

```bash
# Interactive initialization
openspec init

# Initialize in a specific directory
openspec init ./my-project

# Non-interactive: configure for Claude and Cursor
openspec init --tools claude,cursor

# Non-interactive: configure global MiniMax Code skills
openspec init --tools minimax-code

# Configure for all supported tools
openspec init --tools all

# Override profile for this run
openspec init --profile core

# Skip prompts and auto-cleanup legacy files
openspec init --force
```

**作成されるもの:**

```
openspec/
├── specs/              # 仕様（信頼できる唯一の情報源）
├── changes/            # 提案された変更
└── config.yaml         # プロジェクト設定

.claude/skills/         # Claude Code のスキル（claude を選択した場合）
.cursor/skills/         # Cursor のスキル（cursor を選択した場合）
.cursor/commands/       # Cursor の OPSX コマンド（提供方式にコマンドが含まれる場合）
.agents/skills/         # AGENTS.md 互換ツール用の共有スキル（agents を選択した場合）
... (その他のツールの設定)
```

---

<a id="openspec-update"></a>
### `openspec update`

CLI をアップグレードした後で、OpenSpec の指示ファイルを更新します。現在のグローバルプロファイル、選択したワークフロー、提供方式を使って、AI ツールの設定ファイルを再生成します。

```
openspec update [path] [options]
```

**引数:**

| 引数 | 必須 | 説明 |
|----------|----------|-------------|
| `path` | いいえ | 対象ディレクトリ（デフォルト: カレントディレクトリ） |

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--force` | ファイルが最新であっても強制的に更新する |

**例:**

```bash
# Update instruction files after npm upgrade
npm install -g @fission-ai/openspec@latest
openspec update
```

先にパッケージをアップグレードしてください。指示ファイルはインストール済みの CLI によって生成されるため、古いインストールに対して `openspec update` を実行すると、新しいリリースが提供するワークフローが追加されないまま、すべて最新だと報告されます。

これを可視化するため、`openspec update` は npm レジストリに、より新しい CLI が公開されているかを問い合わせます。お使いのものが古い場合は、アップグレードを提案します。

```text
A newer OpenSpec CLI is available (v1.6.0 → v1.7.0).
  Running from: /usr/local/lib/node_modules/@fission-ai/openspec
? Upgrade to v1.7.0 now? (Y/n)
```

「はい」と答えると `npm install -g @fission-ai/openspec@latest` を実行し、新しい CLI で更新を再実行するため、新しいワークフローが同じコマンドの中で反映されます。npm の終了コードを信用するのではなく、インストールされたバイナリにバージョンを問い合わせることでアップグレードを確認します。そのため、`PATH` の前方にある別のインストールがまだ応答している場合は、成功を主張するのではなく、その旨を知らせます。「いいえ」と答えるとコマンドを表示し、現在の CLI で更新します。Ctrl-C でコマンドを中止できます。

この提案は、対話型のターミナルでのみ、かつ npm がそのインストールを所有している場合、つまり `npm install -g` が実際に解決する唯一のケースでのみ表示されます。それ以外の場合は、インストール方法に合ったコマンドが表示されます。

| OpenSpec のインストール方法 | 表示される内容 |
|---------------------------|--------------|
| グローバルな npm インストール | 対話型のターミナルでは、プロンプトが表示され、アップグレードが実行される。パイプされた出力では、代わりにコマンドが表示される |
| グローバルな pnpm、bun、yarn、volta によるインストール | そのマネージャー自身のコマンド: `pnpm add -g …@latest`、`bun add -g …@latest`、`yarn global add …@latest`、または `volta install …@latest` |
| プロジェクトの依存関係 | 依存関係を更新するようにというメモ（ロックファイルは、そのパッケージマネージャーが所有するため） |
| `npx` / `dlx` のキャッシュ | `npx @fission-ai/openspec@latest update` — このコマンド自体が更新なので、2番目のステップはない |
| git clone | 何も表示されない — バージョンはブランチ次第 |

何かが表示される場合は常に、実行中の CLI が読み込まれたディレクトリが示されます。アップグレードしたのに、古いシムが `PATH` を占有し続けている場合に確認すべきものです。

npm が `npm_config_registry` をエクスポートしている場合はそのレジストリに、そうでなければ `https://registry.npmjs.org` に問い合わせます。`.npmrc` は読み込まれません。ファイルの内容によって外部リクエストの送信先が決まるのは避けるべき流れであり、プロジェクトの `.npmrc` はリポジトリとともに配布されるためです。プライベートミラーを使う場合は、`npm_config_registry` をエクスポートするか、`OPENSPEC_NO_UPDATE_CHECK` を設定してチェックを完全にスキップしてください。チェックは、`CI` が明示的な無効値（`false`、`0`、`no`、`off`、または空）以外に設定されている場合、`NODE_ENV=test` の場合、そして `OPENSPEC_NO_UPDATE_CHECK`（任意の値）、`DO_NOT_TRACK=1`、`OPENSPEC_TELEMETRY=0` のいずれかが設定されている場合にスキップされます。チェックは更新の前に実行され、最大でも1.5秒しか遅延させません。ネットワークがパケットを黙って破棄する場合でも、その時間で諦め、レジストリに到達できない場合は何も表示しません。

**「最新」の判定方法:** スキルファイルは、それを生成したバージョンを記録しているため、
OpenSpec はそれをインストール済みの CLI と比較します。コマンドファイルにはバージョンの
スタンプがないため、コマンドはあってもスキルがないツール（提供方式 `commands`）では、
OpenSpec はファイルの内容を、現在生成するであろう内容と比較します。それらのファイルへの
編集はずれとして扱われ、上書きされます。提供方式が `skills` または `both` の場合は、
記録されたバージョンのみがチェックされるため、バージョンが一致したままの手動編集された
ファイルはそのまま残ります。書き直すには `--force` を使用してください。いずれの場合も、
生成されたファイルは OpenSpec が所有するものです。独自の指示は、他の場所に置いてください。

---

<a id="stores-standalone-openspec-repos"></a>
## ストア（独立した OpenSpec リポジトリ）

> **ベータ版。** ストアとその上に構築された機能（参照、作業コンテキスト、ワークセット）は新しいものです。コマンド名、フラグ、ファイル形式、JSON 出力は、リリース間で変わる可能性があります。問題から入るウォークスルーについては、[ストアガイド](stores-beta/user-guide.md) を参照してください。

ストアとは、このマシンに登録した独立した OpenSpec リポジトリのことで、たとえば計画用リポジトリや契約用リポジトリです。ストアを登録すると、通常のコマンド（`list`、`show`、`status`、`validate`、`new change`、`archive` など）が、`--store <id>` を渡すことでどこからでもそのストアに対して動作します。

<a id="openspec-store-setup"></a>
### `openspec store setup`

ローカルストアを作成して登録します。ターミナルで引数なしで実行すると、
OpenSpec がセットアップをガイドします。エージェントやスクリプトは、明示的な入力を渡し、
`--json` を使用してください。

```bash
openspec store setup [id] [options]
```

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--path <path>` | ストアを置くフォルダ（例: `~/openspec/<id>`） |
| `--remote <url>` | 新しいストアの `store.yaml` に、正規のリモートを記録する |
| `--init-git` | 最初のコミット付きで Git リポジトリを初期化する（デフォルト） |
| `--no-init-git` | Git に関するすべての操作をスキップする: init も最初のコミットも行わない |
| `--json` | JSON を出力する |

非対話型の実行（`--json`、スクリプト、エージェント）では、ストア ID と `--path` の両方を渡す必要があります。対話型のターミナルでは、セットアップは、目に見える、ユーザー所有の場所（たとえば `~/openspec/<id>`）の編集可能な候補を示して場所を尋ねます。OpenSpec が管理するデータディレクトリがデフォルトになることはありません。

例:

```bash
openspec store setup
openspec store setup team-context
openspec store setup team-context --path ~/openspec/team-context --no-init-git
openspec store setup team-context --path ~/openspec/team-context --no-init-git --json
```

<a id="openspec-store-register"></a>
### `openspec store register`

既存のローカルストアのフォルダを登録します。ストアのベータ期間中は、変更が存在する前、
仕様が適用される前、変更がアーカイブされる前でも、ルートを登録できます。その場合、
`openspec/changes/`、`openspec/specs/`、`openspec/changes/archive/` は、
通常のコマンドがそれらを作成するまで存在しないことがあります。
`store: <id>` を宣言する設定のみのリポジトリは、別のストアへのポインターのままであり、
そのポインターが削除されない限り、ストアのルートとしては登録されません。

```bash
openspec store register [path] [options]
```

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--id <id>` | ストア ID。デフォルトは、ストアのメタデータまたはフォルダ名 |
| `--yes` | 健全な OpenSpec ルートに対して、ストアの識別メタデータを作成することを確認する |
| `--json` | JSON を出力する |

<a id="openspec-store-unregister"></a>
### `openspec store unregister`

ファイルを削除せずに、ローカルストアの登録を解除します。

```bash
openspec store unregister <id> [--json]
```

ストアが移動された場合、別の場所にクローンされた場合、またはこのマシンで OpenSpec に
もう表示させたくない場合に使用します。

<a id="openspec-store-remove"></a>
### `openspec store remove`

ローカルストアの登録を解除し、そのローカルフォルダを削除します。

```bash
openspec store remove <id> [--yes] [--json]
```

`remove` は、対話型のターミナルでは、削除する前に正確なフォルダを表示します。
エージェント、スクリプト、JSON を使う呼び出し元は、削除を確認するために `--yes` を渡す必要があります。
OpenSpec は、一致するストアのメタデータを含まないフォルダの削除を拒否します。

<a id="openspec-store-list"></a>
### `openspec store list`

ローカルに登録されているストアを一覧表示します。

```bash
openspec store list [--json]
openspec store ls [--json]
```

<a id="openspec-store-doctor"></a>
### `openspec store doctor`

ローカルストアの登録、メタデータ、Git の存在をチェックします。

```bash
openspec store doctor [id] [--json]
```

doctor は診断専用です。ストアを変更することなく、欠落しているルート、メタデータの不一致、無効なローカルレジストリの状態を報告します。

<a id="referencing-stores-from-a-project"></a>
### プロジェクトからストアを参照する

プロジェクトのリポジトリは、作業が利用するストアを `openspec/config.yaml` で宣言できます。

```yaml
schema: spec-driven
references:
  - team-context
```

以後、そのリポジトリでの `openspec instructions` の出力（成果物ごとのものと `apply` のもの、JSON と人間向けのモードの両方）には、参照している各ストアの仕様のインデックスが含まれます。仕様 ID、各仕様の Purpose セクションからの1行の要約、そして取得コマンド（`openspec show <spec-id> --type spec --store <id>`）です。インデックスは、実行のたびに、登録済みのチェックアウトからその場で組み立てられます。仕様の内容が出力にコピーされることはありません。

参照は読み取り専用のコンテキストです。コマンドの動作場所を変更することはありません。作業はリポジトリ自身のルートにとどまり、参照先のストアへの書き込みは、引き続き明示的な `--store` の操作です。解決できない参照（たとえば、このマシンに登録されていないストア）は、インデックス内で正確な修正方法を添えた警告に格下げされ、指示の生成は引き続き行われます。`openspec doctor` は、参照の健全性を1か所で報告します。

<a id="recording-where-a-store-is-cloned-from"></a>
### ストアのクローン元を記録する

ストアは、コミットされる識別ファイルに、正規のクローン元を記録できます。これにより、オンボーディングが「ストアを登録してください」で行き詰まることはありません。

```bash
openspec store setup team-context --path ~/openspec/team-context \
  --remote git@github.com:acme/team-context.git
```

リモートは最初のコミットに含まれる `.openspec-store/store.yaml` に記録されるため、すべてのクローンは最初からそれを把握しています。既存のストアでは、`store.yaml` を手動で編集してコミットします。`store doctor` は記録されたリモート（とチェックアウトで観測された Git の origin）を表示し、setup/register の共有ガイダンスはそれを名指しし、register はチェックアウトの origin をマシンローカルのレジストリに記録します。

参照の宣言にもクローン元を含めることができ、まだストアを持っていないチームメイトは、完全で貼り付け可能な修正方法（`git clone <remote> <path> && openspec store register <path> --id <id>`）を得られます。

```yaml
references:
  - { id: team-context, remote: "git@github.com:acme/team-context.git" }
```

リモートを記録することは同期ではありません。OpenSpec は、自らクローン、プル、プッシュを行うことは決してありません。

<a id="declaring-a-default-store"></a>
### デフォルトのストアを宣言する

計画が完全に外部化されているリポジトリ、つまりローカルの `openspec/specs/` や `openspec/changes/` がないリポジトリは、すべてのコマンドで `--store` を渡す代わりに、ストアを一度だけ宣言できます。

```yaml
# openspec/config.yaml (the only file under openspec/)
store: team-context
```

通常のコマンドは、宣言されたストアに自動的に解決されます。ルートバナーと JSON の `root` ブロックは、ストア ID とともに `source: "declared"` を報告し、表示されるヒントには引き続き `--store <id>` が含まれます。この宣言はフォールバックであり、上書きではありません。明示的な `--store` が常に優先され、実際の計画フォルダがあるディレクトリでは、ポインターは（警告付きで）無視されます。ポインターのリポジトリをローカルの OpenSpec ルートに変換するには、`store:` の行を削除して `openspec init` を実行します。宣言が存在する間は、init は骨組みの作成を拒否します。

マシンレベルのバリエーションにより、すべてのリポジトリを一度にカバーできます。`openspec config set defaultStore <id>`（設定の項を参照）です。これは、`--store`、ローカルルート、プロジェクトのポインターがすべて解決に失敗した後でのみ参照されます。その場合、ルートバナーと JSON の `root` ブロックは `source: "global_default"` を報告します。

<a id="doctor-relationship-health"></a>
## Doctor（関係性の健全性）

読み取り専用の1つの問いを、1か所で: OpenSpec のルートは健全か、そしてそれが参照しているストアは、このマシンで利用可能か？

```bash
openspec doctor [--store <id>] [--json]
```

レポートは、ルートの健全性、ストアのメタデータの健全性（記録されたリモートとチェックアウトの origin が食い違っている場合の注記や、ストアのチェックアウトが最後に fetch した upstream のトラッキング参照より遅れている場合の注記を含む）、そして参照の健全性（instructions が表示するのと同じ診断で、未解決の参照にはクローンの修正方法が付く）を分けて報告します。どの重大度の健全性の所見も終了コード0で終了します。エージェントは `status` 配列を読みます。終了コード1となるのは、コマンドの失敗（ルートがない、不明なストア）のみです。doctor は、クローン、同期、修復を決して行いません。健全性ではなく、組み立てられたセットそのものが必要な場合は、`openspec context` を使用してください。

<a id="working-context-the-assembled-set"></a>
## 作業コンテキスト（組み立てられたセット）

OpenSpec の宣言を通じてこの作業に関連するすべてのものを、1つの作業セットにまとめたもの: OpenSpec のルートと、それが参照するストアです。

```bash
openspec context [--store <id>] [--json] [--code-workspace <path> [--force]]
```

JSON のブリーフはエージェントが利用できる形式です（利用可能な各参照ストアには取得手順が含まれ、未解決のメンバーには、instructions や doctor が表示するのと同じ修正方法が含まれます）。`--code-workspace` を指定すると、ルートと利用可能な参照ストア（`ref:<id>` フォルダ）を含む VS Code のワークスペースファイルが追加で書き込まれます。これはこのコマンドが行う唯一の書き込みで、ファイルがすでに存在する場合は `--force` なしでは拒否されます。利用できないメンバーは報告されるだけで、推測されることはありません。

「作業コンテキスト」は組み立てられたセットのことであり、`openspec/config.yaml` の `context:` フィールドは、指示に注入されるプロジェクトの背景情報です。この2つは別のものです。`openspec doctor` は、そのセットが健全かどうかに答え、`openspec context` は、そのセットが何であるかに答えます。

<a id="personal-worksets"></a>
## 個人用ワークセット

> **ベータ版。** ワークセットは新しいベータ機能の一部です。コマンド、フラグ、ファイル形式は、リリース間で変わる可能性があります。ウォークスルーについては、[ストアガイド](stores-beta/user-guide.md#worksets-reopen-the-folders-you-work-on-together) を参照してください。

ワークセットとは、一緒に作業するフォルダ（計画用のルートと、選択したその他のもの）についての、個人用の名前付きビューで、マシン上に保持され、ツールで名前を指定して開き直されます。完全にローカルのものです。コミットも共有もされず、宣言から導かれることもなく、削除してもメンバーのフォルダには一切触れません。

```bash
openspec workset create [name] [--member <path> | --member <name>=<path>]... [--tool <id>] [--json]
openspec workset list [--json]
openspec workset open <name> [--tool <id>]
openspec workset remove <name> [--yes] [--json]
```

`create` は短いガイド付きのフローを実行します（非対話型では `--member` フラグを受け取ります。最初のメンバーがプライマリで、セッションはそこから始まります）。`open` は選択したツールを起動します。エディタ（VS Code、Cursor）はすべてのメンバーを含むウィンドウを開いて戻ります。CLI エージェント（Claude Code、codex）は、すべてのメンバーがアタッチされ、プロンプトが事前入力されていない状態で、このターミナルをセッションとして引き継ぎ、終了するまで続きます。開くときに存在しないメンバーのフォルダは、注記付きでスキップされ、残りが開かれます。保存されたツールの設定は、`--tool` で開くたびに上書きできます。

新しいツールへの対応は、コードではなく設定で行います。すべてのツールは、2つの起動スタイルのいずれかです。`workspace-file`（生成された `.code-workspace` で起動する）または `attach-dirs`（メンバーごとに1つのアタッチフラグ）です。グローバルな `config.json`（`openspec config edit` で開きます）の `openers` キーで、ツールを追加したり、組み込みのものをフィールドごとに調整したりできます。

```json
{
  "openers": {
    "zed": { "style": "workspace-file" },
    "claude": { "attach_flag": "--dir" }
  }
}
```

ワークセットの状態はすべて、グローバルデータディレクトリの `worksets/` フォルダの下にあります（保存されたビューと、開くたびに再生成される `<name>.code-workspace` ファイル）。そのフォルダを削除すれば、すべての痕跡が消えます。

---

<a id="browsing-commands"></a>
## 閲覧コマンド

<a id="openspec-list"></a>
### `openspec list`

プロジェクト内の変更または仕様を一覧表示します。

```
openspec list [options]
```

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--specs` | 変更ではなく仕様を一覧表示する |
| `--changes` | 変更を一覧表示する（デフォルト） |
| `--sort <order>` | `recent`（デフォルト）または `name` で並べ替える |
| `--json` | JSON として出力する |

**例:**

```bash
# List all active changes
openspec list

# List all specs
openspec list --specs

# JSON output for scripts
openspec list --json
```

**出力（テキスト）:**

```
Changes:
  add-dark-mode     No tasks      just now
```

---

<a id="openspec-view"></a>
### `openspec view`

仕様と変更を探索するための、インタラクティブなダッシュボードを表示します。

```
openspec view
```

プロジェクトの仕様と変更をナビゲートするための、ターミナルベースのインターフェースを開きます。

---

<a id="openspec-show"></a>
### `openspec show`

変更または仕様の詳細を表示します。

```
openspec show [item-name] [options]
```

**引数:**

| 引数 | 必須 | 説明 |
|----------|----------|-------------|
| `item-name` | いいえ | 変更または仕様の名前（省略すると尋ねられる） |

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--type <type>` | 種類を指定する: `change` または `spec`（曖昧でなければ自動検出される） |
| `--json` | JSON として出力する |
| `--no-interactive` | プロンプトを無効にする |

**変更に固有のオプション:**

| オプション | 説明 |
|--------|-------------|
| `--deltas-only` | デルタ仕様のみを表示する（JSON モード） |

**仕様に固有のオプション:**

| オプション | 説明 |
|--------|-------------|
| `--requirements` | シナリオを除き、要件のみを表示する（JSON モード） |
| `--no-scenarios` | シナリオの内容を除外する（JSON モード） |
| `-r, --requirement <id>` | 1始まりのインデックスで特定の要件を表示する（JSON モード） |

**例:**

```bash
# Interactive selection
openspec show

# Show a specific change
openspec show add-dark-mode

# Show a specific spec
openspec show auth --type spec

# JSON output for parsing
openspec show add-dark-mode --json
```

---

<a id="validation-commands"></a>
## 検証コマンド

<a id="openspec-validate"></a>
### `openspec validate`

変更と仕様の構造上の問題を検証し、変更の MODIFIED 要件を、それが置き換えるメイン仕様に照らしてチェックします。

```
openspec validate [item-name] [options]
```

仕様のデルタがゼロの変更は、その `.openspec.yaml` が `skip_specs: true` を宣言していない限り、検証に失敗します（純粋なリファクタリング、ツール、ドキュメント作業向け。[レシピ5](examples.md#recipe-5-a-refactor-with-no-behavior-change) を参照）。

**引数:**

| 引数 | 必須 | 説明 |
|----------|----------|-------------|
| `item-name` | いいえ | 検証する特定の項目（省略すると尋ねられる） |

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--all` | すべての変更と仕様を検証する |
| `--changes` | すべての変更を検証する |
| `--specs` | すべての仕様を検証する |
| `--archived` | アーカイブされた変更のすべてのタスクが完了していることを検証する（pre-commit のリント用） |
| `--type <type>` | 名前が曖昧な場合に種類を指定する: `change` または `spec` |
| `--strict` | 厳格な検証モードを有効にする |
| `--json` | JSON として出力する |
| `--concurrency <n>` | 並列検証の最大数（デフォルト: 6、または環境変数 `OPENSPEC_CONCURRENCY`） |
| `--no-interactive` | プロンプトを無効にする |

`--archived` は独立したスコープです。デルタ仕様は検証しません（アーカイブ時にすでに適用されているため）。`changes/archive/` 配下のすべての変更について、`tasks.md` のチェックボックスがすべてオンになっていることを確認し、オフのものがあれば、ゼロ以外の終了コードで終了します。これにより、未完了の作業を残したままアーカイブされた変更を検出できます。pre-commit フックに便利です。

**例:**

```bash
# Interactive validation
openspec validate

# Validate a specific change
openspec validate add-dark-mode

# Validate all changes
openspec validate --changes

# Validate everything with JSON output (for CI/scripts)
openspec validate --all --json

# Strict validation with increased parallelism
openspec validate --all --strict --concurrency 12

# Fail if any archived change still has unchecked tasks
openspec validate --archived
```

**出力（テキスト）:**

```
Validating add-dark-mode...
  ✓ proposal.md valid
  ✓ specs/ui/spec.md valid
  ⚠ design.md: missing "Technical Approach" section

1 warning found
```

**出力（JSON）:**

```json
{
  "version": "1.0.0",
  "results": {
    "changes": [
      {
        "name": "add-dark-mode",
        "valid": true,
        "warnings": ["design.md: missing 'Technical Approach' section"]
      }
    ]
  },
  "summary": {
    "total": 1,
    "valid": 1,
    "invalid": 0
  }
}
```

---

<a id="lifecycle-commands"></a>
## ライフサイクルコマンド

<a id="openspec-archive"></a>
### `openspec archive`

完了した変更をアーカイブし、デルタ仕様をメインの仕様にマージします。

```
openspec archive [change-name] [options]
```

**引数:**

| 引数 | 必須 | 説明 |
|----------|----------|-------------|
| `change-name` | いいえ | アーカイブする変更（省略すると尋ねられる。プロンプトに答えられない場合は必須） |

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `-y, --yes` | 確認プロンプトをスキップする。AI エージェント、CI ジョブ、標準入力が閉じられた実行など、プロンプトに答えられない場合は必須 |
| `--skip-specs` | 1回のアーカイブ実行で、仕様の更新をスキップする。恒久的に仕様のデルタを持たない変更は、代わりに `.openspec.yaml` で `skip_specs: true` を宣言してください。フラグなしでアーカイブされます |
| `--no-validate` | 検証をスキップする（確認が必要）。機能の廃止も無効になります。検証結果がなければ、何も廃止されません |

**例:**

```bash
# Interactive archive (asks which change, then confirms)
openspec archive

# Archive specific change
openspec archive add-dark-mode

# Archive without prompts (agents, CI, scripts)
openspec archive add-dark-mode --yes

# Archive a tooling change that doesn't affect specs
openspec archive update-ci-config --skip-specs
```

**機能を廃止する:** 変更のメタデータに、廃止マーカーを追加します。

```yaml
# openspec/changes/retire-legacy/.openspec.yaml
schema: spec-driven
retire_capabilities: true
```

その後、通常どおり変更をアーカイブします。

```bash
openspec archive retire-legacy --yes
```

変更がその機能の最後の要件を削除すると、OpenSpec はそのライブの
`spec.md` を削除します。同じ変更に含まれる他の機能のデルタは、引き続きそれぞれの
メイン仕様を更新します。マーカーがない場合、アーカイブはファイルを変更する前に停止し、
追加するよう伝えます。

**動作:**

1. 変更を検証する（`--no-validate` の場合を除く）
2. 確認を求める（`--yes` の場合を除く）
3. メインの仕様を変更する前に、アーカイブ先を確保する
4. アクティブなデルタ仕様を検証して `openspec/specs/` にマージする — 変更が最後の要件を削除する機能は廃止され、その仕様ファイルは削除されるが、それは変更の `.openspec.yaml` が `schema:` と並べて `retire_capabilities: true` を宣言している場合に限る
5. 変更フォルダを `openspec/changes/archive/YYYY-MM-DD-<name>/` に移動する
6. 完全なアーカイブが確保される前に、仕様の変更や最終的な移動が失敗した場合は、仕様を復元し、変更をアクティブなパスに残すか、そこへ戻す
7. 検証済みのフォールバックコピーが完了したが、ステージングされたソースのクリーンアップが失敗した場合は、復旧のために、完全なアーカイブとコミット済みの仕様の状態を保持する

**端末がない場合:** AI エージェント、CI ジョブ、標準入力が閉じられた実行では、
手順2に答えられないため、アーカイブは何も触れる前に停止し、終了コード1で終了し、
再実行するコマンドを示します — 渡した他のフラグを引き継いだ `openspec archive <name> --yes` です。
手間を省くには、`--yes`（と変更名）を最初から渡してください。

---

<a id="workflow-commands"></a>
## ワークフローコマンド

これらのコマンドは、成果物駆動の OPSX ワークフローを支援します。進捗を確認する人間にも、次のステップを判断するエージェントにも役立ちます。

<a id="openspec-new-change"></a>
### `openspec new change`

解決された OpenSpec のルートに、変更ディレクトリと、任意のチェックイン対象のメタデータを作成します。

```bash
openspec new change <name> [options]
```

変更名は、小文字のケバブケースでなければなりません。小文字、数字、単一のハイフンのみ使用できます。
スペース、アンダースコア、大文字、連続するハイフン、先頭/末尾のハイフンを含めることはできません。
先頭に数字を付けることは許可されているため、変更に順序や階層を付けるために
名前にプレフィックスを付けられます。たとえば `100-add-feature` や `00001-add-auth` です。

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--description <text>` | `README.md` に追加する説明 |
| `--goal <text>` | 変更とともに保存する、任意の目標メタデータ |
| `--schema <name>` | 使用するワークフローのスキーマ |
| `--store <id>` | OpenSpec のルートとして使用するストア ID（ストアとは、登録した独立した OpenSpec リポジトリ） |
| `--json` | JSON を出力する |

例:

```bash
openspec new change add-billing-api
openspec new change add-billing-api --store team-context --json
```

<a id="openspec-status"></a>
### `openspec status`

変更の成果物の完了状況を表示します。

```
openspec status [options]
```

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--change <id>` | 変更名（省略すると尋ねられる） |
| `--schema <name>` | スキーマの上書き（変更の設定から自動検出される） |
| `--json` | JSON として出力する |

**例:**

```bash
# Interactive status check
openspec status

# Status for specific change
openspec status --change add-dark-mode

# JSON for agent use
openspec status --change add-dark-mode --json
```

**出力（テキスト）:**

```
Change: add-dark-mode
Schema: spec-driven
Progress: 2/4 artifacts complete

[x] proposal
[x] specs
[ ] design
[-] tasks (blocked by: design)
```

`skip_specs: true` を宣言している変更は、specs のステージが `[~] specs (skipped: change declares skip_specs)` と表示され、進捗のカウントから除外されます。

**出力（JSON）:**

```json
{
  "changeName": "add-dark-mode",
  "schemaName": "spec-driven",
  "isPlanningComplete": false,
  "isComplete": false,
  "applyRequires": ["tasks"],
  "artifacts": [
    {"id": "proposal", "outputPath": "proposal.md", "status": "done", "requires": []},
    {"id": "specs", "outputPath": "specs/**/*.md", "status": "done", "requires": ["proposal"]},
    {"id": "design", "outputPath": "design.md", "status": "ready", "requires": ["proposal"]},
    {"id": "tasks", "outputPath": "tasks.md", "status": "blocked", "requires": ["specs", "design"], "missingDeps": ["design"]}
  ]
}
```

`isPlanningComplete` は、スキップされていないすべての計画用成果物が存在するかどうかを報告します。
スキップされた成果物は、作成されなくても満たされたものとして扱われます。実装タスクが
完了しているかどうかは報告しません。`isComplete` は、同じ値を持つ互換性のためのエイリアスとして
残されています。

成果物は依存関係の順に並べられます。依存先が、それを必要とするものの後に現れることはありません。
また、同時に準備ができる成果物（spec-driven の `specs` と `design` は、どちらも `proposal` のみを必要とします）は、
アルファベット順ではなく、スキーマが宣言した順序を保ちます。したがって、最初の `ready` の
エントリが、次に書くべき成果物です。

---

<a id="openspec-instructions"></a>
### `openspec instructions`

成果物の作成やタスクの適用のための、拡張された指示を取得します。AI エージェントが、次に何を作成すべきかを理解するために使用します。

```
openspec instructions [artifact] [options]
```

**引数:**

| 引数 | 必須 | 説明 |
|----------|----------|-------------|
| `artifact` | いいえ | 成果物 ID、またはワークフローの入力インターフェース: `apply` または `archive` |

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--change <id>` | 変更名（非対話型モードでは必須） |
| `--schema <name>` | スキーマの上書き |
| `--json` | JSON として出力する |

**特別な場合:** タスクの実装の指示を取得するには `apply` を使用します。有効な変更について、現在の読み取り専用のアーカイブ入力（`context` と
`operationGuidance`）を取得するには `archive` を使用します。何かをアーカイブしたり変更したりすることはありません。

**例:**

```bash
# Get instructions for next artifact
openspec instructions --change add-dark-mode

# Get specific artifact instructions
openspec instructions design --change add-dark-mode

# Get apply/implementation instructions
openspec instructions apply --change add-dark-mode

# Get current archive operation inputs without archiving
openspec instructions archive --change add-dark-mode --json

# JSON for agent consumption
openspec instructions design --change add-dark-mode --json
```

**出力に含まれるもの:**

- 成果物のテンプレートの内容
- 設定からのプロジェクトのコンテキスト
- 依存先の成果物の内容
- 設定からの、成果物ごとのルール
- `apply`/`archive` 向けの、現在のプロジェクトのコンテキストと、該当する操作ガイダンス

操作の入力は、呼び出しのたびに、解決されたリポジトリまたは選択したストアから読み取られます。
プロジェクトのコンテキストは、プロンプトレベルで必須の入力です。エージェントはこれを読み、
関連するプロジェクトの事実、規約、制約を適用します。操作ガイダンスは、
省略可能な追加のアドバイスです。エージェントはすべての項目を検討し、組み込みのワークフローと
両立する、該当する項目にのみ従います。どちらのフィールドも、ユーザーの明示的な選択、
CLI が制御する状態、組み込みの指示、成果物のルールとは別のものとして扱われます。
競合するコンテキストは報告され、競合する、または該当しないガイダンスには従わず、その理由が説明されます。
これらは、生成されたエージェントに対する動作上の契約であり、強制可能な CLI のチェックではありません。
`instructions archive` が返すのは、選択した変更、任意の入力、ルートのメタデータのみです。
静的なアーカイブワークフローは含まれません。

`skip_specs: true` でスキップされた成果物については、出力は警告のみです（JSON では `skipped`/`warning` フィールドが追加されます）。その成果物を作成してはいけません。

---

<a id="openspec-templates"></a>
### `openspec templates`

スキーマ内のすべての成果物について、解決されたテンプレートのパスを表示します。

```
openspec templates [options]
```

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--schema <name>` | 調べるスキーマ（デフォルト: `spec-driven`） |
| `--json` | JSON として出力する |

**例:**

```bash
# Show template paths for default schema
openspec templates

# Show templates for custom schema
openspec templates --schema my-workflow

# JSON for programmatic use
openspec templates --json
```

**出力（テキスト）:**

```
Schema: spec-driven

Templates:
  proposal  → ~/.openspec/schemas/spec-driven/templates/proposal.md
  specs     → ~/.openspec/schemas/spec-driven/templates/specs.md
  design    → ~/.openspec/schemas/spec-driven/templates/design.md
  tasks     → ~/.openspec/schemas/spec-driven/templates/tasks.md
```

---

<a id="openspec-schemas"></a>
### `openspec schemas`

利用可能なワークフローのスキーマを、その説明と成果物のフローとともに一覧表示します。

```
openspec schemas [options]
```

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--json` | JSON として出力する |
| `--store <id>` | 登録済みのストアを OpenSpec のルートとして使用する |

**例:**

```bash
openspec schemas
```

**出力:**

```
Available schemas:

  spec-driven (package)
    The default spec-driven development workflow
    Flow: proposal → specs → design → tasks

  my-custom (project)
    Custom workflow for this project
    Flow: research → proposal → tasks
```

---

<a id="schema-commands"></a>
## スキーマコマンド

カスタムワークフローのスキーマを作成・管理するためのコマンドです。

<a id="openspec-schema-init"></a>
### `openspec schema init`

プロジェクトローカルの新しいスキーマを作成します。

```
openspec schema init <name> [options]
```

**引数:**

| 引数 | 必須 | 説明 |
|----------|----------|-------------|
| `name` | はい | スキーマ名（ケバブケース） |

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--description <text>` | スキーマの説明 |
| `--artifacts <list>` | カンマ区切りの成果物 ID（デフォルト: `proposal,specs,design,tasks`） |
| `--default` | プロジェクトのデフォルトスキーマとして設定する |
| `--no-default` | デフォルトに設定するかどうかを尋ねない |
| `--force` | 既存のスキーマを上書きする |
| `--json` | JSON として出力する |

**例:**

```bash
# Interactive schema creation
openspec schema init research-first

# Non-interactive with specific artifacts
openspec schema init rapid \
  --description "Rapid iteration workflow" \
  --artifacts "proposal,tasks" \
  --default
```

**作成されるもの:**

```
openspec/schemas/<name>/
├── schema.yaml           # スキーマの定義
└── templates/
    ├── proposal.md       # 各成果物のテンプレート
    ├── specs.md
    ├── design.md
    └── tasks.md
```

---

<a id="openspec-schema-fork"></a>
### `openspec schema fork`

カスタマイズのために、既存のスキーマをプロジェクトにコピーします。

```
openspec schema fork <source> [name] [options]
```

**引数:**

| 引数 | 必須 | 説明 |
|----------|----------|-------------|
| `source` | はい | コピーするスキーマ |
| `name` | いいえ | 新しいスキーマ名（デフォルト: `<source>-custom`） |

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--force` | 既存のコピー先を上書きする |
| `--json` | JSON として出力する |

**例:**

```bash
# Fork the built-in spec-driven schema
openspec schema fork spec-driven my-workflow
```

---

<a id="openspec-schema-validate"></a>
### `openspec schema validate`

スキーマの構造とテンプレートを検証します。

```
openspec schema validate [name] [options]
```

**引数:**

| 引数 | 必須 | 説明 |
|----------|----------|-------------|
| `name` | いいえ | 検証するスキーマ（省略するとすべてを検証する） |

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--verbose` | 検証の詳細なステップを表示する |
| `--json` | JSON として出力する |

**例:**

```bash
# Validate a specific schema
openspec schema validate my-workflow

# Validate all schemas
openspec schema validate
```

---

<a id="openspec-schema-which"></a>
### `openspec schema which`

スキーマがどこから解決されるかを表示します（優先順位のデバッグに便利です）。

```
openspec schema which [name] [options]
```

**引数:**

| 引数 | 必須 | 説明 |
|----------|----------|-------------|
| `name` | いいえ | スキーマ名 |

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--all` | すべてのスキーマをその解決元とともに一覧表示する |
| `--json` | JSON として出力する |

**例:**

```bash
# Check where a schema comes from
openspec schema which spec-driven
```

**出力:**

```
spec-driven resolves from: package
  Source: /usr/local/lib/node_modules/@fission-ai/openspec/schemas/spec-driven
```

**スキーマの優先順位:**

1. プロジェクト: `openspec/schemas/<name>/`
2. ユーザー: `~/.local/share/openspec/schemas/<name>/`
3. パッケージ: 組み込みのスキーマ

---

<a id="configuration-commands"></a>
## 設定コマンド

<a id="openspec-config"></a>
### `openspec config`

グローバルな OpenSpec の設定を表示・変更します。

```
openspec config <subcommand> [options]
```

**サブコマンド:**

| サブコマンド | 説明 |
|------------|-------------|
| `path` | 設定ファイルの場所を表示する |
| `list` | 現在のすべての設定を表示する |
| `get <key>` | 特定の値を取得する |
| `set <key> <value>` | 値を設定する |
| `unset <key>` | キーを削除する |
| `reset` | デフォルトに戻す |
| `edit` | `$EDITOR` で開く |
| `profile [preset]` | ワークフローのプロファイルを、対話型またはプリセットで設定する |

**例:**

```bash
# Show config file path
openspec config path

# List all settings
openspec config list

# Get a specific value
openspec config get telemetry.enabled

# Set a value (disable anonymous usage telemetry)
openspec config set telemetry.enabled false

# Set a string value explicitly
openspec config set user.name "My Name" --string

# Remove a custom setting
openspec config unset user.name

# Set a machine-level default store (fallback root when no --store,
# local root, or project store: pointer resolves)
openspec config set defaultStore team-plans

# Reset all configuration
openspec config reset --all --yes

# Edit config in your editor
openspec config edit

# Configure profile with action-based wizard
openspec config profile

# Fast preset: switch workflows to core (keeps delivery mode)
openspec config profile core
```

**テレメトリのオプトアウト:** `telemetry.enabled` は、未設定の場合はデフォルトでオンです（オプトアウト方式）。
匿名の利用統計と `openspec update` のバージョンチェックを無効にするには、`false` に設定します。
環境変数は設定よりも優先されます。`OPENSPEC_TELEMETRY=0`、`DO_NOT_TRACK=1`、
および真と解釈される `CI` の値（`true`/`1`/`yes` など）は、設定値にかかわらず、常にテレメトリを無効にします。

`openspec config profile` は、現在の状態の要約から始まり、次から選択できます。
- 提供方式とワークフローを変更する
- 提供方式のみを変更する
- ワークフローのみを変更する
- 現在の設定を維持する（終了）

現在の設定を維持する場合、変更は書き込まれず、更新のプロンプトも表示されません。
設定の変更はないが、現在のプロジェクトのファイルがグローバルなプロファイル/提供方式と同期していない場合、OpenSpec は警告を表示し、`openspec update` を提案します。
`Ctrl+C` を押すと、フローもきれいに（スタックトレースなしで）キャンセルされ、終了コード `130` で終了します。
ワークフローのチェックリストでは、`[x]` は、そのワークフローがグローバル設定で選択されていることを意味します。それらの選択をプロジェクトのファイルに適用するには、`openspec update` を実行します（プロジェクト内で `Apply changes to this project now?` と尋ねられたときに選択することもできます）。

**対話型の例:**

```bash
# Delivery-only update
openspec config profile
# choose: Change delivery only
# choose delivery: Skills only

# Workflows-only update
openspec config profile
# choose: Change workflows only
# toggle workflows in the checklist, then confirm
```

---

<a id="utility-commands"></a>
## ユーティリティコマンド

<a id="openspec-feedback"></a>
### `openspec feedback`

OpenSpec についてのフィードバックを送信します。GitHub の issue を作成します。

```
openspec feedback <message> [options]
```

**引数:**

| 引数 | 必須 | 説明 |
|----------|----------|-------------|
| `message` | はい | フィードバックの要約。長いテキストは issue のタイトルでは短縮され、本文には全文が保持される |

**オプション:**

| オプション | 説明 |
|--------|-------------|
| `--body <text>` | 要約の後に含める追加の詳細 |

**要件:** GitHub CLI（`gh`）がインストールされ、認証されている必要があります。

**例:**

```bash
openspec feedback "Add support for custom artifact types" \
  --body "I'd like to define my own artifact types beyond the built-in ones."
```

---

<a id="openspec-completion"></a>
### `openspec completion`

OpenSpec CLI のシェル補完を管理します。

```
openspec completion <subcommand> [shell]
```

**サブコマンド:**

| サブコマンド | 説明 |
|------------|-------------|
| `generate [shell]` | 補完スクリプトを標準出力に出力する |
| `install [shell]` | お使いのシェル用の補完をインストールする |
| `uninstall [shell]` | インストールした補完を削除する |

**対応しているシェル:** `bash`、`zsh`、`fish`、`powershell`

**例:**

```bash
# Install completions (auto-detects shell)
openspec completion install

# Install for specific shell
openspec completion install zsh

# Generate script for manual installation (bash)
openspec completion generate bash > ~/.bash_completion.d/openspec

# Uninstall
openspec completion uninstall
```

**Windows（PowerShell）:** 現在の PowerShell ホスト用の補完をインストールします。

```powershell
$env:PROFILE = $PROFILE
openspec completion install powershell
. $PROFILE
```

`$env:PROFILE` は、このセッションでどのプロファイルを設定するかを OpenSpec に伝えます。
インストーラーは、存在しないプロファイルのディレクトリを作成し、`OpenSpecCompletion.ps1` を読み込む
管理対象のブロックを追加します。プロファイルを再読み込みすると、すぐに補完が有効になります。

現在のホストからアンインストールするには、次を実行します。

```powershell
$env:PROFILE = $PROFILE
openspec completion uninstall powershell
```

現在のセッションから補完をクリアするには、アンインストール後に PowerShell を再起動してください。

補完はオプトインです。CLI は、対話型のターミナルで初めてコマンドを実行したときに、
stderr に一度だけ補完について触れ、以後は表示しません。すでに補完をインストール済みの場合も
表示されません。このヒントを完全に抑制するには、`OPENSPEC_NO_COMPLETIONS=1` を設定します。

---

<a id="exit-codes"></a>
## 終了コード

| コード | 意味 |
|------|---------|
| `0` | 成功 |
| `1` | エラー（検証の失敗、ファイルの欠落など） |

---

<a id="environment-variables"></a>
## 環境変数

| 変数 | 説明 |
|----------|-------------|
| `OPENSPEC_TELEMETRY` | `0` に設定すると、テレメトリと `openspec update` のバージョンチェックを無効にする（グローバル設定の `telemetry.enabled` を上書きする） |
| `DO_NOT_TRACK` | `1` に設定すると、テレメトリと `openspec update` のバージョンチェックを無効にする（標準の DNT シグナル。設定を上書きする） |
| `OPENSPEC_CONCURRENCY` | 一括検証のデフォルトの並列数（デフォルト: 6） |
| `EDITOR` または `VISUAL` | `openspec config edit` で使用するエディタ |
| `NO_COLOR` | 設定すると、カラー出力を無効にする |
| `OPENSPEC_NO_ANIMATION` | 設定すると、`openspec init` のウェルカムアニメーションを無効にする |
| `OPENSPEC_NO_COMPLETIONS` | `1` に設定すると、シェル補完に関する一度きりのヒントを抑制する |
| `OPENSPEC_NO_UPDATE_CHECK` | 設定すると（空を含む任意の値）、`openspec update` による、公開されたより新しい CLI のチェックを無効にする。`CI` が設定されている場合（`false`/`0`/`no`/`off` でない限り）や `NODE_ENV=test` の場合もスキップされる |
| `npm_config_registry` | `openspec update` のバージョンチェックが問い合わせるレジストリ。`http(s)` の URL でなければならず、そうでない場合は `https://registry.npmjs.org` にフォールバックする。`.npmrc` ファイルは読み込まれない |

---

<a id="related-documentation"></a>
## 関連ドキュメント

- [コマンド](commands.md) - AI のスラッシュコマンド（`/opsx:propose`、`/opsx:apply` など）
- [ワークフロー](workflows.md) - よくあるパターンと、各コマンドをいつ使うか
- [カスタマイズ](customization.md) - カスタムスキーマとテンプレートを作成する
- [はじめに](getting-started.md) - 初回のセットアップガイド
