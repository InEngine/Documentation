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

## Agents / Cursor

Core ships an always-on Cursor rule (`.cursor/rules/git-branch-workflow.mdc`). Mirror it in package repos when present.
