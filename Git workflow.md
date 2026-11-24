# Git workflow (InEngine)

**Standard for all InEngine repos and packages:** develop on **`dev`**, open a pull request into **`main`** or **`master`**.

| Repo type | Default (work) branch | Integration / release branch |
|-----------|----------------------|------------------------------|
| [Core](https://github.com/InEngine/Core) | `dev` | `master` |
| Packages (Modulus, Modulus Composer, Alert, …) | `dev` | `main` |
| [Skeleton](https://github.com/InEngine/Skeleton) | `dev` | `main` |
| [Documentation](https://github.com/InEngine/Documentation) | `dev` | `master` |

## Rules

1. Commit day-to-day work on `dev` (or a short-lived feature branch merged to `dev` first).
2. Open a PR: `dev` → `main`/`master`. Wait for CI green, then merge.
3. Cut GitHub / Packagist releases from the integration branch after merge. **Never** move or retarget a published tag.
4. Do not push product commits directly to `main`/`master` except release automation that already runs on that branch (e.g. Core `chore(release):`).

## Commit attribution

- **Author / committer:** **James Johnson** `<james.johnson@excellentingenuity.com>` (GitHub: **excellentingenuity**).
- No `Co-authored-by: Cursor` (or any agent / IDE) trailers.
- Do not mention Cursor, Copilot, or other AI tools in commit messages, PR titles, or PR bodies unless explicitly requested.
- Repos may ship `.githooks/commit-msg` to strip Cursor co-author lines; enable with `git config core.hooksPath .githooks` locally if desired.

## Project rules

Always-on Cursor rules (`.cursor/rules/git-branch-workflow.mdc`, `.cursor/rules/co-authored-by-jarvis.mdc`) document the same standards for agents. Mirror them in package repos when present.
