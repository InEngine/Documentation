# Packagist publish checklist

## Packages (public MIT)

| Package | GitHub | First tag |
|---------|--------|-----------|
| `inengine/modulus` | https://github.com/InEngine/Modulus | `v0.1.0` |
| `inengine/modulus-composer` | https://github.com/InEngine/Modulus-Composer | `v0.1.0` |

Core (`inengine/core`) stays a **project** (`create-project` / clone); it requires `inengine/modulus` `^0.1` from Packagist in production. Local/CI use gitignored `composer.local.json` (from `composer.local.dist.json`) for path `*@dev`.

## Not on Packagist

| Repo | Why |
|------|-----|
| [InEngine/Skeleton](https://github.com/InEngine/Skeleton) | Public GitHub **template** only (forked from [spatie/package-skeleton-laravel](https://github.com/spatie/package-skeleton-laravel)). Generated `inengine/{feature}` packages may be Packagist-published — the template itself must not. |

## Steps

1. Repos **public** on GitHub.
2. Default branch `main` contains the release commit (Modulus: merge `feat/modulus-state-store`).
3. Tag annotated releases: `git tag -a v0.1.0 -m "v0.1.0"` && `git push origin v0.1.0`.
4. Submit on [packagist.org](https://packagist.org/packages/submit) (or API) with GitHub URL.
5. Enable Packagist GitHub Service / webhook so new tags auto-update.
6. Verify: `composer show inengine/modulus` / `inengine/modulus-composer`.

## Local / CI vs release

- **Never commit** `composer.local.json`.
- **Release / prod Core:** delete `composer.local.json` before deploy (release workflow strips it).
- **Modulus Composer** published `composer.json` has **no** path `repositories`; local/CI copy `composer.local.dist.json` → `composer.local.json`.

## Private Pro (not Packagist public)

- `inengine/modulus-ui` → private repo / private Packagist later.
- InnKeeper → unreleased source until product publish.
