<a id="openspec-documentation"></a>
# OpenSpec ドキュメント

ようこそ。ここは OpenSpec に関するすべての情報の入り口です。

OpenSpec は、あなたと AI コーディングアシスタントが**コードを書く前に、何を作るのかを合意する**ためのツールです。変更内容を説明すると、AI が短い仕様とタスクリストの下書きを作成し、あなたとAIが同じ計画を見たうえで作業に入ります。作業の途中で「AI が間違ったものを作っていた」と気づくことはもうありません。

他に何も読まないとしても、次の2ページだけは読んでください。

1. [はじめに](getting-started.md): インストール、初期化、最初の変更のリリースまで。
2. [コマンドの仕組み](how-commands-work.md): `/opsx:propose` を実際にどこで入力するのか（ヒント: ターミナルではなく AI チャットです）。ほぼ全員が一度はここでつまずきます。

2つ目のページは見た目以上に重要です。OpenSpec には、ターミナルで実行するコマンドラインツールと、AI アシスタントに渡すスラッシュコマンドという2つの側面があります。どちらがどちらかを知っておくだけで、最もよくある混乱を避けられます。

> **最初に身につけたい最良の習慣: 何を作るべきか迷ったら `/opsx:explore` から始めましょう。** これはリスクなしで相談できる思考パートナーで、コードを読み、選択肢を比較し、漠然としたアイデアをコードを書く前に具体的な計画へと磨き上げてくれます。詳しくは [まず探索する](explore.md) ガイドをご覧ください。

<a id="pick-your-path"></a>
## 目的別ガイド

**完全な初心者です。** まず [はじめに](getting-started.md) を読み、続けて [コア概念の概要](overview.md) にざっと目を通してください。わからないことがあれば、[FAQ](faq.md) と [用語集](glossary.md) が役に立ちます。

**問題はあるが、計画はありません。** これは最もよくあるケースで、専用のガイドがあります: [まず探索する](explore.md)。何かに着手する前に `/opsx:explore` を使って AI と一緒に考えを整理しましょう。

**大規模な既存コードベースがあります。** すべてをドキュメント化する必要はありません。[既存プロジェクトでの OpenSpec の利用](existing-projects.md) では、大がかりな作業をせずに実際のブラウンフィールドのコードで始める方法を紹介しています。

**とにかく動かしたい。** [インストール](installation.md) し、`openspec init` を実行してから、[コマンドの仕組み](how-commands-work.md) を読んで、最初のスラッシュコマンドを正しい場所で実行できるようにしましょう。あるいは、[AI によるインストールプロンプト](installation.md#install-with-your-ai-assistant) でセットアップをアシスタントに任せることもできます。

**例から学びたい。** [例とレシピ](examples.md) では、小さな機能追加、バグ修正、リファクタリング、探索といった実際の変更を最初から最後まで順に紹介しています。

**AI が計画を作成しました。次はどうすれば？** それを読みましょう。[変更のレビュー](reviewing-changes.md) では、方向違いをまだ修正コストが低いうちに見つける2分間のチェック方法を、[良い仕様の書き方](writing-specs.md) では、承認するに値する計画の条件を説明しています。

**チームで作業しています。** [チームでの OpenSpec](team-workflow.md) では、変更をブランチやプルリクエストに対応させる方法と、チームメイトがコードの前に計画をレビューする方法を紹介しています。

**旧ワークフローから移行します。** [移行ガイド](migration-guide.md) では、何がなぜ変わったのかを説明し、既存の作業が安全であることを保証します。

**チームのプロセスに合わせてカスタマイズしたい。** [カスタマイズ](customization.md) では、プロジェクト設定、カスタムスキーマ、共有コンテキストを扱います。

**何かが壊れた。** [トラブルシューティング](troubleshooting.md) に、実際によく起きる障害とその対処法をまとめています。

<a id="the-whole-map"></a>
## 全体マップ

<a id="start-here"></a>
### まずはここから

| ドキュメント | 内容 |
|-----|-------------------|
| [はじめに](getting-started.md) | インストール、初期化、最初の変更を最初から最後まで実行する |
| [まず探索する](explore.md) | `/opsx:explore` を使って、決める前にアイデアを整理する |
| [コマンドの仕組み](how-commands-work.md) | スラッシュコマンドをどこで実行するか、「インタラクティブモード」とは何か、ターミナルとチャットの違い |
| [コア概念の概要](overview.md) | 仕様、変更、デルタ、アーカイブといった全体像を1ページで把握する |
| [インストール](installation.md) | npm、pnpm、yarn、bun、Nix、AI アシスタントにセットアップを任せるプロンプト、動作確認の方法 |

<a id="use-it-day-to-day"></a>
### 日常的に使う

| ドキュメント | 内容 |
|-----|-------------------|
| [ワークフロー](workflows.md) | よくあるパターンと、各コマンドを使うべき場面 |
| [例とレシピ](examples.md) | 実際の変更の完全なウォークスルー（コピーして使えます） |
| [良い仕様の書き方](writing-specs.md) | 優れた要件とシナリオの姿、変更の適切なサイズの決め方 |
| [変更のレビュー](reviewing-changes.md) | コードを書く前に、作成された計画を確認する2分間のチェック |
| [チームでの OpenSpec](team-workflow.md) | 変更をブランチ、プルリクエスト、レビューにどう組み込むか |
| [既存プロジェクトでの OpenSpec の利用](existing-projects.md) | 大規模なブラウンフィールドのコードベースに OpenSpec を導入する |
| [変更の編集と反復](editing-changes.md) | 成果物の更新、前の段階への戻り、手動編集の整合 |
| [コマンド](commands.md) | すべての `/opsx:*` スラッシュコマンドのリファレンス |
| [CLI](cli.md) | すべての `openspec` ターミナルコマンドのリファレンス |

<a id="understand-it-deeply"></a>
### 深く理解する

| ドキュメント | 内容 |
|-----|-------------------|
| [概念](concepts.md) | 仕様、変更、成果物、スキーマ、アーカイブの詳細な解説 |
| [OPSX ワークフロー](opsx.md) | ワークフローがフェーズ固定ではなく柔軟である理由と、アーキテクチャの詳細解説 |
| [用語集](glossary.md) | すべての用語を1か所にまとめた定義集 |

<a id="make-it-yours"></a>
### 自分用にカスタマイズする

| ドキュメント | 内容 |
|-----|-------------------|
| [カスタマイズ](customization.md) | プロジェクト設定、カスタムスキーマ、共有コンテキスト |
| [多言語対応](multi-language.md) | 英語以外の言語で成果物を生成する |
| [対応ツール](supported-tools.md) | OpenSpec が連携する30以上の AI ツールと、ファイルの配置先 |
| [コミュニティショーケース](community.md) | OpenSpec で、あるいは OpenSpec のために作られたプロジェクトやリソース |

<a id="when-you-need-help"></a>
### ヘルプが必要なとき

| ドキュメント | 内容 |
|-----|-------------------|
| [FAQ](faq.md) | よくある質問への簡潔な回答 |
| [トラブルシューティング](troubleshooting.md) | 具体的な障害への具体的な対処法 |
| [移行ガイド](migration-guide.md) | レガシーワークフローから OPSX への移行 |

<a id="coordinate-across-repos-beta"></a>
### 複数リポジトリをまたいで調整する（ベータ）

| ドキュメント | 内容 |
|-----|-------------------|
| [ストア: ユーザーガイド](stores-beta/user-guide.md) | 作業が複数のリポジトリやチームにまたがる場合に、計画を専用リポジトリで管理する |
| [エージェントコントラクト](agent-contract.md) | エージェントが操作する、機械可読な CLI インターフェース |

<a id="the-thirty-second-version"></a>
## 30秒でわかる流れ

```text
1. Install        npm install -g @fission-ai/openspec@latest
2. Initialize     cd your-project && openspec init
3. Explore        (in your AI chat)  /opsx:explore           ← optional, but a great habit
4. Propose        (in your AI chat)  /opsx:propose add-dark-mode
5. Build          (in your AI chat)  /opsx:apply
6. Archive        (in your AI chat)  /opsx:archive
```

手順1と2はターミナルで行います。残りは AI アシスタントのチャットで行います。この区別だけは覚えておく価値があり、その理由は [コマンドの仕組み](how-commands-work.md) で詳しく説明しています。手順3は任意ですが、迷ったときに `/opsx:explore` から始めるのが、最も身につけたい習慣です。

<a id="where-else-to-get-help"></a>
## その他のヘルプ

- **Discord:** 質問、アイデア、ヘルプは [discord.gg/YctCnvvshC](https://discord.gg/YctCnvvshC) へ。
- **GitHub Issues:** バグ報告や機能要望は [github.com/Fission-AI/OpenSpec/issues](https://github.com/Fission-AI/OpenSpec/issues) へ。
- **`openspec feedback "your message"`** を使うと、ターミナルから直接フィードバックを送信できます（GitHub の issue が開きます）。

このドキュメントに誤り、古い情報、わかりにくい箇所を見つけましたか？ それはバグです。issue や PR で報告してください。ドキュメントの改善は、最も価値のある貢献の一つです。
