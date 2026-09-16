# untrust

[English](README.md)

Claude Code の `~/.claude.json` から、プロジェクトのエントリを削除するツールです。

Claude Code 専用です。Anthropic の公式ツールではありません。

Claude Code は、信頼ダイアログの承認状態を含むプロジェクトごとの状態を、`~/.claude.json` の `projects` キーに保存しています。`untrust` は、指定したプロジェクトパスのエントリを削除します。削除されるのは信頼の記録だけではなく、エントリ全体です。

## 必要なもの

- bash
- jq
- `realpath`、`cmp`、`stat`（GNU 版と BSD 版のどちらでも可）

macOS（bash 3.2、BSD 版 `stat`）と Ubuntu 24.04（bash 5.2、GNU coreutils 9.4）で動作を確認しています。

## インストール

```sh
git clone https://github.com/nemalabs/untrust.git
cp untrust/bin/untrust ~/.local/bin/
```

`PATH` に含まれるディレクトリならどこに置いても動きます。

## 使い方

```
untrust [-n] [-c FILE] [PATH]
```

| 引数 / オプション | 説明 |
|---|---|
| `PATH` | 対象のプロジェクトパス。省略時はカレントディレクトリ。 |
| `-n`, `--dry-run` | 一致したエントリを表示するだけで、何も変更しない。 |
| `-c`, `--config FILE` | config のパス。`CLAUDE_CONFIG` より優先される。 |
| `-h`, `--help` | ヘルプを表示する。 |

```sh
untrust -n                                   # カレントディレクトリのエントリを確認する
untrust ~/work/project                       # ~/work/project のエントリを削除する
untrust -c ~/other/.claude.json ~/work/project
```

## 環境変数

| 変数 | 説明 | 既定値 |
|---|---|---|
| `CLAUDE_CONFIG` | config のパス | `~/.claude.json` |
| `UNTRUST_BACKUP_DIR` | バックアップの保存先 | `~/.claude/untrust-backups` |
| `UNTRUST_BACKUP_KEEP` | 残すバックアップの数（`0` で削除しない） | `10` |

`CLAUDE_CONFIG_DIR` は読みません。config が `~/.claude.json` 以外の場所にある場合は、`-c` か `CLAUDE_CONFIG` で指定してください。

## 動作

- `PATH` は絶対パスにしたうえで、`.` と `..` を字面で解決します。その結果を `projects` のキーと完全一致で照合し、一致しなければシンボリックリンクを解決した実パスでもう一度照合します。
- エントリを削除する前に、config 全体のバックアップを日時付きのファイル名で保存します。残すのは新しいものから `UNTRUST_BACKUP_KEEP` 個だけです。
- config がシンボリックリンクの場合は、リンク先のファイルを書き換えます。リンクはそのまま残り、ファイルの権限も保たれます。
- 書き換えている途中で config が変更された場合は、config を置き換えずに終了します。
- 新しく作るファイルとディレクトリは `umask 077` で作成します。

## 注意

- 実行中の Claude Code セッションは、終了時にエントリを書き戻すことがあります。`untrust` を実行する前に、そのプロジェクトのセッションを閉じてください。
