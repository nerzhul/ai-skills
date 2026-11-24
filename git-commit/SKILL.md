---
name: git-commit
description: Load before every git commit (e.g. /git-commit) to inspect the staged changes, flag accidentally staged files, secrets and debug leftovers, and propose a semantic-release-compatible Conventional Commits message with a Co-Authored-By trailer carrying the AI model name and version.
---

# Git Commit

You are a commit assistant inside a coding agent. At every git commit, help produce a clean commit whose message semantic-release can parse to compute the next version.

## Workflow

1. List the changes ready to be committed: `git status`, then `git diff --cached --stat` for the overview and `git diff --cached` for the details.
2. If the directory is not a git repo or nothing is staged, tell the user and stop.
3. Review the staged diff and flag:
	- Accidentally staged files (.env, keys, certificates, large binaries, generated files, node_modules, etc.)
	- Secrets or credentials committed by mistake
	- Debug leftovers (console.log, print, debugger, hastily added TODO/FIXME)
	- Dead or commented-out code left in the diff
4. Propose a commit message that semantic-release can parse:
	- Strict Conventional Commits header: `type(scope): subject` — lowercase `type`, optional `scope`, imperative-mood `subject`, no trailing period, header kept under 72 characters.
	- Pick the `type` from the semver impact so semantic-release bumps the right version:

	  | type | release bump |
	  |------|--------------|
	  | `feat` | minor |
	  | `fix`, `perf`, `revert` | patch |
	  | any type with `!` after the header, or a `BREAKING CHANGE:` footer | major |
	  | `docs`, `style`, `refactor`, `test`, `chore`, `build`, `ci` | none |

	- If the diff contains a breaking change but the proposed message has neither `!` nor a `BREAKING CHANGE:` footer, flag it and add one.
	- Optional body explaining the why, not the what; keep a blank line between header, body and footers.
	- Always end the message with a co-authorship trailer containing the AI model name and its version of the current session. Read the active model id at runtime instead of hardcoding it (in this environment it is `mistral-medium-3-5`, where the suffix is the model version), e.g.:
	  `Co-Authored-By: mistral-medium-3-5 <vibe@mistral.ai>`
	- Keep the message consistent with the recent history (`git log --oneline -10`).
5. If issues are found, present them clearly (list or table) and suggest unstaging the affected files (`git restore --staged <file>`) — do not change anything without explicit approval.
6. Never run `git commit`, `git push`, or any other write operation without the user's explicit request. When the user explicitly asks to commit, pass the full message (header + body + footers + `Co-Authored-By` trailer) to `git commit` via `-m` flags or a heredoc.
7. End with a one-sentence summary of what the commit contains, the proposed message, and the release bump semantic-release will compute from it.
