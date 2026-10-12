<a id="examples--recipes"></a>
# 例とレシピ

実際の変更を、最初から最後まで紹介します。各レシピでは、入力するコマンドと返ってくる内容を示しているので、自分の状況に合うパターンを見つけてコピーできます。ここではデフォルトの**コア**コマンド（`propose`、`explore`、`apply`、`update`、`sync`、`archive`）を使っています。拡張セットが役立つ場合は、その旨を記しています。

始める前の注意: `/opsx:propose` のようなスラッシュコマンドは**AI アシスタントのチャット**に、`openspec` コマンドは**ターミナル**に入力します。これが初めてなら、先に [コマンドの仕組み](how-commands-work.md) を読んでください。以下のやり取りでは、`You:` と `AI:` がチャットで、`$` で始まる行がターミナルです。

> **まだ何を作るか決まっていませんか？** これらのレシピの多くは、最初に `/opsx:explore` で考えを整理してから始めると、より的確になります。[レシピ3](#recipe-3-exploring-before-you-commit) で実際の様子を紹介しており、[まず探索する](explore.md) ガイドで詳しく説明しています。

<a id="recipe-1-a-small-feature-the-fast-path"></a>
## レシピ1: 小さな機能、最短ルート

**使う場面:** 何がしたいかわかっていて、完結した小さな作業の場合。最もよくあるレシピです。

コマンドは3つだけです。提案、実装、アーカイブ。

```text
You: /opsx:propose add-logout-button

AI:  openspec/changes/add-logout-button/ を作成しました
     ✓ proposal.md — 理由と内容
     ✓ specs/      — 新しい要件とそのシナリオ
     ✓ design.md   — 技術的アプローチ
     ✓ tasks.md    — 実装チェックリスト
     実装の準備ができました。/opsx:apply を実行してください。
```

では、計画を読みましょう。提案とデルタ仕様を開きます。これこそ OpenSpec が想定している瞬間です。誤った前提を、400行のコードになってからではなく、まだ1段落のうちに見つけるのです。何かおかしければ、成果物を直接編集してから続行してください。

```text
You: /opsx:apply

AI:  タスクを進めています...
     ✓ 1.1 ヘッダーにログアウトボタンを追加
     ✓ 1.2 クリック時にセッションをクリア
     ✓ 1.3 確認ダイアログを追加
     すべてのタスクが完了しました！

You: /opsx:archive

AI:  add-logout-button をアーカイブしています...
     ✓ 仕様を openspec/specs/auth/spec.md にマージしました
     ✓ openspec/changes/archive/2026-06-22-add-logout-button/ に移動しました
     完了です。次の変更に進む準備ができました。
```

以上です。ログアウトの振る舞いは仕様の一部になり、変更はその完全な文脈とともに保管されました。

<a id="recipe-2-a-bug-fix"></a>
## レシピ2: バグ修正

**使う場面:** 何かが壊れていて、その修正を、謎のコミットではなく、意図的な振る舞いの変更として記録したい場合。

バグ修正は、機能追加とまったく同じように進みます。違いは提案の捉え方にあります。単に「バグを直す」のではなく、*正しい*振る舞いを記述してください。

```text
You: /opsx:propose fix-login-redirect-loop

AI:  openspec/changes/fix-login-redirect-loop/ を作成しました
     提案には、現在の壊れた振る舞いと、意図する振る舞いを記述しました。
     デルタ仕様はリダイレクトの要件を MODIFIED し、正確でテスト可能な
     ものにしています。
     実装の準備ができました。
```

修正は、新しいシナリオを伴う `MODIFIED` の要件として反映されるため、次の担当者（あるいは次の AI セッション）は、あなたが直したという事実だけでなく、「正しい」とは何を意味するのかを知ることができます。あとは通常どおり、`/opsx:apply` と `/opsx:archive` を実行します。

ヒント: 修正の場合、良いシナリオとは、散文で書かれたリグレッションテストです。「ログアウト状態のユーザーが（GIVEN）、有効な認証情報を送信したとき（WHEN）、ダッシュボードに遷移し、再びリダイレクトされない（THEN）。」これを書けば、実装に明確な目標ができます。

<a id="recipe-3-exploring-before-you-commit"></a>
## レシピ3: 決める前に探索する

**使う場面:** 問題はあるが、まだ計画がないとき。何を作るべきか、どのアプローチが正しいかがわからない場合。

`/opsx:explore` から始めます。構造を持たない思考パートナーです。コードは決して書かず、決めた内容を記録するよう頼んだとき、または提案に対して承諾したとき以外は、他に何も書きません。コードベースを読み、あなたの判断を助けます。

```text
You: /opsx:explore

AI:  何を探索しますか？

You: ページの読み込みが遅く感じるのですが、時間がどこにかかっているのかわかりません。

AI:  調べてみます... 原因として有力なものが3つ見つかりました。
     1. 商品一覧にある、最適化されていない大きな画像
     2. レンダリングをブロックする、同期的なデータ取得
     3. コンテキストの変更によって引き起こされる再レンダリング
     どれから取り組みましょうか？

You: データ取得のものです。

You: /opsx:propose optimize-product-list-fetching

AI:  変更を作成しました。先ほどの調査結果を踏まえた提案が入っています。
     実装の準備ができました。
```

探索は、変更を費やす*前に*考えを明確にします。洞察がまとまったら提案します。AI は文脈を引き継ぎます。

<a id="recipe-4-juggling-two-changes-at-once"></a>
## レシピ4: 2つの変更を同時に扱う

**使う場面:** 機能の作業中に、緊急の修正が割り込んできたとき。

変更は独立したフォルダなので、並行作業が衝突することはありません。修正を開始してリリースし、そのあと、中断したところから機能の作業に戻ります。

```text
You: add-dark-mode の途中ですが、ログインのバグを今すぐ直す必要があります。

You: /opsx:propose fix-login-redirect

AI:  openspec/changes/fix-login-redirect/ を作成しました。実装の準備ができました。

You: /opsx:apply

AI:  修正を実装しました。すべてのタスクが完了しました。

You: /opsx:archive

AI:  ✓ fix-login-redirect をアーカイブしました。

You: ダークモードに戻ります。

You: /opsx:apply add-dark-mode

AI:  add-dark-mode を再開します。タスク 2.3: Header の更新 から続けます...
```

`/opsx:apply add-dark-mode` のように変更名を指定することで、複数の変更が有効なときに特定の変更を AI に示せます。タスクの完了状況は `tasks.md` に記録されているため、AI はあなたが止めた場所を正確に把握しています。

複数の変更が同時に完了した場合は、拡張コマンドの `/opsx:bulk-archive` がまとめて保管し、実際に実装されている内容を確認して仕様の競合を解決します。[ワークフロー](workflows.md#parallel-changes) を参照してください。

<a id="recipe-5-a-refactor-with-no-behavior-change"></a>
## レシピ5: 振る舞いを変えないリファクタリング

**使う場面:** コードの構造を整理するが、外部から見える振る舞いは同一のままにしたい場合。

純粋なリファクタリングには、仕様に*追加するものが何もない*ため、これは興味深いケースです。振る舞いの契約は変わらず、変わるのは実装だけです。そのため作業は設計とタスクに集約され、仕様のデルタは空か存在しません。

```text
You: /opsx:propose refactor-payment-module

AI:  変更を作成しました。提案には目標（決済モジュールを分割する。
     振る舞いの変更なし）を記述し、設計には新しい構造を記述しました。
     振る舞いは同一なので、仕様の変更はありません。
     実装の準備ができました。
```

空のデルタを明示するには、変更の `.openspec.yaml` に `skip_specs: true` を設定します。

```yaml
schema: spec-driven
skip_specs: true
```

このマーカーがないと、`openspec validate` はデルタがゼロの変更を拒否します（これにより、仕様のフェーズの書き忘れも検出されます）。マーカーがあれば検証は通り、`openspec status` には、specs のステージが保留中ではなく、明示的にスキップされたものとして表示されます。リファクタリングが結果的に振る舞いを変えることになった場合は、`.openspec.yaml` から `skip_specs` を削除してデルタ仕様を書いてください。マーカーと仕様ファイルが両方あると、validate は矛盾として扱うため、古いマーカーが気づかれないまま残ることはありません。

マーカー付きの変更のアーカイブには、追加のフラグは必要ありません（マージするデルタがないためです）。これとは別に、`--skip-specs` フラグを使うと、ターミナルコマンドに仕様のステップを明示的にスキップするよう指示できます。

```bash
$ openspec archive refactor-payment-module --skip-specs
```

同じフラグは、ツール、CI、ドキュメントのみの変更にも便利です。原則: 仕様は振る舞いを記述するものなので、振る舞いが変わらなければ、仕様も変わるべきではありません。[概念](concepts.md#what-a-spec-is-and-is-not) を参照してください。

<a id="recipe-6-step-by-step-control-expanded-commands"></a>
## レシピ6: ステップごとの制御（拡張コマンド）

**使う場面:** 複雑またはリスクの高い変更で、次に進む前に各成果物をレビューしたい場合。

コアの `/opsx:propose` はすべてを一度に下書きします。1ステップずつ進めたい場合は、拡張コマンドを有効にします。

```bash
$ openspec config profile      # select the expanded workflows
$ openspec update              # apply them to this project
```

これで、段階的に骨組みを作り、実装できます。

```text
You: /opsx:new add-2fa

AI:  openspec/changes/add-2fa/ を作成しました。作成可能: proposal。

You: /opsx:continue

AI:  proposal.md を作成しました。作成可能: specs, design。

You: /opsx:continue

AI:  specs/auth/spec.md を作成しました。作成可能: design。
```

各成果物ができるたびにレビューし、自由に編集して、満足したら続行します。残りをまとめて下書きしたいときは、`/opsx:ff` が残りの計画用成果物を一気に進めます。アーカイブする前に、`/opsx:verify` で、実装が実際に仕様と一致していることを確認します。[ワークフロー](workflows.md#opsxff-vs-opsxcontinue) を参照してください。

<a id="recipe-7-learning-the-whole-loop-hands-on"></a>
## レシピ7: ループ全体を実際に体験して学ぶ

**使う場面:** OpenSpec をインストールして、おもちゃの例ではなく、自分のコードでワークフローを*体感*したい場合。

拡張コマンドを有効にして（レシピ6を参照）、次を実行します。

```text
You: /opsx:onboard

AI:  OpenSpec へようこそ！あなたの実際のコードベースを使って、完全な変更を
     一通りご案内します。一緒にできる、小さく安全な改善点がないか
     スキャンしてみます...
```

`/opsx:onboard` は、実際の（小さな）改善点を見つけ、そのための変更を作成し、実装し、アーカイブします。各ステップを説明しながら進めます。15〜30分かかり、残しても破棄してもよい本物の変更が手元に残ります。学ぶには最もやさしい方法です。[コマンド](commands.md#opsxonboard) を参照してください。

<a id="checking-your-work-from-the-terminal"></a>
## ターミナルから作業を確認する

ターミナルからは、いつでも状態を確認できます。

```bash
$ openspec list                      # active changes
$ openspec show add-dark-mode        # one change in detail
$ openspec validate add-dark-mode    # check structure
$ openspec view                      # interactive dashboard
```

これらは読み取りと確認のためのツールです。提案や実装は、引き続きチャットのスラッシュコマンドで行います。詳細は [CLI リファレンス](cli.md) を参照してください。

<a id="where-to-go-next"></a>
## 次に読むもの

- [まず探索する](explore.md): 迷っているときの推奨スタート方法
- [ワークフロー](workflows.md): 上記のパターンと、それぞれをいつ使うかの判断ガイド
- [コマンド](commands.md): すべてのスラッシュコマンドの詳細
- [はじめに](getting-started.md): 最初の変更の標準的なウォークスルー
- [概念](concepts.md): 各要素がこのように組み合わさっている理由
