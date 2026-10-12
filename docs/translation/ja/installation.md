<a id="installation"></a>
# インストール

<a id="prerequisites"></a>
## 前提条件

- **Node.js 20.19.0 以上** — バージョンの確認: `node --version`

<a id="install-with-your-ai-assistant"></a>
## AI アシスタントでインストールする

手作業で行いたくない場合は、シェルコマンドを実行できる任意のコーディングアシスタント（Claude Code、Codex、Cursor、Gemini CLI、Copilot、その他の [対応ツール](supported-tools.md)）に、以下のプロンプトを貼り付けてください。CLI のインストール、このプロジェクトの初期化を行い、実際に何が起きたかを報告します。

下記の手動手順が正式な手順であり、プロンプトはそれを代わりに実行するだけです。アシスタントが途中で止まって何かを返してきた場合、それは意図された動作です。特権が必要な操作の前には確認を求め、シェルの起動ファイルを編集することは決してありません。その部分は、[パッケージマネージャー](#package-managers) と [トラブルシューティング](troubleshooting.md) を参照して、ご自身で完了してください。

```text
Install OpenSpec in this project and set it up for me. Follow these steps in
order, and stop where a step tells you to stop.

1. RUNTIME. Run `node --version`. OpenSpec needs Node.js 20.19.0 or higher. If
   Node is missing or older, say so and stop — don't install Node, switch
   versions, or reconfigure my version manager for me.

2. INSTALL. Use whichever package manager is already on my PATH, preferring npm:
     npm install -g @fission-ai/openspec@latest
     pnpm add -g @fission-ai/openspec@latest
     bun add -g @fission-ai/openspec@latest
     yarn global add @fission-ai/openspec@latest   (Yarn 1.x only)
   Don't pick based on this project's lockfile — a global install has nothing to
   do with how this repo's own dependencies are installed. If none of those four
   is available, stop and tell me — don't improvise an install. (If I'm on Nix,
   point me at the Nix section of the OpenSpec installation docs instead.)
   Show me the exact command and let me confirm before you run it; this installs
   software outside the project, and I may want a different package manager to
   own it.
   Stop and ask me again if the install needs sudo or admin rights, fails with a
   permissions error, or reports that its global bin directory is missing or
   unconfigured. Never edit my shell startup files (.bashrc, .zshrc, .profile,
   fish, PowerShell profile), and never run a setup command that edits them for
   me — show me the change and let me make it.

3. PATH. Run `openspec --version`. If the command isn't found, it may just be
   missing from this shell: tell me where the package manager installed it and
   how to add that directory to PATH for my shell and OS, then stop until I
   confirm. If it prints an older version than the one the install just
   reported, an earlier copy is shadowing it on PATH — tell me both versions
   instead of continuing. If I use a version manager, say so rather than editing
   PATH around it: with nvm or fnm the CLI is tied to the Node version that was
   active when you installed it, and with asdf or volta a shim may need
   regenerating.

4. INITIALIZE. Ask me which AI coding tool or tools I use and map each to an id
   from `openspec init --help` (Copilot is `github-copilot`, Zoo Code is
   `roocode`). `--tools` takes a comma-separated list, so name all of them.
   `openspec init --tools <ids>` deletes leftovers from older OpenSpec versions
   automatically, without asking — including `opsx-*.md` prompt files in my home
   directory (Codex keeps them in ~/.codex/prompts). Before you run it, look for
   those: `.../commands/openspec/` folders, OpenSpec marker blocks in files like
   CLAUDE.md or AGENTS.md, and home-directory `opsx-*.md` prompts. List whatever
   you find and wait for my go-ahead; if you find nothing, say so and carry on
   without asking. An existing `openspec/` folder is not a problem — init
   refreshes it and leaves my specs and changes alone.
   Confirm I'm in the right folder too: init creates `openspec/` wherever it
   runs, including inside a monorepo package.
   Then run: openspec init --tools <ids>

5. REPORT. Don't assume what should exist — tell me what init actually printed:
   how many skills and/or commands it created and where, the config file line,
   any "Setup required" note, and what to restart or reload. Some tools are
   skills-only and correctly create zero command files, so missing commands is
   not a failure on its own. If init said nothing was generated, relay the fix
   it suggested instead of retrying. Finish by telling me how to invoke OpenSpec
   in my tool, and take the exact spelling from the files init created rather
   than from its summary line: the punctuation differs per tool (/opsx:propose
   in some, /opsx-propose in others, @opsx-propose in Amazon Q), and tools that
   get skills instead of commands are invoked by skill name (/openspec-propose,
   or $openspec-propose in Codex, or /skill:openspec-propose in Kimi Code).
```

（注: 上記のプロンプトは、アシスタントに正確に伝わるよう、原文の英語のまま掲載しています。）

このプロンプトにはベンダー固有のものは何もありません。平易な指示と、このページに記載されているのと同じコマンドで構成されています。macOS、Linux、Windows で動作し、手順が許可を必要とする場合は、即興で対処せず意図的に停止します。アシスタントがシェルコマンドを実行できる必要があります。一部の IDE 統合では実行できません。

<a id="package-managers"></a>
## パッケージマネージャー

<a id="npm"></a>
### npm

```bash
npm install -g @fission-ai/openspec@latest
```

<a id="pnpm"></a>
### pnpm

```bash
pnpm add -g @fission-ai/openspec@latest
```

<a id="yarn"></a>
### yarn

```bash
yarn global add @fission-ai/openspec@latest
```

Yarn 2 以降（Berry）では `global` コマンドが削除されました。これらのバージョンでは、代わりに npm、pnpm、または bun で OpenSpec をインストールしてください。グローバル CLI が、プロジェクトのパッケージマネージャーと同じである必要はありません。

<a id="deno"></a>
### deno

Deno では、@latest タグの解釈に問題が生じることがありますが、最初のインストール時にバージョンを指定できます。
その場合は、@latest タグをバージョンに置き換えてみてください。たとえば `@^1.3.1` のようにします。

```bash
deno install --global \
  --allow-read --allow-write --allow-env --allow-sys=cpus,homedir --allow-net=edge.openspec.dev \
  npm:@fission-ai/openspec@latest
# or
deno install --global \
  --allow-read --allow-write --allow-env --allow-sys=cpus,homedir --allow-net=edge.openspec.dev \
  npm:@fission-ai/openspec@^1.3.1
```

注: サブコマンドが config edit、feedback、workspace open のような外部ツールを起動する場合は、スコープを絞った --allow-run=<program> が必要になることがあります。

<a id="bun"></a>
### bun

Bun で OpenSpec をグローバルにインストールできますが、OpenSpec は現在 Node.js 上で動作します。
`PATH` 上で Node.js 20.19.0 以上が使える状態である必要があります。

```bash
bun add -g @fission-ai/openspec@latest
```

<a id="nix"></a>
## Nix

インストールせずに OpenSpec を直接実行します。

```bash
nix run github:Fission-AI/OpenSpec -- init
```

プロファイルにインストールすることもできます。

```bash
nix profile install github:Fission-AI/OpenSpec
```

または、`flake.nix` の開発環境に追加します。

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    openspec.url = "github:Fission-AI/OpenSpec";
  };

  outputs = { nixpkgs, openspec, ... }: {
    devShells.x86_64-linux.default = nixpkgs.legacyPackages.x86_64-linux.mkShell {
      buildInputs = [ openspec.packages.x86_64-linux.default ];
    };
  };
}
```

<a id="verify-installation"></a>
## インストールの確認

```bash
openspec --version
```

<a id="updating"></a>
## 更新

パッケージをアップグレードしてから、各プロジェクトの生成ファイルを更新します。

```bash
npm install -g @fission-ai/openspec@latest   # or pnpm/yarn/bun equivalent
openspec update                              # run inside each project
```

`openspec update` は、設定済みのツール用のスキルファイルとコマンドファイルを再生成するため、スラッシュコマンドがインストール済みのバージョンに合わせて最新の状態に保たれます。また、新しい CLI が公開されているかどうかも確認し、アップグレードを提案します。新しいワークフローを利用できるようにするのはアップグレードそのものだからです。[CLI リファレンス](cli.md#openspec-update) を参照してください。

<a id="uninstalling"></a>
## アンインストール

`openspec uninstall` コマンドはありません。OpenSpec は、グローバルパッケージとプロジェクト内のいくつかのファイルにすぎないからです。削除はいくつかの手動の手順で済み、ここでの作業がソースコードに触れることはありません。

**1. グローバルパッケージを削除する:**

```bash
npm uninstall -g @fission-ai/openspec   # or: pnpm rm -g / yarn global remove / bun rm -g
```

**2. プロジェクトから OpenSpec を削除する（任意）。** 仕様や変更が不要になった場合は、`openspec/` ディレクトリを削除します。

```bash
rm -rf openspec/
```

実行する前によく考えてください。`openspec/specs/` と `openspec/changes/archive/` は、システムがどう振る舞い、なぜ変更されたかの記録です。その履歴が必要になるかもしれない場合は、アンインストールした後でも、このフォルダを残して（または git に保存して）おいてください。

**3. 生成された AI ツールのファイルを削除する（任意）。** OpenSpec は、`.claude/skills/openspec-*/` や `.cursor/commands/opsx-*` のような、ツールごとのディレクトリにスキルファイルとコマンドファイルを書き込みます。設定したツールの `openspec-*` スキルと `opsx-*` コマンドを削除してください。ツールごとの正確なパスは、[対応ツール](supported-tools.md) に記載されています。

`CLAUDE.md` や `AGENTS.md` のようなファイルに OpenSpec のマーカーブロックもある場合は、そのブロックを手動で削除してください。それらのファイルのあなた自身の内容は、そのまま残して構いません。

<a id="next-steps"></a>
## 次のステップ

インストール後、プロジェクトで OpenSpec を初期化します。

```bash
cd your-project
openspec init
```

詳しい手順は [はじめに](getting-started.md) を参照してください。
