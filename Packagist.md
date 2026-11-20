# Packagist publish checklist

**Status:** Beta published — `inengine/modulus` and `inengine/modulus-composer` at **`v0.1.0-beta.2`**. Alert at **`v1.0.2`**. Core project beta **`v0.1.0-beta.1`** (GitHub Releases; not Packagist).

**Immutable tags:** Never move/retarget a published GitHub/Packagist tag. Fix on **`dev`**, PR into **`main`**, then cut `v0.1.0-beta.N+1` (or stable) if a tagged tree itself must change. See [Git workflow.md](Git%20workflow.md).

## Packages (public MIT)

| Package | GitHub | Tag |
|---------|--------|-----|
| `inengine/modulus` | https://github.com/InEngine/Modulus | `v0.1.0-beta.2` |
| `inengine/modulus-composer` | https://github.com/InEngine/Modulus-Composer | `v0.1.0-beta.2` |
| `inengine/alert` | https://github.com/InEngine/Alert | `v1.0.2` |

```bash
composer require inengine/modulus:^0.1@beta
composer require inengine/modulus-composer:^0.1@beta
```

Core (`inengine/core`) stays a **project** (`create-project` / clone). Published Core requires both packages `^0.1@beta`. Path `*@dev` only in gitignored `composer.local.json` (from `composer.local.dist.json`).

## Not on Packagist

| Repo | Why |
|------|-----|
| [InEngine/Skeleton](https://github.com/InEngine/Skeleton) | Public GitHub **template** only (forked from [spatie/package-skeleton-laravel](https://github.com/spatie/package-skeleton-laravel)). Generated `inengine/{feature}` packages may be Packagist-published — the template itself must not. |
| [InEngine/Core](https://github.com/InEngine/Core) | `type: project` create-project source (publish separately when ready). |

## Steps (next stable)

1. Merge `dev` → `main` so the release commit is on `main`.
2. Tag annotated release: `git tag -a v0.1.0 -m "v0.1.0"` && `git push origin v0.1.0`.
3. Packagist webhook auto-updates (packages already submitted).
4. Switch Core / Composer published constraints from `^0.1@beta` → `^0.1`.

## Local / CI vs release

- **Never commit** `composer.local.json`.
- **Release / prod Core:** do not ship `composer.local.json` (release workflow strips it).
- **CI:** Core keeps path Modulus checkout until CI is switched to Packagist-only (optional follow-up).
- **Modulus Composer** published `composer.json` has **no** path `repositories`; local/CI copy `composer.local.dist.json` → `composer.local.json`.

## Private Pro (not Packagist public)

- `inengine/modulus-ui` → private repo / private Packagist later.
