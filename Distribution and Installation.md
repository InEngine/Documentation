# InEngine — Distribution and Installation

Living notes on how operators and product repos should **acquire** Core, Modulus, and feature modules.

Related: [Plan and Design.md](Plan%20and%20Design.md) · [Packagist.md](Packagist.md).

---

## Short answer

Use **both** shapes — they solve different jobs:

| Artifact | Composer shape | How you install it |
|----------|----------------|--------------------|
| **Core** | `type: project` (like `laravel/laravel`) | `composer create-project inengine/core my-app` |
| **Modulus** | package (`inengine/modulus`) | Required **by Core**; also `composer require` into any Laravel host that wants the runtime |
| **Feature modules** | packages (`inengine/{feature}`) | Create from public [Skeleton](https://github.com/InEngine/Skeleton) template (not Packagist), then `composer require` + `modulus:enable` |
| **Modulus extensions** | packages (`inengine/modulus-{role}`) | Optional `composer require` — public MIT for Composer; **`modulus-ui` private Pro**; marketplace/inspect planned |
| **Products** | `type: project` | `composer create-project …` **or** clone + `composer install` — a Core-shaped app with product modules pre-declared |

Do **not** try to ship Core only as a library you `require` into a blank Laravel app as the primary path. Core *is* the opinionated host (routes shell, Vite, Pest, Livewire, early Modulus boot). Treating it like a package fights that role and duplicates what `create-project` already does well.

Do **not** fold a whole product into one giant monolith package. Reusable product domains belong in **`inengine/{feature}`** (or shared) modules; the product repo stays the composition root + branding + provisioning glue.

---

## Why not “only packages” or “only create-project”?

### Only `composer require` (everything is a package)

**Works well for:** Modulus, feature modules, Modulus extensions — anything that enables/disables and must SemVer independently.

**Fails as the primary story for Core:** a blank `laravel new` plus twenty `require`s recreates Core poorly, drifts from the blessed stack, and skips Core’s composition-root assumptions (no default User in Core, Modulus boots early, discovery gating).

**Fails as the only story for a product:** operators need “spin up the product,” not “assemble fourteen packages by hand.”

### Only `composer create-project` (everything is a project)

**Works well for:** Core and product apps (deployable roots, `.env`, Herd sites, CI).

**Fails for modules:** you cannot enable/disable or share modules across products if they only exist as forked app trees. Modulus’s whole point is **Composer packages + enable state**.

### Hybrid (recommended)

```mermaid
flowchart TB
  createCore["composer create-project inengine/core"]
  createProduct["composer create-project inengine/product"]
  requireMod["composer require inengine/modulus-* / inengine/feature"]
  enable["modulus:enable ..."]
  createCore --> requireMod
  createProduct --> requireMod
  requireMod --> enable
```

1. **Greenfield platform / custom product** → `create-project inengine/core`, then require + enable modules.  
2. **Named product** → `create-project` (or git clone) of the **product project**, which already lists Core-equivalent stack + product modules.  
3. **Day-2 add-ons** → `composer require` + Modulus enable (marketplace / Modulus Composer later).

Install ≠ enable (see Plan and Design): acquire with Composer, then `modulus:enable`.

---

## Recommended public install flows

### A. Core (platform host)

```bash
composer create-project inengine/core:^0.1@beta my-engine   # when Core is on Packagist
cd my-engine
php bin/setup                    # or runs from post-create-project-cmd
php artisan modulus:list
php artisan modulus-composer:require inengine/alert
php artisan modulus:enable inengine/alert
```

**Existing git / non-empty directory:** `create-project` cannot install there. Clone Core (or your fork), `composer install`, then `php bin/setup`.

Local monorepo: `composer.local.json` from `composer.local.dist.json` → path `*@dev`. Published Core requires `inengine/modulus` + `inengine/modulus-composer` `^0.1@beta`.

### B. Product apps

Target UX when distribution is ready:

```bash
composer create-project inengine/my-product my-site
# or: git clone … && composer install
cd my-site
cp .env.example .env && php artisan key:generate
php artisan migrate
php artisan modulus:enable …   # if product modules ship disabled by default
```

Product `composer.json` stays `type: project`. It should **not** depend on “copy Core by hand forever”; long-term options (pick when packaging Core for release):

1. **Synced Core sibling** — product is a create-project sibling of Core: Laravel app tree + requires modules. Simple; Core fixes merge forward carefully.  
2. **Core as path/VCS project template** — release Core tags; product create-project is a thin preset (extra requires, seeders, product config) generated from Core.  
3. **Avoid:** `composer require inengine/core` into Laravel as the main product path (Core is not designed as a drop-in package).

### C. Module authors

```bash
# from InEngine/Skeleton
composer require inengine/my-module   # in a Core or product host
php artisan modulus:enable inengine/my-module
```

---

## Local monorepo layout (today)

While packages are unpublished:

| Path | Role |
|------|------|
| `Projects/InEngine/Core` | Host / create-project source |
| `Projects/InEngine/Modules/*` | Path-required packages |

Wire with Composer path repositories (and optional merge-plugin), e.g.:

```json
{
  "repositories": [
    { "type": "path", "url": "../InEngine/Modules/Modulus", "options": { "symlink": true } },
    { "type": "path", "url": "../InEngine/Modules/*", "options": { "symlink": true } }
  ]
}
```

Production replaces path repos with Packagist / private Packagist / Git tags. Keep gitignored `composer.local.json` for local/dev only.

---

## Licensing (locked 2026-11-19)

| Work | License / visibility |
|------|----------------------|
| Core, Modulus, Modulus Composer, Skeleton | **MIT** — public GitHub / Packagist when published |
| Feature modules | Prefer **MIT** unless a product needs otherwise |
| `inengine/modulus-ui` | **Private Pro** — commercial; private repo / private Composer; not public Packagist |

Do not put Pro UI code inside MIT packages. Pro depends on Modulus; Modulus never depends on Pro.

## Decision checklist (when locking “how we ship”)

1. **Publish target:** public Packagist vs private Packagist vs VCS-only for early beta.  
2. **Core versioning:** SemVer tags on `inengine/core` create-project; document PHP/Laravel floor.  
3. **Product presets:** create-project name and which modules are required vs optional add-ons.  
4. **Hosted vs self-serve:** hosted provision clones/create-projects the product; self-serve docs mirror the same Composer entrypoint.  
5. **Modulus marketplace:** optional later; does not replace Composer as source of truth for acquire.  
6. **`modulus-ui`:** private Pro packaging (license key / private Packagist) when scaffolded.
