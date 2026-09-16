# untrust

[日本語](README.ja.md)

Remove a project entry from Claude Code's `~/.claude.json`.

This tool is for Claude Code only. It is not an official Anthropic tool.

Claude Code keeps per-project state, including trust-dialog acceptance, under the `projects` key of `~/.claude.json`. `untrust` deletes the entry for one project path. The whole entry is removed, not only the trust decision.

## Requirements

- bash
- jq
- `realpath`, `cmp`, `stat` (GNU or BSD)

Checked on macOS (bash 3.2, BSD `stat`) and Ubuntu 24.04 (bash 5.2, GNU coreutils 9.4).

## Install

```sh
git clone https://github.com/nemalabs/untrust.git
cp untrust/bin/untrust ~/.local/bin/
```

Any directory on your `PATH` works.

## Usage

```
untrust [-n] [-c FILE] [PATH]
```

| Argument / option | Description |
|---|---|
| `PATH` | Target project path. Defaults to the current directory. |
| `-n`, `--dry-run` | Report the matched entry without modifying anything. |
| `-c`, `--config FILE` | Config path. Overrides `CLAUDE_CONFIG`. |
| `-h`, `--help` | Show help. |

```sh
untrust -n                                   # check the entry for the current directory
untrust ~/work/project                       # remove the entry for ~/work/project
untrust -c ~/other/.claude.json ~/work/project
```

## Environment

| Variable | Description | Default |
|---|---|---|
| `CLAUDE_CONFIG` | Config path | `~/.claude.json` |
| `UNTRUST_BACKUP_DIR` | Backup directory | `~/.claude/untrust-backups` |
| `UNTRUST_BACKUP_KEEP` | Number of backups to keep (`0` disables pruning) | `10` |

`CLAUDE_CONFIG_DIR` is not read. If your config is somewhere other than `~/.claude.json`, pass it with `-c` or `CLAUDE_CONFIG`.

## Behavior

- `PATH` is made absolute and `.` / `..` are resolved lexically. The result is matched against the keys of `projects` by exact string. If nothing matches, the physical path (symlinks resolved) is tried.
- A timestamped backup of the whole config is written before the entry is removed. Only the newest `UNTRUST_BACKUP_KEEP` backups are kept.
- If the config is a symlink, the file it points to is rewritten and the link is kept. The file mode is preserved.
- If the config changes while `untrust` is rewriting it, `untrust` exits without replacing the config.
- New files and directories are created with `umask 077`.

## Notes

- A running Claude Code session may write the entry back when it exits. Close sessions for the project before running `untrust`.
