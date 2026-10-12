<a id="faq"></a>
# FAQ

よくある質問への簡潔な回答です。質問が実際には「何かが壊れている」というものであれば、[トラブルシューティング](troubleshooting.md) のほうが適したページです。用語の定義が知りたい場合は、[用語集](glossary.md) を参照してください。

<a id="the-basics"></a>
## 基本

<a id="what-is-openspec-in-one-sentence"></a>
### OpenSpec とは、一言でいうと何ですか？

コードを書く前に、何を作るのかについて、あなたと AI コーディングアシスタントが文書で合意するための軽量なレイヤーです。

<a id="why-would-i-want-that"></a>
### なぜそれが必要なのですか？

AI アシスタントは、間違っているときでも自信満々だからです。要件がチャットのスレッドの中にしかないと、AI は隙間を推測で埋め、あなたはコードができてから初めてそれに気づきます。OpenSpec は、間違いを安く直せる、より早い段階に合意を移します。詳しい説明は [コア概念の概要](overview.md) を参照してください。

<a id="do-i-have-to-use-it-for-everything"></a>
### すべての作業で使う必要がありますか？

いいえ。合意が重要な場面、つまり些細ではないほとんどの作業で使ってください。1文字の誤字修正に手順をかけても割に合わないでしょうが、それで構いません。

<a id="can-i-use-it-on-a-big-existing-codebase-or-only-new-projects"></a>
### 大規模な既存コードベースでも使えますか？ それとも新規プロジェクトだけですか？

既存のコードベースこそが主役です。OpenSpec はブラウンフィールドを第一に考えています。アプリ全体を最初にドキュメント化する必要はありません。各変更が触れる部分についてだけ仕様を書けば、実際に行う作業の周りに、時間をかけて仕様が埋まっていきます。専用のガイドがあります: [既存プロジェクトでの OpenSpec の利用](existing-projects.md)。

<a id="is-it-tied-to-one-ai-tool"></a>
### 特定の AI ツールに縛られますか？

いいえ。OpenSpec は、Claude Code、Cursor、Devin Desktop、GitHub Copilot、Gemini CLI、Codex など、30以上のアシスタントで動作します。完全な一覧とツールごとの詳細は [対応ツール](supported-tools.md) にあります。

<a id="running-commands"></a>
## コマンドの実行

<a id="where-do-i-type-opsxpropose"></a>
### `/opsx:propose` はどこに入力しますか？

ターミナルではなく、AI アシスタントのチャットです。これは最もよくある混乱のポイントなので、専用のページがあります: [コマンドの仕組み](how-commands-work.md)。簡潔に言うと、`openspec ...` はターミナルで、`/opsx:...` はチャットで実行します。

<a id="how-do-i-start-interactive-mode"></a>
### 「インタラクティブモード」はどうやって開始しますか？

開始すべき独立したモードはありません。AI アシスタントをいつもどおり開き、そのチャットにスラッシュコマンドを入力します。スラッシュコマンドが、OpenSpec に「入る」方法です。（本当にインタラクティブなターミナル機能は `openspec view` だけで、仕様や変更を閲覧するダッシュボードです。）詳しくは [コマンドの仕組み](how-commands-work.md) を参照してください。

<a id="i-typed-a-slash-command-and-nothing-happened-why"></a>
### スラッシュコマンドを入力しても何も起きません。なぜですか？

ターミナルに入力してしまった、ツールが登録していない表記を使った、あるいはコマンドがまだインストールされていない、のいずれかである可能性が高いです。ファイルが存在しない場合、またはツールを一度もセットアップしていない場合は、`openspec init` を実行してください。`openspec update` は、すでに存在するファイルを更新するだけです。そのあとアシスタントを再起動し、「Getting started」の下に表示された形式を使ってください。[呼び出し方](supported-tools.md#how-to-invoke) を参照してください。完全なチェックリストは [トラブルシューティング](troubleshooting.md#commands-dont-show-up) にあります。

<a id="why-is-the-syntax-opsxpropose-in-one-tool-and-opsx-propose-in-another"></a>
### なぜツールによって構文が `/opsx:propose` だったり `/opsx-propose` だったりするのですか？

AI ツールごとにカスタムコマンドの扱いが少しずつ異なり、OpenSpec は、ツールが読み込む、書き込んだファイルに合わせて表記します。`opsx-propose.md` という名前のコマンドファイルは `/opsx-propose` と入力し、`commands/opsx/` の下に置かれたものは `/opsx:propose` と入力します。コマンドの代わりにスキルを使うツールは、スキル名を使います。Codex では `$openspec-propose`、Kimi Code では `/skill:openspec-propose` が必要です。`openspec init` の「Getting started」の行には、選択したツールに適した形式がすでに表示されます。完全な表は [呼び出し方](supported-tools.md#how-to-invoke) にあります。

<a id="whats-the-difference-between-a-skill-and-a-command"></a>
### スキルとコマンドの違いは何ですか？

どちらも、アシスタントがワークフローを実行できるよう OpenSpec が書き込むファイルです。スキル（`.../skills/openspec-*/SKILL.md`）は新しいツール横断の標準で、コマンド（`.../commands/opsx-*`）は古い、ツールごとのスラッシュファイルです。選ぶ必要はありません。スラッシュコマンドを入力するだけで、OpenSpec がツールが使う方をインストールします。

<a id="the-workflow"></a>
## ワークフロー

<a id="where-should-i-start-if-im-not-sure-what-to-build"></a>
### 何を作るべきか迷っている場合、どこから始めればよいですか？

`/opsx:explore` からです。リスクなしで使える思考パートナーで、コードベースを読み、選択肢を示し、曖昧な問題を、コードを書く前に具体的な計画に変えます。デフォルトのプロファイルに含まれているため、いつでも使えます。計画が明確になったら、`/opsx:propose` に引き継ぎます。これは、積極的な AI が自信満々に間違ったものを作るのを防ぐため、身につけるべき最良の習慣です。[まず探索する](explore.md) を参照してください。

<a id="whats-the-simplest-possible-flow"></a>
### 最もシンプルな流れは何ですか？

```text
/opsx:explore (optional)   then   /opsx:propose <what you want>   then   /opsx:apply   then   /opsx:archive
```

考えを整理するために探索し、計画を下書きするために提案し、実装するために apply を実行し、保管するためにアーカイブします。何がしたいかを正確にわかっているなら、探索は省略してください。

<a id="whats-the-difference-between-opsxpropose-and-opsxnew"></a>
### `/opsx:propose` と `/opsx:new` の違いは何ですか？

`/opsx:propose` はデフォルトの1ステップのコマンドで、変更を作成し、すべての計画用成果物を一度に下書きします。`/opsx:new` は拡張コマンドセットの一部で、空の変更の骨組みを作るだけです。成果物は `/opsx:continue` で1つずつ（または `/opsx:ff` ですべて一度に）作成します。ステップごとに制御したい場合以外は、propose を使ってください。[コマンド](commands.md) を参照してください。

<a id="what-are-core-and-expanded-profiles"></a>
### `core` と拡張プロファイルとは何ですか？

プロファイルは、どのスラッシュコマンドをインストールするかを決めます。**コア**（デフォルト）には、`propose`、`explore`、`apply`、`update`、`sync`、`archive` が含まれます。**拡張**セットでは、より細かい制御のために、`new`、`continue`、`ff`、`verify`、`bulk-archive`、`onboard` が加わります。`openspec config profile` で切り替え、`openspec update` で適用します。

<a id="do-i-need-to-run-opsxsync"></a>
### `/opsx:sync` を実行する必要はありますか？

通常は不要です。同期は、変更のデルタ仕様をメインの仕様にマージするもので、`/opsx:archive` が代わりに実行するかどうかを尋ねます。アーカイブの前に仕様をマージしたい場合（たとえば長期にわたる変更など）にだけ、手動で同期を実行してください。[コマンド](commands.md#opsxsync) を参照してください。

<a id="how-do-i-edit-a-proposal-spec-or-task-after-ive-started"></a>
### 作業を始めたあとで、提案、仕様、タスクを編集するにはどうすればよいですか？

ファイルを編集するだけです。すべての成果物は `openspec/changes/<name>/` にあるプレーンな Markdown で、ロックされたフェーズも特別な編集モードもありません。手で変更するか、AI に修正を依頼し（「設計をキューを使う内容に更新して」）、そのまま続けてください。AI は常に現在のファイルの内容に基づいて作業します。完全なガイド: [変更の編集と反復](editing-changes.md)。

<a id="can-i-go-back-and-change-the-plan-after-implementing-some-of-it"></a>
### 一部を実装した後で、戻って計画を変更できますか？

はい、いつでもできます。ワークフローは流動的なので、レビューや編集は、閉め出されるフェーズではありません。成果物を編集して、続けてください。コードが計画と一致しているかを体系的に確認したい場合は、`/opsx:verify` を実行してください。[変更の編集と反復](editing-changes.md#how-do-i-go-back-to-review-after-implementing) を参照してください。

<a id="i-edited-the-code-by-hand-how-do-i-reconcile-it-with-the-spec"></a>
### コードを手で編集しました。仕様とどう整合させればよいですか？

アーカイブは仕様を公式な記録にするので、アーカイブする前に同期を取り戻してください。コードが現在正しいなら、リリースした内容に合わせてデルタ仕様を更新します。仕様が正しいなら、コードが一致するまで実装を続けます。`/opsx:verify` が不一致を洗い出します。[変更の編集と反復](editing-changes.md#i-edited-the-code-by-hand-how-do-i-reconcile-that-with-openspec) を参照してください。

<a id="when-should-i-update-an-existing-change-versus-start-a-new-one"></a>
### 既存の変更を更新すべきか、新しい変更を始めるべきか、どう判断しますか？

同じ作業を洗練させるなら更新します。意図が根本的に変わった場合や、スコープが膨らんで別の作業になった場合は、新しく始めます。判断のフローチャートと例が [ワークフロー](workflows.md#when-to-update-vs-start-fresh) にあります。

<a id="what-if-my-session-runs-out-of-context-or-requirements-change-mid-implementation"></a>
### セッションのコンテキストが尽きた場合や、実装の途中で要件が変わった場合は？

ここで仕様の価値が発揮されます。計画がファイルに（チャット履歴だけでなく）保存されているため、コンテキストをクリアし、新しい AI セッションを開始して、`/opsx:apply` で再開できます。成果物を読み込み、最初の未チェックのタスクから再開します。要件が変わった場合は、新しい現実に合わせて成果物を編集し、続行してください。コンテキストウィンドウをきれいに保つことでも結果が良くなるため、実装の前にクリアしてください。

<a id="should-i-commit-the-openspec-folder-to-git"></a>
### `openspec/` フォルダは git にコミットすべきですか？

はい。仕様、アクティブな変更、アーカイブは、プロジェクト履歴の一部です。他のソースと同様にコミットしてください。特にアーカイブは、システムがなぜ今のように動作するのかについての、長く残る記録になります。

<a id="specs-and-changes"></a>
## 仕様と変更

<a id="what-goes-in-a-spec-versus-a-design"></a>
### 仕様と設計には、それぞれ何を書きますか？

仕様は、観測可能な振る舞いを記述します。システムが何をするのか、その入力、出力、エラー条件です。設計は、どのように作るかを記述します。技術的アプローチ、アーキテクチャ上の判断、ファイルの変更です。外部から見える振る舞いを変えずに実装を変えられるのであれば、それは仕様ではなく設計に属します。詳しくは [概念](concepts.md#what-a-spec-is-and-is-not) を参照してください。

<a id="whats-a-delta-spec"></a>
### デルタ仕様とは何ですか？

仕様全体を再度記述する代わりに、`ADDED`、`MODIFIED`、`REMOVED` のセクションを使って、何が変わるかだけを記述する仕様です。OpenSpec が既存システムへの編集をきれいに扱う方法です。[概念](concepts.md#delta-specs) を参照してください。

<a id="where-do-archived-changes-go"></a>
### アーカイブされた変更はどこに行きますか？

`openspec/changes/archive/YYYY-MM-DD-<name>/` に、すべての変更の成果物を保持したまま移動します。変更はアクティブなリストから外れます。`retire_capabilities: true` を明示的に宣言した変更は、機能の最後の要件を削除する際に、メインの機能仕様を削除することもできます。

<a id="configuration-and-customization"></a>
## 設定とカスタマイズ

<a id="how-do-i-tell-the-ai-about-my-tech-stack"></a>
### 技術スタックを AI に伝えるにはどうすればよいですか？

`openspec/config.yaml` の `context:` に記述します。そのテキストはすべての計画リクエストに注入されるため、AI は常にスタックと規約を把握します。[カスタマイズ](customization.md#project-configuration) を参照してください。

<a id="can-i-generate-specs-in-a-language-other-than-english"></a>
### 英語以外の言語で仕様を生成できますか？

はい。設定の `context:` に言語の指示を追加してください。[多言語対応](multi-language.md) に、複数の言語のコピー可能なスニペットがあります。

<a id="can-i-change-the-workflow-itself"></a>
### ワークフロー自体を変更できますか？

はい、カスタムスキーマを使います。スキーマは、どの成果物が存在し、それらが互いにどう依存するかを定義します。`openspec schema fork spec-driven my-workflow` でデフォルトをフォークして、編集してください。[カスタマイズ](customization.md#custom-schemas) を参照してください。

<a id="models-privacy-and-upgrades"></a>
## モデル、プライバシー、アップグレード

<a id="which-ai-model-should-i-use"></a>
### どの AI モデルを使うべきですか？

OpenSpec は、推論能力の高いモデルで最もよく機能します。README では、計画と実装の両方に Codex 5.5 や Opus 4.7 のようなモデルを推奨しています。また、コンテキストウィンドウをきれいに保ってください。最良の結果を得るために、実装の前にクリアしましょう。

<a id="does-openspec-collect-data"></a>
### OpenSpec はデータを収集しますか？

匿名の利用統計を収集します。収集するのは、コマンド名とバージョンだけです。引数、パス、内容、個人データは含まれず、CI では自動的に無効になります。オプトアウトするには、`export OPENSPEC_TELEMETRY=0` または `export DO_NOT_TRACK=1` を設定してください。

<a id="how-do-i-upgrade"></a>
### アップグレードするにはどうすればよいですか？

2ステップです。パッケージをアップグレードし（`npm install -g @fission-ai/openspec@latest`）、各プロジェクト内で `openspec update` を実行して、生成されたスキルとコマンドを更新します。

<a id="how-do-i-uninstall-openspec"></a>
### OpenSpec をアンインストールするにはどうすればよいですか？

アンインストールコマンドはありません。グローバルパッケージとプロジェクト内のファイルにすぎないからです。パッケージを削除し（`npm uninstall -g @fission-ai/openspec`）、必要に応じて `openspec/` ディレクトリと生成されたツールファイルを削除してください。残しておいても安全なものを含む手順は、[インストール: アンインストール](installation.md#uninstalling) にあります。

<a id="getting-help"></a>
## ヘルプを得る

<a id="where-do-i-ask-questions-or-report-bugs"></a>
### 質問やバグ報告はどこにすればよいですか？

- **Discord:** [discord.gg/YctCnvvshC](https://discord.gg/YctCnvvshC)
- **GitHub Issues:** [github.com/Fission-AI/OpenSpec/issues](https://github.com/Fission-AI/OpenSpec/issues)
- **ターミナルから:** `openspec feedback "your message"` で GitHub の issue が作成されます。

<a id="these-docs-are-wrong-or-confusing-what-do-i-do"></a>
### このドキュメントが間違っている、またはわかりにくい場合は？

お知らせいただくか、修正してください。ドキュメントの PR は歓迎され、高く評価されます。issue を作成するか、プルリクエストを送ってください。
