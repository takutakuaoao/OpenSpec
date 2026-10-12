<a id="stores-plan-in-its-own-repo"></a>
# ストア: 計画を専用のリポジトリで管理する

> **ベータ版。** ストア、参照、作業コンテキスト、ワークセットは
> 新しいものです。コマンド名、フラグ、ファイル形式、JSON 出力は、リリース間で
> 形が変わる可能性があります。以下のウォークスルーはすべて現在のビルドで
> 実行されたものですが、アップグレード後にこのガイドを読み直してください。

<a id="the-problem-this-solves"></a>
## これが解決する問題

OpenSpec は通常、1つのコードリポジトリの中にあります。コードの隣に `openspec/` フォルダがあり、
そのリポジトリの仕様と変更を保持します。

計画が1つのリポジトリより大きくなると、それは合わなくなります。

- 作業が複数のリポジトリにまたがる — 1つの機能が、API サーバー、Web アプリ、共有ライブラリに影響する。
  計画は、誰の `openspec/` フォルダにあるべきか？
- チームがコードが存在する前に計画する、あるいは*このリポジトリ*では決してコードにならないものを計画する。
- 要件があるチームによって所有され、他のチームに利用される。Wiki 版は乖離していき、
  しかもコーディングエージェントはそれを読めない。

**ストア**がその答えです。計画だけを役割とする、独立したリポジトリです。
すでにご存じの `openspec/` の構成（仕様と変更）に、小さな識別ファイルが加わったものです。
マシン上に名前を付けて一度登録すれば、通常の OpenSpec コマンドはすべて、どこからでもその中で動作します。

<a id="the-shape"></a>
## 構成

```
            team-plans  (ストア: 専用のリポジトリでの計画)
            ├── .openspec-store/store.yaml     identity: "I am team-plans"
            └── openspec/
                ├── specs/      真実であるもの
                └── changes/    進行中のもの
                      ▲
                      │ 各マシンで名前を付けて登録される。
                      │ 通常のリポジトリと同様に、プッシュ/クローンで共有される
        ┌─────────────┼─────────────┐
        │             │             │
    web-app       api-server     mobile-app
   (コードリポジトリ) (コードリポジトリ) (コードリポジトリ)
```

2つのルールで、これをシンプルに保ちます。

1. **ストアは単なる git リポジトリです。** コミット、プッシュ、プル、レビューは自分で行います。
   OpenSpec が自らクローン、同期、プッシュを行うことは決してありません。
2. **仕組みではなく、宣言。** リポジトリは、ストアとどう関係するかを*宣言*できます（後述）。
   宣言は、OpenSpec があなたに伝えられることを変えますが、
   コマンドがどこで動作するかを変えることは決してありません。

<a id="five-minutes-to-your-first-store"></a>
## 最初のストアまで5分

2つのコマンドで、何もない状態から、ストアを対象とした動作する変更まで進めます。

```bash
openspec store setup team-plans --path ~/openspec/team-plans
```

```
Store ready: team-plans
Location: /Users/you/openspec/team-plans
OpenSpec root: ready
Registry: registered

Next: run normal OpenSpec commands against this store, for example:
  openspec new change <change-id> --store team-plans
Share this store by committing and pushing it like any Git repo.
```

```bash
openspec new change add-login --store team-plans
```

```
Using OpenSpec root: team-plans (/Users/you/openspec/team-plans)
Created change 'add-login' at /Users/you/openspec/team-plans/openspec/changes/add-login/
Schema: spec-driven
Next: openspec status --change add-login --store team-plans
```

これがモデルのすべてです。ここから先のライフサイクルは、ご存じのとおり — `status`、`instructions`、`validate`、`archive` —
各コマンドに `--store team-plans` を付けるだけで、表示されるヒントにはすべて、そのフラグが含まれています。
`Using OpenSpec root:` の行は、コマンドがどこで動作しているかを常に教えてくれます。

<a id="story-one-team-one-planning-repo"></a>
## ストーリー: 1つのチーム、1つの計画リポジトリ

チームは、仕様と変更を、コードリポジトリに散らばらせる代わりに、`team-plans` に保管しています。

**初日（セットアップする人）:**

```bash
openspec store setup team-plans --path ~/openspec/team-plans \
  --remote git@github.com:acme/team-plans.git
git -C ~/openspec/team-plans push -u origin main
```

`--remote` を渡すと、クローン URL が、ストア自身の識別ファイル
（`.openspec-store/store.yaml`）の最初のコミットに記録されます。以後のすべての
クローンは、自分がどこから来たかを最初から把握しているため、健全性チェックやエラーメッセージは、
まだ持っていないチームメイトのために、完全で貼り付け可能な修正方法を表示できます。

**すべてのチームメイト（マシンごとに一度）:**

```bash
git clone git@github.com:acme/team-plans.git ~/openspec/team-plans
openspec store register ~/openspec/team-plans
```

その後は、全員が同じ計画リポジトリで、名前を指定して作業します。

```bash
openspec status --store team-plans --change add-login
openspec show add-login --store team-plans
```

**作業の共有は、意図的に git で行います。** 作成した変更は、コードと同じように、
コミットしてプッシュするまで、自分のチェックアウトにしか存在しません。ストアは通常の
リポジトリなので、計画にもブランチ、プルリクエスト、レビューが自然に使えます。

**チームのコードリポジトリを接続する。** 計画が完全に外部化されているコードリポジトリには、
`openspec/config.yaml` に次の1行だけが必要です。

```yaml
# web-app/openspec/config.yaml
store: team-plans
```

これで、`web-app` 内で実行される OpenSpec のコマンドはすべて、フラグなしで
`team-plans` に対して動作します。

```bash
cd ~/src/web-app
openspec status --change add-login
```

```
Using OpenSpec root: team-plans (/Users/you/openspec/team-plans)
...
```

ポインターはフォールバックであり、上書きではありません。明示的な `--store` が常に優先され、
リポジトリが独自の実際の計画フォルダを持つようになった場合は、そちらが優先されます
（古いポインターを削除するよう、警告が出ます）。

**マシン上のすべてのリポジトリに対する、1つのデフォルト。** 同じストアに計画を入れる
多くのコードリポジトリで作業する場合は、各リポジトリに `store:` の行を追加する代わりに、
グローバルに一度だけ設定します。

```bash
openspec config set defaultStore team-plans
```

これで、計画のルートの外で実行され、`--store` もプロジェクトのポインターもないコマンドは、
`team-plans` に解決されます。これは優先順位のリストの最下位にあるため、`--store`、ローカルルート、
プロジェクトの `store:` ポインターは、いずれも引き続き優先されます。ルートバナーと JSON の `root` ブロックは、
ストア ID とともに `source: "global_default"` を報告するため、マシン全体のデフォルトと、
リポジトリ自身のポインターを、常に区別できます。解除するには
`openspec config unset defaultStore` を実行します。その ID が登録されていない場合、
コマンドはエラーになり、登録するか、古いデフォルトを解除するよう伝えます。

<a id="example-one-feature-two-component-repos"></a>
## 例: 1つの機能、2つのコンポーネントのリポジトリ

`add-checkout-promo` が、`checkout-api` と `checkout-web` の両方を変更するとします。
チームは、共有の製品契約を1つ持ちたい一方で、各コードリポジトリは、独自の
実装タスク、ブランチ、レビューを必要とします。

2つのレイヤーを使います。

1. 共有する振る舞いは `team-plans` に置く。
2. 実装計画は各コンポーネントのリポジトリに置き、ストアを読み取り専用の上流コンテキストとして参照する。

まず、ストアで共有の契約を計画します。

```bash
openspec new change add-checkout-promo --store team-plans
openspec status --change add-checkout-promo --store team-plans
```

提案と仕様は、コンポーネント間の境界での振る舞いを記述するべきです。たとえば、
サービスが返すプロモーションのフィールドや、フロントエンドが対象外のチェックアウトを
どう扱うかです。この変更は、他のブランチやプルリクエストと同様に、ストアのリポジトリでレビューします。

<a id="what-context-does-planning-see"></a>
### 計画は、どのコンテキストを見るのか？

ストアを選択すると OpenSpec のルートが変わりますが、そのストアを使用するすべての
コードリポジトリを検出したり読み取ったりするわけではありません。ストアの指示が見るのは、
ストア内の成果物と設定されたコンテキストです。コンポーネントのコードを見るのは、
それらのフォルダがエージェントやエディタからも利用可能で、エージェントがそれを読んだ場合に限られます。

ワークセットは、計画用のストアと2つのコードリポジトリを一緒に開く便利な方法です。

```bash
openspec workset create checkout-promo \
  --member ~/openspec/team-plans \
  --member ~/src/checkout-api \
  --member ~/src/checkout-web \
  --tool code
openspec workset open checkout-promo
```

これにより、フォルダが1つの IDE ワークスペースに表示されます。ソースのコンテキストを
ストアにコピーしたり、影響を受けるリポジトリを選択したり、エージェントにそれらを編集する
権限を与えたりはしません。コンポーネントをまたぐ永続的な事実は、共有の仕様に記述してください。
計画者がたまたま調べたソースを覚えていることに頼ってはいけません。

<a id="how-does-implementation-start-in-each-repo"></a>
### 各リポジトリでの実装は、どう始まるのか？

明示的な `--store` や、より近い `openspec/` ルートが適用されない場合、`store: team-plans` の
ポインターは、コマンドをそのストアにルーティングします。`apply` が呼び出されたディレクトリによって、
ストアの1つのタスクリストを分割することはありません。OpenSpec は現在、タスクをリポジトリに
ルーティングしません。

各コンポーネントが独立したスコープの apply/レビューのサイクルを必要とする場合は、
中央のストアを指す代わりに、ローカルの OpenSpec ルートを持たせ、ストアを参照させます。

```yaml
# checkout-api/openspec/config.yaml (and likewise in checkout-web)
schema: spec-driven
references:
  - team-plans
```

共有の契約が承認され、ストアのメイン仕様で利用できるようになったら、コンポーネントの
担当部分のために、小さなローカルの変更を作成します。

```bash
cd ~/src/checkout-api
openspec new change implement-checkout-promo-api

cd ~/src/checkout-web
openspec new change implement-checkout-promo-ui
```

各リポジトリの instructions にある参照インデックスは、ストアの仕様の要約と、正確な
`openspec show ... --store team-plans` の取得コマンドを提供します。各ローカルの提案は
その共有の契約を引用し、そのタスクはそのコンポーネント内の作業のみを記述します。
その後、各リポジトリで別々に `/opsx:apply` を実行します。ルートの解決により、成果物と
実装の編集は、そのリポジトリに限定されます。サービスとフロントエンドの変更は、
独立してテスト、レビュー、マージ、アーカイブできるようになります。

共有のストアの変更がまだアクティブな間に実装を始める必要がある場合は、
`openspec show add-checkout-promo --store team-plans` で明示的に取得してください。
参照インデックスが列挙するのは、正規のストアの仕様であり、アクティブなストアの変更ではありません。
ストアのブランチとコンポーネントのブランチを、プルリクエストの説明の中で関連付けておくと、
レビュアーは、各実装が契約のどのバージョンに従っているかを確認できます。

<a id="story-requirements-that-cross-team-lines"></a>
## ストーリー: チームの境界をまたぐ要件

プラットフォームチームが要件を所有しています。プロダクトチームは、自分たちのリポジトリで、
自分たちの設計に従ってそれらに対して構築します。参照は、誰の作業も動かすことなく、
その関係を記述します。

```
   platform-reqs (ストア)                 api-server (コードリポジトリ)
   プラットフォームチームが所有            プロダクトチームが所有
   ┌──────────────────────────┐          ┌──────────────────────────┐
   │ openspec/specs/          │ ◀────────│ openspec/config.yaml     │
   │   payments/spec.md       │ 読み取り  │   references:            │
   │   auth/spec.md           │          │     - platform-reqs      │
   │                          │          │ openspec/specs/          │
   │ openspec/changes/        │          │   (自分たちの設計)         │
   │   プラットフォームの作業    │          │ openspec/changes/        │
   │                          │          │   (自分たちの作業)         │
   │                          │          └──────────────────────────┘
   └──────────────────────────┘
```

**プロダクトチームは、利用するものを宣言します。** リポジトリの
`openspec/config.yaml` に次のように書きます。

```yaml
references:
  - platform-reqs
```

参照は読み取り専用のコンテキストです。リポジトリは自身の `openspec/` ルートを保持し、
作業はそこにとどまります。変わるのは、そのリポジトリでの `openspec instructions` に、
参照先のストアの仕様のインデックスが含まれるようになる点です。それぞれに1行の要約と、
正確な取得コマンド（`openspec show <spec-id> --type spec
--store platform-reqs`）が付きます。`api-server` で作業するエージェントは、
上流の決済に関する要件を見つけて引用し、リポジトリ自身のルートに低レベルの設計を書くことができます。
誰かがコンテキストをあちこちに貼り付ける必要はありません。

参照にはクローン元を含めることができるため、まだストアを持っていないチームメイトは、
行き止まりではなく、完全な修正方法を得られます。

```yaml
references:
  - { id: platform-reqs, remote: "git@github.com:acme/platform-reqs.git" }
```

**計画とコードを一緒に開きたいときは、ワークセットを作ります。** これは個人的で
明示的なものです。各人が、自分のマシンで実際に作業するフォルダを選びます。それらの
ローカルのチェックアウトのパスについては、何も共有の計画リポジトリにコミットされません。

```bash
openspec workset create platform \
  --member ~/openspec/platform-reqs \
  --member ~/src/api-server \
  --member ~/src/web-app
```

<a id="two-questions-you-can-always-ask"></a>
## いつでも尋ねられる2つの問い

**「自分のセットアップは健全か？」** — `openspec doctor` は、現在のルートと、
それが参照するストアを、読み取り専用でチェックし、所見ごとに貼り付け可能な修正方法を示します。

```
Doctor

Root
  Location: /Users/you/src/api-server
  OpenSpec root: ok

References
  - platform-reqs: ok (/Users/you/openspec/platform-reqs)
  - design-system: Referenced store 'design-system' is not registered on this machine.
    Fix: git clone -- git@github.com:acme/design-system.git '/Users/you/openspec/design-system' && openspec store register '/Users/you/openspec/design-system' --id design-system

```

**「何を扱っているのか？」** — `openspec context` は、OpenSpec の宣言から作業セットを
組み立てます。ルートと、それが参照するストアです。

```
Working context for api-server (/Users/you/src/api-server)

OpenSpec root
  api-server  /Users/you/src/api-server

Referenced stores
  platform-reqs  /Users/you/openspec/platform-reqs
    Fetch: openspec show <spec-id> --type spec --store platform-reqs
```

どちらも、エージェント向けに `--json` に対応しています。`openspec context --code-workspace
<path>` は、さらにセット全体を含む VS Code のワークスペースファイルを書き込みます。
これがこのコマンドが行う唯一の書き込みです。

<a id="worksets-reopen-the-folders-you-work-on-together"></a>
## ワークセット: 一緒に作業するフォルダを開き直す

ここまでとは別の話です。多くの人は、セッションのたびに同じいくつかのフォルダ、つまり
計画リポジトリと2〜3個のコードリポジトリを一緒に開きます。**ワークセット**は、まさにそれを表す、
個人用の名前付きビューであり、お好みのツールで1つのコマンドで開き直せます。

```
  workset "platform"                 openspec workset open platform
  ├── team-plans   ~/openspec/team-plans         │
  ├── api-server   ~/src/api-server              ▼
  └── web-app      ~/src/web-app       3つすべてがツールで開く
```

```bash
openspec workset create platform \
  --member ~/openspec/team-plans --member ~/src/api-server \
  --tool code
openspec workset list
```

```
platform  (opens in VS Code)
  team-plans  /Users/you/openspec/team-plans
  api-server  /Users/you/src/api-server
```

その後、`openspec workset open platform` が保存されたツールを起動します。エディタ
（VS Code、Cursor）は、すべてのメンバーを含む1つのウィンドウを開いて戻ります。最初の
メンバーがプライマリです。ツールはいつでも `--tool <id>` で上書きできます。

ワークセットは、意図的に共有状態では*ありません*。あなたのマシン上にあり、コミットされることはなく、
作業内容については何も主張しません。一緒に開いておきたいものを記録するだけです。
削除しても、メンバーのフォルダには一切触れません。新しいツールへの対応は、コードではなく設定です。
ワークスペースファイルやフォルダごとのアタッチフラグで起動できるものは、グローバル設定
（`openspec config edit`）の `openers` キーに追加できます。

<a id="how-commands-decide-where-to-act"></a>
## コマンドは、どこで動作するかをどう決めるか

通常のコマンドはすべて、次の順序で同じようにルートを解決します。

```
1. --store <id>          明示的に指定した場合         → そのストア
2. nearest openspec/     ここに本物の計画ルートがある   → このリポジトリ
   (cwd から上に向かって探す)
3. store: pointer        config.yaml がストアを宣言    → そのストア
4. defaultStore          グローバル設定がマシンの       → そのストア
                         デフォルトを設定している
5. 上記のいずれでもない    このマシンに登録された         → 選択のヒント付きの
                         ストアがある？                  エラー
                         登録されたストアがない？        → カレントディレクトリ
                                                          （従来の動作）
```

`Using OpenSpec root:` の行（と、`--json` 出力の `root` ブロック）が、
どのケースにいるかを教えてくれます。

<a id="known-limitations"></a>
## 既知の制限

- **ベータ版の形式。** このページのすべては、リリース間で変わる可能性があります —
  名前、フラグ、ファイル形式、JSON のキー。
- **マシンごとに、ストア ID ごとのチェックアウトは1つ。** 同じ ID で2つ目のチェックアウトを
  登録すると失敗し、先に `store unregister` するよう案内されます。
- **同期は、設計上、決して行わない。** OpenSpec は、クローン、プル、プッシュを決して行いません。
  古いチェックアウトは、*あなた*がプルするまで、古い仕様を表示します。参照は、
  ディスク上にあるものからその場でインデックスされます。
- **空の計画フォルダは存在しなくてよい。** 新しいストアには、Git 上に
  `openspec/changes/`、`openspec/specs/`、`openspec/changes/archive/` がまだない場合があります。
  ベータ期間中はこれが許容され、通常のコマンドがそれらのためのファイルを作成すると、
  フォルダが現れます。
- **ポインターのリポジトリは、ポインターのまま。** `openspec/config.yaml` が `store: <id>` を
  宣言している設定のみのリポジトリは、登録すべきストアのチェックアウトではなく、
  外部化された計画として扱われます。そのリポジトリを意図的にローカルのストアルートに変換したい場合は、
  先に `store:` の行を削除してください。
- **一部のコマンドは、従来のまま。** `templates` と、
  非推奨の名詞形式（`openspec change show` など）は、カレントディレクトリでのみ動作し、
  `--store` はありません。`schemas` は、正規のルート選択の優先順位に従い、`--store <id>` を受け付ける一方、
  成功時の JSON の配列形式は変わりません。
- **マシンごとの状態は、マシンごと。** ストアのレジストリとワークセットは、
  ローカルの設定です。あなたのマシンの構成に関することは、
  共有の計画にコミットされることはありません。
- **ワークセットの起動スタイルは2つ。** ワークスペースファイルやフォルダごとのアタッチフラグで
  起動できないツールは、オープナーとして追加できません。
- **エージェント向け JSON には、既知の表記の分裂がある**（ストア系のキーは
  snake_case、ワークフロー系は camelCase）。[エージェントコントラクト](../agent-contract.md)に
  記載されています。統一は、バージョン付きのリリースまで先送りされています。

<a id="where-things-live"></a>
## 各種ファイルの場所

| 内容 | 場所 | 共有されるか？ |
|---|---|---|
| ストアの計画 | `<store>/openspec/`（仕様、変更） | はい — コミットしてプッシュする |
| ストアの識別情報 | `<store>/.openspec-store/store.yaml` | はい — ストアとともにコミットされる |
| ストアのレジストリ | `<data dir>/openspec/stores/registry.yaml` | いいえ — このマシンのみ |
| ワークセット | `<data dir>/openspec/worksets/` | いいえ — このマシンのみ |

`<data dir>` は、macOS と Linux では `~/.local/share/openspec`（`$XDG_DATA_HOME/openspec` が
設定されている場合はそちら）、Windows では `%LOCALAPPDATA%\openspec` です。

<a id="reference"></a>
## リファレンス

このページのすべてのコマンドの正確なフラグと JSON 形式は、
[CLI リファレンス](../cli.md)（ストア、Doctor、作業コンテキスト、個人用ワークセット）と
[エージェントコントラクト](../agent-contract.md) にあります。
