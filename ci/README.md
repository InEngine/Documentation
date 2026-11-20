# InEngine CI workflow templates (Step 11)

**Not for operator UI.** Step 11 is quality gates (Pest / PHPStan / Pint / SemVer discipline). Operator UI is **`inengine/modulus-ui` (private Pro)** — later, after the CLI spine is solid.

## Where these live

| Path | Role |
|------|------|
| `Documentation/ci/*.yml` | **Canonical templates** — copy into package repos |
| `Modules/*/.github/workflows/` | Per-package workflows (Modulus, Skeleton, Modulus-Composer, feature modules) |
| Core `.github/workflows/` | Host app CI (checkout path Modulus; not Testbench) |

GitHub **reusable** `workflow_call` workflows need a stable public repo callers can reference. Until an `InEngine/workflows` (or public Documentation) house is chosen, **copy** these files into each package and keep them in sync when the floor changes (PHP 8.5, Laravel 12/13).

## What each template is for

| File | Use when |
|------|----------|
| `package-run-tests.yml` | Spatie/Testbench packages (`inengine/modulus`, generated Skeleton modules, …) — Pest matrix PHP 8.5 × Laravel 12/13 |
| `package-phpstan.yml` | Same packages — Larastan / PHPStan |
| `package-pint.yml` | Same packages — Pint on push (auto-commit style) |
| `package-with-modulus-path.yml` | Packages that path-require Modulus in CI (Skeleton template, Modulus Composer) — checkout sibling `InEngine/Modulus` @ `main` |

## How to adopt

1. Copy the matching file(s) into `your-package/.github/workflows/`.
2. Adjust `paths:` filters / `working-directory` if the package uses a nested checkout layout.
3. For Skeleton-style dual checkout, start from `package-with-modulus-path.yml`.
4. Keep SemVer + Conventional Commits documented in the package README / Skeleton `TEMPLATE.md`.

## Integration testing

- **Package unit/feature:** Pest in the package (Testbench).
- **Host integration:** Core `tests/Feature/*` against a real `inengine/modulus` path/Packagist require (early boot, Artisan `modulus:*`, discovery filter).
- **Composer client:** Modulus Composer tests with path Modulus — not folded into Modulus core CI.
