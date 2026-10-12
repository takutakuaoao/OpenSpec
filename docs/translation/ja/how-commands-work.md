<a id="how-commands-work"></a>
# コマンドの仕組み

**知っておくべき唯一のこと: OpenSpec には2種類のコマンドがあり、それぞれ異なる場所で実行します。**

- `openspec ...` コマンドは、**ターミナル**で実行します。（例: `openspec init`）
- `/opsx:...` コマンドは、**AI アシスタントのチャット**で実行します。（例: `/opsx:propose`）

ターミナルに `/opsx:propose` と入力して何も起きなかったなら、このページがその理由です。OpenSpec の話しかける相手を間違えています。スラッシュコマンドはターミナルのコマンドではありません。「ログインフォームを追加して」と普段入力するのと同じチャットボックスで、AI コーディングアシスタントに与える指示です。

この区別は、新規ユーザーが最もよくつまずくポイントなので、はっきりさせておきましょう。

<a id="the-two-halves"></a>
## 2つの側面

OpenSpec は、2つの顔を持つ1つのプロジェクトです。

**CLI（ターミナル側）。** インストールしてシェルから実行する `openspec` というプログラムです。プロジェクトのセットアップ、変更の一覧表示と検証、ダッシュボードの表示、完了した作業のアーカイブを行います。iTerm、VS Code のターミナル、PowerShell など、`git` や `npm` を実行する場所ならどこでも入力できます。

```bash
openspec init        # set up OpenSpec in this project
openspec list        # see active changes
openspec view        # open the interactive dashboard
```

**スラッシュコマンド（チャット側）。** `/opsx:propose` や `/opsx:apply` のような短いコマンドで、AI アシスタントに入力します。これらは AI に OpenSpec のワークフロー（提案の下書き、仕様の作成、タスクリストに沿った実装、完了時のアーカイブ）に従うよう指示します。Claude Code、Cursor、Devin Desktop、Copilot など、お使いのアシスタントに入力します。

```text
/opsx:propose add-dark-mode    (typed in your AI chat)
/opsx:apply                    (typed in your AI chat)
/opsx:archive                  (typed in your AI chat)
```

考え方を1枚の図で表すと次のとおりです。

```text
        あなたのターミナル                    あなたの AI アシスタントのチャット
   ┌──────────────────────┐               ┌──────────────────────────────┐
   │  $ openspec init     │   installs    │  /opsx:propose add-dark-mode  │
   │  $ openspec list     │  ──────────►  │  /opsx:apply                  │
   │  $ openspec view     │   commands    │  /opsx:archive                │
   └──────────────────────┘    & skills   └──────────────────────────────┘
        openspec はここで実行                   /opsx:* はここで実行
```

矢印に注目してください。ターミナルで `openspec init` を実行することが、スラッシュコマンドを AI ツールに*インストールする*ことになります。ターミナル側がチャット側をセットアップするのです。その後の日常的な操作は、主にチャットで行います。

<a id="how-do-i-start-interactive-mode"></a>
## 「インタラクティブモードはどうやって開始するの？」

**開始すべき、独立したインタラクティブモードはありません。** この質問はとても多いので、はっきり答えておきます。

特別な OpenSpec モードに入る必要はありません。いつもどおり AI コーディングアシスタントを開き、チャットにスラッシュコマンドを入力するだけです。スラッシュコマンドこそが、OpenSpec に「入る」方法です。アシスタントがそれを認識し、対応する OpenSpec のスキルを読み込み、ワークフローに従い始めます。

具体的な手順は次のとおりです。

1. プロジェクトで AI コーディングアシスタント（Claude Code、Cursor、Devin Desktop など）を開きます。
2. そのチャット、つまり他のリクエストを入力するのと同じ場所に `/opsx:propose` と入力します。
3. オートコンプリートを確認します。OpenSpec がインストールされていれば、スラッシュを入力した時点で `/opsx:propose`、`/opsx:apply` などが表示されます。

以上です。切り替えるモードも、起動するデーモンも、別ウィンドウもありません。

ターミナルには、本当にインタラクティブなものが1つあります。`openspec view` です。仕様や変更を閲覧するためのダッシュボードを開きます。ただしこれはビューアであり、提案や実装に使うものではありません。実装はチャットのスラッシュコマンドで行います。

<a id="why-this-split-exists"></a>
## この分離が存在する理由

これを理解しておく価値があるのは、OpenSpec が30以上の異なる AI ツールで動作する理由を説明してくれるからです。

CLI は**エンジン**です。変更フォルダがどのような姿か、どの成果物がどれに依存するか、デルタ仕様を信頼できる唯一の情報源にどうマージするか、といったルールを知っています。どこでも同じです。

スラッシュコマンドは**ハンドル**であり、AI ツールごとに微妙に異なります。Claude Code はコマンドと呼びます。Cursor や Devin Desktop には独自の形式があります。スキルと呼ぶツールもあります。`openspec init` を実行すると、OpenSpec は選択した各ツールに適した種類のファイルを生成するため、どのアシスタントを好むかにかかわらず、同じ `/opsx:propose` の意図が機能します。

この設計の強みは、ワークフローを一度学べば、ツールをまたいで使えることです。トレードオフは、コマンドの正確な構文がツール間で少し異なる場合があることで、次の節で説明します。

<a id="slash-command-syntax-by-tool"></a>
## ツール別のスラッシュコマンド構文

意図はどこでも同じです。表記は、ツールが読み込むファイルに従います。

| ツールのコマンドファイル | 入力方法 | ツールの例 |
|--------------------------|-----------------|---------------|
| `.../commands/opsx/<id>.*` | `/opsx:propose` | Claude Code, Gemini CLI, Crush |
| `.../opsx-<id>.*` | `/opsx-propose` | Cursor, GitHub Copilot (IDE), Devin Desktop, Trae, Oh My Pi |
| `.amazonq/prompts/opsx-<id>.md` | `@opsx-propose` | Amazon Q Developer |
| なし — スキルのみ | `/openspec-propose` | CodeArts, ForgeCode, Hermes, Mistral Vibe, Zed Agent, 共有 `.agents` |
| なし — Kimi Code | `/skill:openspec-propose` | Kimi Code |
| なし — Codex CLI | `$openspec-propose` | Codex |

Devin は2つの行にまたがる唯一のツールです。Devin Desktop は
`.devin/workflows/` を読み込むため、そこでは `/opsx-propose` が使えます。一方、[Devin Local は
対応していない](https://docs.devin.ai/desktop/devin-local)ため、そのエージェントでは代わりに
`/openspec-propose` スキルを使ってください。OpenSpec が
`.devin/skills/` に書き込むスキルは両方で動作するため、スキルは互いにスキル名で
参照し合っています。

すべてのツールは [呼び出し方](supported-tools.md#how-to-invoke) に一覧があり、
その表が正式なものです。2つの行はそもそもスラッシュコマンドではありません。Amazon Q
はファイルをプロンプトライブラリに読み込み、`@` で呼び出します。また、最後の3行は
コマンド ID ではなく*スキル*名を使います（`/opsx:apply` は
`openspec-apply-change` スキルに対応します）。

迷ったときは、`openspec init` が出力した「Getting started」の行を読んでください。そこには、
ツールが登録した形式がすでに使われています。スラッシュを入力してオートコンプリートを
確認するのも有効です（スラッシュコマンドを表示するツールに限ります）。

<a id="how-the-commands-got-there-skills-and-commands"></a>
## コマンドがどのように届くか: スキルとコマンド

`openspec init`（または `openspec update`）を実行すると、OpenSpec は小さなファイルをプロジェクトに書き込み、AI ツールがワークフローを見つけられるようにします。ツールと設定によって、これらは**スキル**、**コマンド**、またはその両方になります。

- **スキル**は、`.claude/skills/openspec-*/SKILL.md` のような場所にあります。これは、新しく登場しつつあるツール横断の標準で、アシスタントが自動検出する指示のフォルダです。
- **コマンド**は、`.cursor/commands/opsx-<id>.md` や `.claude/commands/opsx/<id>.md` のような場所にあります。レイアウトはツール側のもので、コマンドの入力方法もそれによって決まります。これらは、より古い、ツールごとのスラッシュコマンドファイルです。Codex にはコマンドファイルは生成されません。`.agents/skills/openspec-*` を使用してください。

どちらをツールが使うかを気にする必要はありません。スラッシュコマンドを入力すれば動作します。ただし、これらのファイルが存在することを知っていると、問題が起きたときに役立ちます。コマンドが消えた場合、たいていはこれらのファイルが欠落しているか古くなっているのが原因で、`openspec update` で再生成されます。

ツールごとの正確なパスは [対応ツール](supported-tools.md) を、スキルが従来のコマンドのみの方式にどう置き換わったかは [移行ガイド](migration-guide.md) を参照してください。

<a id="confirming-its-installed"></a>
## インストールされているかの確認

素早く確認できる順に、簡易チェックを示します。

1. **AI チャットでスラッシュを入力する。** `/opsx` と入力し始めて、オートコンプリートの候補が表示されるかを確認します。表示されれば準備完了です。スキルのみのツール（Codex、Kimi Code、CodeArts、ForgeCode、Hermes、Mistral Vibe、Zed Agent、共有 `.agents` ターゲット）では、正常にインストールされていても `/opsx` は補完されません。代わりに、上の表のスキル名を試してください。
2. **ファイルを探す。** Claude Code の場合は、`.claude/skills/` に `openspec-*` フォルダがあることを確認します。他のツールは独自のディレクトリを使用します（[対応ツール](supported-tools.md) に一覧があります）。
3. **セットアップを再実行する。** プロジェクトのルートから `openspec update` を実行します。設定したすべてのツールのスキルファイルとコマンドファイルが再生成されます。
4. **アシスタントを再起動する。** 多くのツールは起動時にスキルやコマンドをスキャンするため、新しいウィンドウで解決することがあります。

<a id="which-commands-do-i-even-have"></a>
## どのコマンドが使えるのか？

デフォルトでは、OpenSpec は**コア**セットのスラッシュコマンドをインストールします。

- `/opsx:explore`: 変更を決める前に、AI とアイデアを整理する（迷っているときの最初の一歩に最適）
- `/opsx:propose`: 変更を作成し、すべての計画用成果物を一度に下書きする
- `/opsx:apply`: タスクリストを順に進めて、変更を実装する
- `/opsx:update`: 変更の計画用成果物を修正し、整合性を保つ
- `/opsx:sync`: 変更の仕様更新を、メインの仕様にマージする（通常は自動）
- `/opsx:archive`: 変更を完了して保管する

おすすめの基本のリズム: 何をするか考えているときは `explore`、そのあと `propose`、`apply`、`archive`。この最初のステップが報われる理由は、[まず探索する](explore.md) ガイドで説明しています。

より細かく制御したい人向けの**拡張**セットもあります（`/opsx:new`、`/opsx:continue`、`/opsx:ff`、`/opsx:verify`、`/opsx:bulk-archive`、`/opsx:onboard`）。`openspec config profile` で有効にし、`openspec update` で適用します。

初めてですか？ `/opsx:onboard`（拡張セットに含まれます）は、あなた自身のコードベースで、完全な変更の流れを各ステップを説明しながら案内してくれます。最もやさしい導入です。

各コマンドの詳細は [コマンド](commands.md) を、どれを使うべきかは [ワークフロー](workflows.md) を参照してください。

<a id="a-clean-first-run"></a>
## クリーンな最初の実行

まとめとして、各ステップが行われる場所のラベル付きで、全体の流れを示します。

```text
TERMINAL   $ npm install -g @fission-ai/openspec@latest
TERMINAL   $ cd your-project
TERMINAL   $ openspec init
              (AI ツールにスラッシュコマンドをインストールする)

AI CHAT      /opsx:explore
              (任意: 先に AI とアイデアを整理する)

AI CHAT      /opsx:propose add-dark-mode
              (AI が提案、仕様、設計、タスクを下書きする)

AI CHAT      /opsx:apply
              (AI が実装し、タスクにチェックを付ける)

AI CHAT      /opsx:archive
              (変更が仕様にマージされ、保管される)
```

セットアップのターミナル操作が2つ。そのあとはチャットが中心です。それがリズムです。

<a id="related"></a>
## 関連ドキュメント

- [はじめに](getting-started.md): 最初の変更の完全なウォークスルー
- [コマンド](commands.md): すべてのスラッシュコマンドの詳細
- [CLI](cli.md): すべてのターミナルコマンドの詳細
- [対応ツール](supported-tools.md): ツールごとの構文とファイルの場所
- [FAQ](faq.md): その他の簡潔な回答
- [トラブルシューティング](troubleshooting.md): コマンドが表示されない場合の対処法
