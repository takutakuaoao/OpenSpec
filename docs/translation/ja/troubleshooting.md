<a id="troubleshooting"></a>
# トラブルシューティング

具体的な問題に対する、具体的な対処法です。各項目では、症状を示し、考えられる原因を一文で説明し、対処法を示します。ここに問題が見つからない場合は、[FAQ](faq.md) が役立つかもしれません。[Discord](https://discord.gg/YctCnvvshC) なら間違いなく力になってくれます。

<a id="installation-and-setup"></a>
## インストールとセットアップ

<a id="openspec-command-not-found"></a>
### `openspec: command not found`

CLI がインストールされていないか、シェルから見つけられません。グローバルにインストールして、確認します。

```bash
npm install -g @fission-ai/openspec@latest
openspec --version
```

インストールしたのに見つからない場合は、グローバルな npm の bin ディレクトリが `PATH` に含まれていない可能性があります。`npm prefix -g` を実行して、グローバルパッケージの場所を確認してください。macOS と Linux では、実行ファイルはそのディレクトリの `bin/` にあり、Windows ではそのディレクトリ直下にあります。そのパスが `PATH` に含まれていることを確認してください。（`npm bin -g` は npm 9 で削除されました。）

[AI アシスタントによるインストール](installation.md#install-with-your-ai-assistant) を使った場合、ここで手が止まるのは想定どおりです。そのプロンプトは、アシスタントにシェルの起動ファイルを自分で編集させるのではなく、`PATH` の変更内容を示すよう指示しています。

<a id="requires-nodejs-20190-or-higher"></a>
### 「Requires Node.js 20.19.0 or higher」

OpenSpec は Node 20.19.0 以上で動作します。バージョンを確認し、必要に応じてアップグレードしてください。

```bash
node --version
```

bun で OpenSpec をインストールした場合も、OpenSpec は Node 上で*動作する*ため、いずれにせよ Node 20.19.0 以上が `PATH` 上で使える必要があります。[インストール](installation.md) を参照してください。

<a id="openspec-init-didnt-configure-my-ai-tool"></a>
### `openspec init` で AI ツールが設定されなかった

init は、セットアップするツールを尋ねます。ツールを選択しなかった場合や、別のツールを追加したい場合は、もう一度実行するか、非対話形式を使ってください。

```bash
openspec init --tools claude,cursor
```

ツール ID の完全な一覧は [対応ツール](supported-tools.md) にあります。すべてを対象にするには `--tools all`、ツールのセットアップをスキップするには `--tools none` を使います。

<a id="commands-dont-show-up"></a>
## コマンドが表示されない

`/opsx:propose`（またはお使いのツールでの同等のもの）が表示されない、または何も起きない場合は、次のリストを上から順に確認してください。確認が速い順に並んでいます。

1. **場所を間違えている可能性があります。** スラッシュコマンドは、ターミナルではなく、AI アシスタントのチャットに入力します。シェルに `/opsx:propose` と入力したのであれば、それが原因です。[コマンドの仕組み](how-commands-work.md) を参照してください。

2. **ファイルを再生成する。** プロジェクトのルートから次を実行します。

   ```bash
   openspec update
   ```

   これにより、設定済みのすべてのツールのスキルファイルとコマンドファイルが書き直されます。

   指示ファイルは*インストール済みの* CLI から生成されるため、古い CLI では、新しいワークフローが書き込まれないまま、すべて最新だと報告されます。`openspec update` はこれを検出してアップグレードを提案するようになりました。提案が表示されたら受け入れてください。

3. **アシスタントを再起動する。** ほとんどのツールは起動時にスキルとコマンドをスキャンします。新しいウィンドウで解決することがよくあります。

4. **ファイルが存在することを確認する。** Claude Code の場合は、`.claude/skills/` に `openspec-*` フォルダがあることを確認します。他のツールは独自のディレクトリを使用します。すべて [対応ツール](supported-tools.md) に記載されています。

5. **このプロジェクトを初期化したことを確認する。** スキルはプロジェクトごとに書き込まれます。リポジトリをクローンした場合やフォルダを切り替えた場合は、そこで `openspec init`（または `openspec update`）を実行してください。

6. **ツールがコマンドファイルに対応しているか確認する。** Codex、CodeArts、ForgeCode、Hermes、Kimi Code、Mistral Vibe、Zed Agent、共有 `.agents` ターゲットには、`opsx-*` コマンドファイルが生成されません。代わりにスキルベースの呼び出しを使うため、これらでは `/opsx` が補完されることはありません。Codex では `$openspec-propose`、Kimi Code では `/skill:openspec-propose`、それ以外では `/openspec-propose` と入力してください。共有 `.agents` ターゲットはベンダー中立なので、`/openspec-propose` は一般的な形式ですが、保証されたものではありません。アシスタントが反応しない場合は、そのアシスタント自身のドキュメントで、スキルの呼び出し方を確認してください。Amazon Q にはコマンドファイルが生成されますが、それらはスラッシュメニューではなく、プロンプトライブラリに読み込まれます。そこでは `/opsx` ではなく、`@opsx-propose` と入力してください。すべてのツールの形式は [呼び出し方](supported-tools.md#how-to-invoke) に一覧があります。

<a id="working-with-changes"></a>
## 変更の操作

<a id="change-not-found"></a>
### 「Change not found」

コマンドが、どの変更を指しているのか判断できませんでした。明示的に名前を指定するか、存在するものを確認してください。

```bash
openspec list                    # see active changes
/opsx:apply add-dark-mode        # name the change in chat
```

正しいプロジェクトディレクトリにいることも確認してください。

<a id="no-artifacts-ready"></a>
### 「No artifacts ready」

すべての成果物が、すでに作成済みか、依存関係の待ちでブロックされています。何がブロックしているかを確認します。

```bash
openspec status --change <name>
```

その後、不足している依存先を先に作成してください。順序を思い出してください。proposal が specs と design を可能にし、specs と design が揃うと tasks が可能になります。

<a id="openspec-validate-reports-warnings-or-errors"></a>
### `openspec validate` が警告やエラーを報告する

検証は、仕様と変更に構造上の問題がないかをチェックします。メッセージを読んでください。ファイルと問題が示されています。

```bash
openspec validate <name>           # validate one item
openspec validate --all            # validate everything
openspec validate --all --strict   # stricter checks, good for CI
openspec validate --archived       # fail if archived changes have unchecked tasks
```

よくある原因は、必須セクションの欠落（シナリオのない仕様など）や、不正な形式のデルタヘッダーです。ファイルを修正して再実行してください。出力形式は [CLI リファレンス](cli.md#openspec-validate) に記載されています。

1つのメッセージについては、個別に説明しておきます。

```text
MODIFIED "<requirement>" omits scenario(s) the current spec still has: "<scenario>"
```

`MODIFIED` の要件は、要件ブロック全体を置き換えるため、編集したシナリオだけでなく、変更後も残るすべてのシナリオを含めなければなりません。名前が示されたシナリオを `openspec/specs/<capability-path>/spec.md` からデルタにコピーし直してください。パス内のドメインディレクトリはそのままにします。これは、他の人の変更が同じ要件にシナリオを追加した後に、古い変更で表示されることがよくあります。アーカイブはどちらの場合もその変更を拒否しますが、検証は実装する前にそれを知らせてくれるようになりました。

<a id="the-ai-created-incomplete-or-wrong-artifacts"></a>
### AI が不完全な、または間違った成果物を作成した

AI に十分な文脈がありませんでした。いくつかの手段が役立ちます。

- `openspec/config.yaml` にプロジェクトのコンテキストを追加し、スタックや規約がすべてのリクエストに注入されるようにします。[カスタマイズ](customization.md#project-configuration) を参照してください。
- たとえば仕様にだけ適用されるガイダンスのために、成果物ごとの `rules:` を追加します。
- 提案するときに、より詳細な説明を与えます。
- `/opsx:ff` ですべてを一度に作成するのではなく、拡張コマンドの `/opsx:continue` を使って、成果物を1つずつ作成し、それぞれをレビューします。

<a id="archive-wont-finish-or-warns-about-incomplete-tasks"></a>
### アーカイブが完了しない、または未完了のタスクについて警告される

アーカイブは、未完了のタスクがあっても*ブロック*しませんが、通常アーカイブは作業の完了を意味するため、警告します。意図的にタスクを残している場合（部分的な変更を保管する場合）は、そのまま進めてください。そうでなければ、先にタスクを完了させてください。アーカイブは、まだ同期していない場合に、デルタ仕様をメインの仕様に同期することも提案します。特別な理由がない限り、「はい」と答えてください。

<a id="user-force-closed-the-prompt-with-0-null"></a>
### 「User force closed the prompt with 0 null」

質問に答えられない環境で、何かが `openspec archive` を実行しました。ツールから呼び出す AI エージェント、CI ジョブ、標準入力が閉じられたシェルなどです。アーカイブは最大3回の確認を行いますが、答えられない確認は、以前はその生のメッセージで失敗していました。

`--yes` を渡して、あらかじめ回答しておきます。

```bash
openspec archive <change-name> --yes
```

すでに渡していたフラグはそのまま維持してください。`--skip-specs` や `--no-validate` はアーカイブの動作を変えるため、`--yes` だけで再実行すると、同じコマンドにはなりません。現在のバージョンでは、フラグ名が示され、貼り付けられる `Fix:` 行が出力されます。リストから選択するつもりだった場合は、変更名を明示的に渡してください。選択画面にも回答が必要だからです。

一方、アーカイブの出力をファイルにリダイレクトしたりツールにキャプチャさせたりしながら、回答をパイプで渡した場合（`printf 'y\n' | openspec archive …`）、古いバージョンでは、プロンプトの描画中にターミナルのエスケープコードがそのキャプチャに書き込まれ、環境によってはファイルが大きく肥大化することがありました。現在のバージョンでは、標準出力がターミナルでない場合は、確認プロンプトをプレーンテキストとして読み取ります。また、引数なしの `openspec archive`（本来は対話的な変更選択画面を描画する）では、メニューをキャプチャに描画する代わりに、先に変更名を渡すよう求めます。いずれの場合も、リダイレクトされた実行やエージェントによる実行はクリーンなままです。`--yes`（と変更名）を渡すと、プロンプトは完全にスキップされます。

<a id="configuration"></a>
## 設定

<a id="my-configyaml-isnt-being-applied"></a>
### `config.yaml` が適用されない

よくある原因は3つです。

1. **ファイル名が違う。** `openspec/config.yaml` でなければなりません。`.yml` ではありません。
2. **YAML が不正。** 任意の YAML バリデータに通してください。CLI も、構文エラーを行番号付きで報告します。
3. **再起動が必要だと思っている。** 必要ありません。設定の変更はすぐに反映されます。

<a id="unknown-artifact-id-in-rules-x"></a>
### 「Unknown artifact ID in rules: X」

`rules:` の下のキーが、スキーマ内のどの成果物にも一致しません。デフォルトの `spec-driven` スキーマで有効な ID は、`proposal`、`specs`、`design`、`tasks` です。任意のスキーマの ID を確認するには、次を実行します。

```bash
openspec schemas --json
```

<a id="context-too-large"></a>
### 「Context too large」

`context:` フィールドは、すべてのリクエストに注入されるため、意図的に 50KB に制限されています。要約するか、長いドキュメントを貼り付ける代わりにリンクしてください。簡潔なコンテキストのほうが、結果もより良く、より速くなります。

<a id="schema-not-found"></a>
### 「Schema not found」

参照したスキーマ名が存在しません。利用可能なものを一覧表示し、綴りを確認してください。

```bash
openspec schemas                    # list available schemas
openspec schema which <name>        # see where a schema resolves from
openspec schema init <name>         # create a custom one
```

[カスタマイズ](customization.md#custom-schemas) を参照してください。

<a id="migration-from-the-legacy-workflow"></a>
## レガシーワークフローからの移行

<a id="legacy-files-detected-in-non-interactive-mode"></a>
### 「Legacy files detected in non-interactive mode」

CI や非対話型シェルで実行しており、OpenSpec がクリーンアップすべき古いファイルを見つけましたが、確認を求められません。自動的に承認してください。

```bash
openspec init --force
```

Codex の場合、OpenSpec は `$CODEX_HOME/prompts` または `~/.codex/prompts` にある、古い管理対象のプロンプトファイルを検出することがあります。そのクリーンアップは、OpenSpec が許可リストに載せたレガシーな Codex プロンプトのファイル名に限定され、非対話型の `openspec init` は、置き換え先の `.agents/skills/openspec-*` スキルが存在するファイルだけを削除します。非対話型の `openspec update` は、`--force` を渡さない限り、レガシーなクリーンアップをすべてそのままにします。

<a id="commands-didnt-appear-after-migrating"></a>
### 移行後にコマンドが表示されなかった

IDE を再起動してください。スキルは起動時に検出されます。それでも表示されない場合は、`openspec update` を実行し、[対応ツール](supported-tools.md) でファイルの場所を確認してください。

<a id="my-old-projectmd-wasnt-migrated"></a>
### 古い `project.md` が移行されなかった

それは意図的です。`project.md` には、あなたが書いたコンテキストが含まれている可能性があるため、OpenSpec がこれを自動で削除することはありません。有用な部分を `config.yaml` の `context:` セクションに移してから、ご自身で削除してください。[移行ガイド](migration-guide.md#migrating-projectmd-to-configyaml) で、この手順を、AI に要点の抽出を任せるためのプロンプトを含めて説明しています。

<a id="still-stuck"></a>
## それでも解決しない場合は？

- **Discord:** [discord.gg/YctCnvvshC](https://discord.gg/YctCnvvshC)
- **GitHub Issues:** [github.com/Fission-AI/OpenSpec/issues](https://github.com/Fission-AI/OpenSpec/issues)
- **ターミナルから:** `openspec feedback "what went wrong"` で issue が作成されます。

問題を報告する際は、OpenSpec のバージョン（`openspec --version`）、Node のバージョン（`node --version`）、使用している AI ツール、そして実行したコマンドとその出力を含めてください。そうすると、サポートがずっと速くなります。
