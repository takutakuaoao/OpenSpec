<a id="openspec-agent-contract"></a>
# OpenSpec エージェントコントラクト

`openspec` CLI の機械可読なインターフェースです。`src/` に照らして検証されています（最終監査、2026-06-11）。以下のすべての形式は、出力を行うコードから文書化されています。

<a id="1-general-conventions"></a>
## 1. 一般的な規約

- **1回の呼び出しにつき、JSON ドキュメントは1つ。** `--json` モードでは、stdout にはちょうど1つの JSON ドキュメント（インデント2スペースで整形）が出力されます。人間向けの文章、スピナー、ストアのバナーは stderr に出力されます。
- **ストアのバナー。** 人間向けモードでは、ストアで選択されたルートは `Using OpenSpec root: <id> (<path>)` を stderr に出力します。JSON モードでは出力されません。
- **キーの大文字小文字の表記は、インターフェースによって異なります**（「既知の不整合」を参照）。ストア/doctor/context のペイロードは `snake_case`、ワークフローのペイロード（`status`、`instructions`、`new change`、`validate`、`list`）は `camelCase` を使用します。ただし、埋め込まれた `root` オブジェクトは、常に `store_id` を使用します。
- **省略可能なキーは、ほとんどのペイロードで null ではなく省略されます**（例: `root.store_id`、`member.path`）。明示的な `null` を使用する例外は、形式ごとに明記しています（store doctor の `git.*`、失敗のペイロード）。

<a id="2-the-diagnostic-envelope"></a>
## 2. 診断エンベロープ

機械可読な診断はすべて、1つのエンベロープ形式（`StoreDiagnostic`）を共有します。

```json
{
  "severity": "error" | "warning" | "info",
  "code": "snake_case_string",
  "message": "human sentence",
  "target": "dotted.surface (optional)",
  "fix": "one actionable sentence/command (optional)"
}
```

診断は2つの位置に現れます。健全性の所見を示す**ステータス配列**（トップレベル、またはエントリごとの `status: StoreDiagnostic[]`）と、コマンドの失敗時に1要素の `status` 配列に変換される**スローされたエラー**です。

<a id="3-root-selection-and-rootoutput"></a>
## 3. ルートの選択と `RootOutput`

ルートを解決するすべてのコマンド（`list`、`show`、`validate`、`status`、`instructions`、`instructions apply`、`instructions archive`、`new change`、`archive`、`doctor`、`context`、`schemas`）は、同じ優先順位で1つの OpenSpec ルートを解決します。

1. `--store <id>` → 登録済みストアのルート（`source: "store"`）。
2. そうでなければ、`openspec/` を持つ最も近い祖先: 計画の形式 → `source: "nearest"`（`store:` ポインターは stderr に警告を出して無視される）。有効な `store:` ポインターを持つ設定のみのディレクトリ → そのストア、`source: "declared"`。
3. 最も近いルートがなく、グローバルな `defaultStore` が設定されている（`openspec config set defaultStore <id>`）→ そのストア、`source: "global_default"`。古い ID は、元のストアのエラーと、`openspec config unset defaultStore` を示す `fix` とともに失敗する。
4. 最も近いルートがなく、デフォルトもなく、登録済みのストアが存在する → エラー `no_root_with_registered_stores`。
5. ルートなし、デフォルトなし、ストアなし: コマンドは cwd を `source: "implicit"` として扱うことがある。`doctor`、`context`、`list`、一括 `validate` は、代わりに `no_openspec_root` で失敗する。`list` は、`openspec/project.md` を持つレガシーなプロジェクトのために、暗黙のフォールバックを維持する。

成功した JSON ペイロードは、通常ルートを埋め込みます。成功した `schemas --json` は、
意図的に §4.13 に記載された互換性のための素の配列のままです。

```json
"root": { "path": "/abs/path", "source": "store" | "declared" | "global_default" | "nearest" | "implicit", "store_id": "id (only when store-selected)" }
```

**ルート失敗の契約**: JSON モードで解決に失敗すると、stdout に `{ ...commandNullShape, "status": [diagnostic] }` を出力し、終了コード1で終了します。

<a id="4-command-json-shapes"></a>
## 4. コマンドの JSON 形式

<a id="41-list---json"></a>
### 4.1 `list --json`
`{ "changes": [ { "name", "completedTasks", "totalTasks", "lastModified", "status": "no-tasks"|"complete"|"in-progress", "nested"?: ["<area>/<name>", ...] } ], "warnings"?: [ { "code", "name", "nested", "message" } ], "root": RootOutput }` — ここでは、変更ごとの `status` が文字列の列挙型であることに注意してください。`--specs`: `{ "specs": [ { "id", "requirementCount" } ], "root" }`。

`warnings`（空の場合は省略）は、`changes/` の下にある、変更ではないディレクトリを報告します。現時点で唯一のコードは `nested_change_directory` で、変更のディレクトリを包む名前空間フォルダです。変更は常に `changes/` の直下のディレクトリでなければならないため、OpenSpec はこれを扱えません。同じエントリが、一覧表示された変更に `nested` を伴い、その `status` は意味を持ちません。このようなエントリを変更として扱わないでください。メッセージを報告し、ディレクトリには手を加えないでください。

<a id="42-show-item---json"></a>
### 4.2 `show <item> --json`
変更: `{ "id", "title", "deltaCount", "deltas": [...], "root" }`。仕様: `{ "id", "title", "overview", "requirementCount", "requirements": [...], "metadata": { "version", "format", "sourcePath"? }, "root" }`。仕様内、または変更のデルタの `requirement`/`requirements` 内の要件は、`{ "name", "text", "scenarios": [ { "name", "rawText" } ] }` です。要件の `name` は、`Requirement:` と末尾の `#` の連続を除いたヘッダーであり、アーカイブが MODIFIED/REMOVED/RENAMED のエントリと照合する正確な名前です。シナリオの `name` は、`Scenario:` と末尾の `#` の連続を除いたレベル4のヘッダーであり、MODIFIED のシナリオ消失チェックが比較する名前です。

<a id="43-validate---json"></a>
### 4.3 `validate --json`
`{ "items": [ { "id", "type": "change"|"spec", "valid", "issues": [ { "level", "path", "message", "line"?, "column"? } ], "durationMs" } ], "summary": { "totals": {items,passed,failed}, "byType": {...} }, "version": "1.0", "root" }`。いずれかの項目が失敗した場合は、終了コード1。

<a id="44-status---json"></a>
### 4.4 `status --json`
`{ "changeName", "schemaName", "planningHome"?: { "kind", "root", "changesDir", "defaultSchema" }, "changeRoot", "artifactPaths": { "<id>": {outputPath, resolvedOutputPath, existingOutputPaths} }, "nextSteps": ["..."], "actionContext": { "mode": "repo-local", "sourceOfTruth": "repo", "planningArtifacts", "linkedContext", "allowedEditRoots", "requiresAffectedAreaSelection", "constraints" }, "isPlanningComplete", "isComplete", "applyRequires", "artifacts": [ {id, outputPath, status: "done"|"skipped"|"ready"|"blocked", requires, missingDeps?} ], "root" }`。ストアで選択されたルートの場合、`allowedEditRoots` は、現在のパス上で最も近いプロジェクトが、`store:` ポインターがそのストアを指す設定のみのルートであれば `[<declaring project>, <store>]`、そうでなければ（グローバルな `defaultStore` の場合を含む）`[<store>]` となり、どのリポジトリを編集するかを尋ねるようエージェントに伝える制約が付きます。OpenSpec はストアのタスクをリポジトリにルーティングしないため、宣言しているプロジェクトは現在のものであり、変更が触れるすべてのリポジトリではありません。`isPlanningComplete` は、スキップされていないすべての計画用成果物が存在することを意味します。スキップされた成果物は、作成されなくても満たされたものとして扱われます。実装タスクが完了していることを意味するものではありません。`isComplete` は、同じ値を持つ互換性のためのエイリアスとして残されています。各成果物の `requires` は、その直接の依存先 ID です（すべてのステータスで存在するため、成果物が `done` であっても、推移的に必要な集合を計算できます）。`missingDeps` は `blocked` の場合にのみ現れます。`artifacts` 配列は依存関係の順に並んでおり、同時に準備ができる成果物の間の同順位は、スキーマの `artifacts:` の宣言順で解消されます（アルファベット順ではありません）。したがって、最初の `ready` のエントリが、次に書くべき成果物です。`missingDeps` も同じ順序を使います。`"skipped"` は、`generates` パスが `specs/` の下にあり、変更の `.openspec.yaml` が `skip_specs: true` を宣言している成果物を示します。依存関係は満たしますが、作成してはいけません。アクティブな変更がない場合: `{ "changes": [], "message", "root" }`、終了コード0。

`--all`（バッチ。`--change` とは相互に排他的で、組み合わせると `{ "changes": [], "root": null, "status": [d] }` の null 形式のエラーになります）: `{ "changes": [ <変更ごとのステータスオブジェクト。変更ごとの root なし>, ... ], "root" }`。変更名でソートされます。読み込みに失敗した変更は、その位置に `{ "changeName", "status": [d] }` を返します。走査は継続され、完全なエンベロープを保持し、テキストモードでも JSON モードでも終了コード1で終了します。無効な `--schema` は、変更が存在しない場合でも、null 形式で呼び出し全体を失敗させます。

<a id="45-instructions-artifact---json"></a>
### 4.5 `instructions <artifact> --json`
`{ "changeName", "artifactId", "schemaName", "changeDir", "planningHome"?, "outputPath", "resolvedOutputPath", "existingOutputPaths", "description", "instruction"?, "context"?, "rules"?, "references"?: ReferenceIndexEntry[], "skipped"?, "warning"?, "template", "dependencies": [{id,done,path,description,skipped?}], "unlocks", "root" }`。`unlocks` は、この成果物によって準備ができる成果物を、スキーマの宣言順（`status` が推奨するのと同じ順序）で列挙します。変更が `skip_specs: true` を宣言し、この成果物がスキップされる場合は、`"skipped": true`（`"warning"` を伴う）が現れます。そのファイルを作成してはいけません。`skipped: true` の依存関係エントリは、ファイルなしで満たされています。そのパスを読もうとしないでください。

`ReferenceIndexEntry`: `{ "store_id", "root"?, "specs"?: [{id,summary}], "fetch"?, "status": [] }` — 解決されたエントリには root/specs/fetch が含まれ、未解決のものには store_id と警告のステータスが含まれます。インデックスは 50KB が上限です（`reference_index_truncated`）。

<a id="46-instructions-apply---json"></a>
### 4.6 `instructions apply --json`
`{ "changeName", "changeDir", "schemaName", "contextFiles": { "<artifactId>": ["/abs", ...] }, "progress": {total,complete,remaining}, "tasks": [{id,description,done}], "state": "blocked"|"all_done"|"ready", "missingArtifacts"?, "missingPrerequisites"?, "warnings"?, "instruction", "references"?, "context"?, "operationGuidance"?, "root" }`。`missingArtifacts` は apply がブロックされる原因（スキーマの `apply.requires`）です。`missingPrerequisites` は、apply を実行できるようになる前にまだ作成すべきすべてのものを、作成順に並べたものです。これらの requires の推移的閉包なので、より長いリストになることがあります。`warnings` は、変更自体の、ブロックしない問題を列挙します。現時点では、デルタ仕様がなく、`skip_specs: true` もない状態で実装の準備ができている変更で、`openspec validate` が拒否する状態です。どちらの省略可能なルートフィールド（`context`、`operationGuidance`）も、呼び出しのたびに、選択したルートから読み取られます。`context` は、プロンプトレベルで必須の入力であり、関連するプロジェクトの事実、規約、制約を適用しなければなりません。`operationGuidance` は助言的な入力であり、その項目は、組み込みのワークフローと両立し、該当する場合にのみ従われます。どちらも、状態、タスク、進捗、コンテキストファイル、組み込みの指示とは別のものとして扱われます。

<a id="47-instructions-archive---json"></a>
### 4.7 `instructions archive --json`
`{ "changeName", "context"?, "operationGuidance"?, "root" }`。解決されたリポジトリ/ストアのルートで有効な `--change` が必要で、apply と同じ、必須のコンテキスト/助言的なガイダンスの意味論を使用します。これは読み取り専用の実行時入力のインターフェースです。静的なアーカイブワークフローを返さず、デルタ仕様の検査やマージ、メイン仕様の書き込み、変更の移動も行いません。

<a id="48-new-change-name---json"></a>
### 4.8 `new change <name> --json`
成功: `{ "change": { "id", "path", "metadataPath", "schema" }, "root" }`。失敗: `{ "change": null, "status": [d] }`、終了コード1。

<a id="49-archive-name---json"></a>
### 4.9 `archive <name> --json`
成功: `{ "archive": { "change", "archivedAs": "YYYY-MM-DD-name", "path", "specsUpdated", "totals"?, "warnings"? }, "root" }`。失敗: `{ "archive": null, "root"?, "status": [d] }`、終了コード1。`specsUpdated` は、少なくとも1つの仕様ファイルが書き込まれた、または廃止された場合にのみ true になります（変更が最後の要件を削除した機能は、その仕様が削除されます。これには変更の `.openspec.yaml` に `retire_capabilities: true` が必要です。すべての廃止は `warnings` に名前が挙げられ、仕様が呼び出し元のチェックアウト内にあった場合にのみ、貼り付け可能な Git の復旧コマンドが付きます）。すでに同期済みの変更は、すべてゼロの totals と、`warnings` に列挙されたスキップとともにアーカイブされます。JSON モードは厳密に非対話型です。すべてのプロンプトの箇所は `archive_*` コードになります。

- **`archive: null`**: コマンドが失敗しました。ファイルが変更されていないことは保証されません。
- **`archive_retirement_cleanup_failed`**: 変更はアーカイブされましたが、廃止のバックアップの検証またはクリーンアップが失敗しました。列挙されたバックアップのパスが変更または消失している可能性があります。メッセージは、フォールバックコピーのクリーンアップの失敗により残されたステージング済みのソースも報告することがあります。クリーンアップの前に、現在のアーカイブと、報告されたすべての復旧パスを確認してください。必要な内容は保存してください。アーカイブを再試行しても、これらのパスはクリーンアップされません。
- **`archive_error`**: このフォールバックの診断では、ファイルが変更されたかどうかは特定されません。再試行する前に、変更、アーカイブ、影響を受けた仕様を確認してください。

<a id="410-doctor---json"></a>
### 4.10 `doctor --json`
`{ "root": { "path", "source", "store_id"?, "healthy", "status": [] }, "store": { "id", "metadata": {present,valid,remote?}, "origin_url"?, "drift"?: {ahead,behind}, "status": [] } | null, "references": [...], "status": [] }`。`drift`（upstream のトラッキング参照を持つ、Git ベースのストアのチェックアウトにのみ存在）は、実際のリモートではなく、最後に fetch した upstream に対する ahead/behind の数です。どの重大度の健全性の所見も、終了コード0です。失敗のペイロード: `{ "root": null, "store": null, "references": [], "status": [d] }`、終了コード1。

<a id="411-context---json"></a>
### 4.11 `context --json`
`{ "root": { "path", "source", "store_id"?, "role": "openspec_root" }, "members": [ { "role": "referenced_store", "id", "path"?, "remote"?, "fetch"?, "status": [] } ], "status": [] }`。AVAILABLE = パスが存在し、かつステータスが空。`--code-workspace <path>` は `{folders:[{name,path}]}`（利用可能な参照ストアのみ、`ref:` プレフィックス付き）を書き込みます。JSON モードでは、書き込みが出力の前に実行されるため、書き込みが失敗した場合でも、stdout にはちょうど1つのドキュメントが含まれます。失敗: `{ "root": null, "members": [], "status": [d] }`、終了コード1。

<a id="412-store----json"></a>
### 4.12 `store ... --json`
setup/register: `{ "store": {id, root, metadata_path?}, "registry": {path, registered, already_registered}, "git": {is_repository, initialized, committed}, "created_files": [], "status": [] }`。unregister/remove: `{ "store", "registry": {path, removed}, "files": {deleted, deleted_path, left_on_disk}, "status": [] }`。list: `{ "stores": [{id, root}], "status": [] }`。doctor: `{ "stores": [ { id, root, metadata_path?, openspec_root: {...healthy, status}, metadata: {present, valid, id?, remote}, git: {is_repository, has_commits, has_uncommitted_changes, has_remote, origin_url}, status } ], "status": [] }`（`null` = 不明/調査していない）。健全性の所見は終了コード0で、失敗は対応する null 形式とともに終了コード1です。プロンプトのキャンセルは終了コード130です。

<a id="413-schemas---json--templates---json"></a>
### 4.13 `schemas --json` / `templates --json`
`schemas`: 成功時は素の配列 `[ {name, description, artifacts, source} ]` のままです。正規のルート選択の優先順位を解決し、`--store <id>` を受け付けます。ルート選択の失敗: `{ "schemas": [], "root": null, "status": [d] }`、終了コード1。`templates`: キー付きのオブジェクト `{ "<artifactId>": {path, source} }` で、引き続き cwd ベースであり、root/status キーはありません。

<a id="5-exit-code-contract"></a>
## 5. 終了コードの契約

| 状況 | 終了コード | stdout |
|---|---|---|
| 成功（健全性の所見を含む。doctor/context/store doctor） | 0 | ペイロード |
| `--json` モードでのコマンドの失敗 | 1 | `status: [d]` とそのコマンドの null 形式を持つ、1つの JSON ドキュメント |
| 失敗する項目がある `validate` | 1 | 完全なレポート |
| プロンプトのキャンセル（`store` グループ、人間向けモード） | 130 | stderr のみ |

<a id="6-diagnostic-code-catalog"></a>
## 6. 診断コードのカタログ

<a id="resolution"></a>
### 解決
`no_openspec_root`, `no_root_with_registered_stores`, `no_registered_stores`, `unknown_store`, `store_identity_mismatch`, `unhealthy_store_root`, `store_path_not_supported`, `invalid_store_pointer`, `initiative_option_removed`, `areas_option_removed`; パススルー: `invalid_store_id`, `invalid_store_registry`, `invalid_store_metadata`。

<a id="openspec-root-health-error-no-fix"></a>
### OpenSpec ルートの健全性（エラー、fix なし）
`openspec_store_root_missing`, `openspec_store_root_not_directory`, `openspec_root_missing`, `openspec_root_not_directory`, `openspec_config_missing`, `openspec_config_not_file`, `openspec_specs_not_directory`, `openspec_changes_not_directory`, `openspec_archive_not_directory`。ストアのベータ期間中は、健全なルートでも `openspec/specs/`、`openspec/changes/`、`openspec/changes/archive/` が存在しないことがあります。これらが健全性エラーになるのは、存在するがディレクトリではない場合のみです。

<a id="store-registryidentitystate"></a>
### ストアのレジストリ/識別/状態
`invalid_store_id`, `invalid_store_registry`, `invalid_store_metadata`, `store_registry_busy`, `store_not_found`, `no_store_registry`, `store_registry_changed`, `store_metadata_missing`, `store_metadata_id_mismatch`, `store_metadata_invalid`, `store_id_conflict`, `store_path_conflict`, `store_already_registered`（info）。

<a id="store-setupregisterremove"></a>
### ストアの setup/register/remove
`store_setup_id_required`, `store_setup_path_required`, `store_setup_path_not_directory`, `store_setup_inside_git_repo`, `store_setup_non_empty_directory`, `store_setup_cancelled`, `store_path_required`, `store_path_missing`, `store_path_not_directory`, `store_root_pointer_declared`, `store_register_root_unhealthy`, `store_register_identity_confirmation_required`, `store_register_cancelled`, `store_remote_empty`, `store_remote_requires_hand_edit`, `store_remove_confirmation_required`, `store_remove_cancelled`, `store_remove_path_not_directory`, `store_remove_metadata_missing`, `store_remove_contains_registered_store`, `store_root_missing`（remove では警告、doctor ではエラー）, `store_root_not_directory`。

<a id="store-git"></a>
### ストアの Git
`store_git_init_failed`, `store_git_identity_missing`, `store_git_commit_failed`, `store_git_no_commits`（警告）, `store_clone_fragile_directories`（警告）, `store_remote_divergence`（info、doctor）, `store_checkout_drift`（info、doctor）。

<a id="references-warning"></a>
### 参照（警告）
`reference_invalid_id`, `reference_registry_unreadable`, `reference_unresolved`, `reference_root_unhealthy`, `reference_index_truncated`。

<a id="relationships-warning-doctor-context-keeps-only-the-registry-one"></a>
### 関係性（警告。doctor。context はレジストリのものだけを保持する）
`relationship_registry_unreadable`, `root_pointer_ignored`, `root_pointer_invalid`, `pointer_declarations_inert`。

<a id="archive-json-mode"></a>
### アーカイブ（JSON モード）
`archive_change_name_required`, `archive_change_not_found`, `archive_change_symlink`, `archive_validation_failed`, `archive_confirmation_required`, `archive_tasks_incomplete`, `archive_spec_update_failed`, `archive_spec_validation_failed`, `archive_target_exists`, `archive_retirement_cleanup_failed`, `archive_error`。

<a id="context-writes"></a>
### コンテキストの書き込み
`context_file_exists`, `context_output_dir_missing`。

<a id="fallbacks"></a>
### フォールバック
`doctor_failed`, `context_failed`, `store_error`, `change_error`, `archive_error`。

<a id="known-inconsistencies"></a>
## 既知の不整合

最終監査で記録されたものです。公開済みのキーの名前変更は、このリリースの後に先送りされた製品上の判断です。

1. ~~`--json` モードで、いくつかの失敗経路が、JSON ドキュメントなしで stderr のみを出力していた。~~ 最終的な検証ラウンドで修正: `show`/`validate` の不明な項目や曖昧な項目は `{status:[{code: unknown_item | ambiguous_item, ...}]}` を出力する。`status`/`instructions`/`list`/`show`/`validate` でスローされたエラーは、JSON 対応の失敗ヘルパー（そのコマンドの null 形式 + `status`）を経由する。`store <unknown subcommand> --json` は `{status:[{code: unknown_store_subcommand}]}` を出力する。`list` は、解決の失敗時に `{changes|specs: [], root: null}` の null 形式を保持する。
2. `store_root_missing` は2つの重大度で出力される（remove では警告、store doctor ではエラー）— 文脈依存であり、上記に記載済み。
3. snake_case（ストア系）と camelCase（ワークフロー系）のキーの表記の違い。`root.store_id` はどこでも snake_case。
4. src には、4つの並行するエンベロープ型の宣言が存在する。アーカイブの診断は `target` を持たない。
5. `list --json` は、変更ごとに `status` キーを文字列の列挙型として再利用している。
6. `version` フィールドを持つのは `validate` の出力のみ。
7. `templates` はルート選択を無視する（cwd ベースで、`--store` なし）。
8. 非推奨の名詞形式（`change`/`spec` サブコマンド）は、`root`/`status` を持たない、エンベロープなしのペイロードを出力する。
