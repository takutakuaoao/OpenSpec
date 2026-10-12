<a id="multi-language-guide"></a>
# 多言語ガイド

OpenSpec が英語以外の言語で成果物を生成するように設定します。

<a id="quick-setup"></a>
## クイックセットアップ

新規プロジェクトでは、初期化時に言語を設定します。

```bash
openspec init --language "Portuguese (pt-BR)"
```

これにより、言語の指示が `openspec/config.yaml` に書き込まれます。プロジェクトに
すでに設定ファイルがある場合は、既存のプロジェクトガイダンスを保持するため、
`context` フィールドを直接編集してください。

同じ動作を手動で設定することもできます。

`openspec/config.yaml` に言語の指示を追加します。

```yaml
schema: spec-driven

context: |
  Language: Portuguese (pt-BR)
  All artifacts must be written in Brazilian Portuguese.
  Keep OpenSpec structural headings and SHALL/MUST keywords in English.

  # Your other project context below...
  Tech stack: TypeScript, React, Node.js
```

これだけです。以後、生成されるすべての成果物がポルトガル語になります。

OpenSpec のドキュメント構造と、規範的なキーワードである `SHALL`/`MUST` は、
検証がそれらに依存しているため、英語のままにします。その周囲の要件やシナリオの
文章は、選択した言語で記述できます。

<a id="language-examples"></a>
## 言語の例

<a id="portuguese-brazil"></a>
### ポルトガル語（ブラジル）

```yaml
context: |
  Language: Portuguese (pt-BR)
  All artifacts must be written in Brazilian Portuguese.
```

<a id="spanish"></a>
### スペイン語

```yaml
context: |
  Idioma: Español
  Todos los artefactos deben escribirse en español.
```

<a id="chinese-simplified"></a>
### 中国語（簡体字）

```yaml
context: |
  语言：中文（简体）
  所有产出物必须用简体中文撰写。
```

<a id="japanese"></a>
### 日本語

```yaml
context: |
  言語：日本語
  すべての成果物は日本語で作成してください。
```

<a id="french"></a>
### フランス語

```yaml
context: |
  Langue : Français
  Tous les artefacts doivent être rédigés en français.
```

<a id="german"></a>
### ドイツ語

```yaml
context: |
  Sprache: Deutsch
  Alle Artefakte müssen auf Deutsch verfasst werden.
```

<a id="tips"></a>
## ヒント

<a id="handle-technical-terms"></a>
### 専門用語の扱い

専門用語をどう扱うかを決めておきます。

```yaml
context: |
  Language: Japanese
  Write in Japanese, but:
  - Keep technical terms like "API", "REST", "GraphQL" in English
  - Code examples and file paths remain in English
```

<a id="combine-with-other-context"></a>
### 他のコンテキストと組み合わせる

言語設定は、他のプロジェクトコンテキストと併用できます。

```yaml
schema: spec-driven

context: |
  Language: Portuguese (pt-BR)
  All artifacts must be written in Brazilian Portuguese.

  Tech stack: TypeScript, React 18, Node.js 20
  Database: PostgreSQL with Prisma ORM
```

<a id="verification"></a>
## 動作確認

言語設定が機能していることを確認するには、次を実行します。

```bash
# Check the instructions - should show your language context
openspec instructions proposal --change my-change

# Output will include your language context
```

<a id="related-documentation"></a>
## 関連ドキュメント

- [カスタマイズガイド](./customization.md) - プロジェクト設定のオプション
- [ワークフローガイド](./workflows.md) - ワークフローの完全なドキュメント
