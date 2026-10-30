**中文** · [README.md](./README.md)

<!-- badges -->
[![Node](https://img.shields.io/badge/Node.js-≥18-339933?logo=node.js&logoColor=white)](https://nodejs.org)
[![Zero-Deps](https://img.shields.io/badge/dependencies-zero-0f9d58?logo=npm&logoColor=white)](https://www.npmjs.com/)
[![License](https://img.shields.io/github/license/huanweide/commitdoctor)](LICENSE)
<!-- /badges -->

# commitdoctor

A zero-dependency, single-file CLI that audits your git commit-message hygiene. It scans the format smells in your most recent N commit messages, produces a 0–100 health score, and works as a CI gate.

It complements [`repodoctor`](https://github.com/huanweide/repodoctor) (repo engineering hygiene: dangerous files / large files / LICENSE / noise-commit ratio): repodoctor looks at "repo engineering hygiene", commitdoctor looks at "commit-message format discipline" — together they form the git-layer governance pair.

## Who should use it / Who shouldn't

- **Use it if** your team wants a readable, traceable commit history; you just inherited a repo with a messy history and want to quantify it first; or you want a commit-discipline gate in CI.
- **Skip it if** you already enforce commitlint + Conventional Commits everywhere with a clean history (it still works as a supplement); or you uniformly squash-merge and the history is naturally tidy.

## Features

- **Zero dependency**: pure Node.js stdlib, single file `index.js`, no `npm install`.
- **Scan immediately**: `node index.js` scans the current repo.
- **Health score**: severity-weighted (high×3 / medium×2 / low×1) into a 0–100 score.
- **CI gate**: `--fail-on-high` / `--max-high` / `--max-medium` / `--max-issues` / `--min-score`.
- **Machine-readable**: `--json` outputs structured results.
- **Read-only & safe**: only calls system `git` to read logs; no network, no file writes, no execution surface.

## Install / Use

```bash
# Run directly (Node >= 18)
node index.js

# Or via npx (after publish)
npx commitdoctor
```

## Rules

| Rule | Severity | Description |
| --- | --- | --- |
| `empty-subject` | high | Commit subject line is empty |
| `subject-too-long` | high | Subject line exceeds 100 chars (hard cap) |
| `has-secret` | high | Message looks like a leaked secret / credential (incl. private-key blocks) |
| `subject-long` | medium | Subject line exceeds the recommended length (default 72) |
| `has-todo-marker` | medium | Message contains `TODO` / `FIXME` / `HACK` / `WIP` |
| `subject-trailing-period` | low | Subject line ends with punctuation |
| `subject-leading-space` | low | Subject line has leading whitespace |
| `subject-trailing-space` | low | Subject line has trailing whitespace |
| `merge-no-desc` | low | Merge commit without description (skip with `--ignore-merges`) |
| `no-body-separator` | low | Missing blank line between body and subject |
| `draft-markers` | low | Message contains `TMP` / `DEBUG` / `temp` / `asdf` |
| `subject-emoji` | low | Subject contains emoji noise (on by default; disable with `--no-check-emoji`) |
| `non-standard-type` | low | Looks like a type prefix but format / casing is off |

## Health score

Each issue deducts by severity (high 3 / medium 2 / low 1): `health = max(0, 100 - total deductions)`.

## CI gate example

```yaml
# .github/workflows/commitlint.yml
- name: commitdoctor
  run: npx commitdoctor --fail-on-high --max-medium 5 --min-score 90
```

Exit codes: `0` pass; `1` gate failed; `2` argument error.

## Options

```
-p, --path <dir>          target git repo (default: current dir)
-n, --max-commits <n>     scan the last n commits (default 50)
    --since <spec>        only scan after this date (passed to git --since, e.g. "2.weeks")
    --max-subject-len <n> recommended subject length cap (default 72)
    --ignore-merges       skip merge commits
    --no-check-emoji      disable emoji detection
    --json                output machine-readable JSON
    --fail-on-high        fail if any high issue exists
    --max-high <n>        fail if high count exceeds n
    --max-medium <n>      fail if medium count exceeds n
    --max-issues <n>      fail if total issues exceed n
    --min-score <s>       fail if health score is below s (0-100)
-V, --version             print version
-h, --help                print help
```

## Limitations

- `has-secret` does a lightweight regex match at the message level and may occasionally hit legitimate descriptions containing words like `token` / `access_key`. It flags a *high-severity plaintext-credential-leak signal* and complements repodoctor's source-layer secret scan; it does not replace a dedicated secret scanner.
- It does not enforce Conventional Commits; it only gives a low-severity hint when something "looks like a type prefix but is malformed".

## Roadmap

Rule set is stable. Possible future additions: configurable severity overrides and a conventional-commit preset. No breaking changes planned.

## Contributing

Issues and pull requests are welcome. Keep it single-file and zero-dependency.

## ⭐ Star

If commitdoctor ever cleaned up a messy commit history for you, a Star ⭐ helps others find it: https://github.com/huanweide/commitdoctor

## Author

[huanweide](https://github.com/huanweide)

## License

MIT
