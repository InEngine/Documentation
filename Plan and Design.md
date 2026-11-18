# InEngine — Plan and Design

Living design notes for the modular platform. Updated 2026-11-03 (extension naming + Composer sub-module) from Core / Modulus planning.

## Vision

InEngine is a **modular application system**: a thin Laravel host (**Core**) plus **Composer modules**, so products can look plug-and-play (WordPress-like) across many use cases — stores, LMS, CMS, CRM, and more — by enabling modules and applying light configuration.

Core stays free of product features and of a default Laravel `User`. Identity/auth belongs in a future Users module. **Core requires Modulus** and boots it early so module discovery gating works before feature providers load.

## Platform roles

| Piece | Role |
|-------|------|
| **Core** | Opinionated Laravel 13 / PHP 8.5 shell (Livewire, Pest, Tailwind, Pint, PHPStan). Requires Modulus; composition root only. |
| **Modulus** | Module runtime composed of focused classes/services (not a single god object): state, discovery sync, lifecycle, CLI, Composer/Packagist client submodule. |
| **Feature modules** | SemVer’d Composer packages (`inengine/*`) — Spatie layout, Pint/Pest/PHPStan, CLI/API first. |
| **Products** | Apps on Core + modules (e.g. Provost, **InnKeeper**) assembled by which modules are installed and enabled. |

Local path wiring: Core `composer.local.dist.json` → `composer.local.json` with path repos under `../Modules/*`. Production/CI require published SemVer constraints of `inengine/modulus` (and modules) from Packagist / private Packagist / VCS.

**How to ship / install** (create-project vs require, InnKeeper stub): see [Distribution and Installation.md](Distribution%20and%20Installation.md). Product stub: `../../InnKeeper` (sibling under `Projects/`).

---

## Design principles (summary)

1. **CLI / API first; UI optional** — Artisan covers every UI capability; same PHP API underneath.  
2. **Spatie-style packages** — `laravel-package-tools` + planned `InEngine/module-skeleton`.  
3. **Laravel `extra.laravel.providers` only** — no parallel `extra.inengine` metadata.  
4. **Register-with-Modulus handshake** — if Modulus present, register and stay disabled until enabled; if absent, boot standalone.  
5. **File-backed state via `ModulusStateStore`** — not DB; atomic temp-write + swap (nWidart-style durability).  
6. **Enable/disable via Laravel package discovery** — decorate/filter `PackageManifest` / effective `packages.php` load; do **not** fight Composer by rewriting host `dont-discover` on every toggle.  
7. **Inter-module: events bus + contracts/DTOs** — discover/notify via bus; typed data via interfaces and DTOs.  
8. **Non-destructive disable/uninstall** — never wipe module data by default; install/enable/upgrade own schema via migrations.  
9. **SemVer + Pint + Pest + PHPStan** — required for Core, Modulus, and every feature module.  
10. **`modulus:doctor`** — diagnose drift between Composer, state file, and discovery cache.  
11. **Modulus extensions are separate packages** — `inengine/modulus-{role}` (composer, marketplace, ui, inspect); never fold Composer hooks or operator UI into runtime core.  
12. **Feature modules are not `modulus-*`** — use `inengine/{feature}` from the module skeleton.

---

## Modulus internal architecture

Modulus is **several collaborating classes** (and optional subpackages), not one monolithic class.

| Component | Responsibility |
|-----------|----------------|
| **`ModulusStateStore`** | Read/write enabled/disabled (and installed metadata) on disk; **atomic temp file + rename/swap** |
| **`ModuleRepository` / registrar** | List modules, enable/disable/uninstall orchestration, dependency checks |
| **`ModuleServiceProvider` (base)** | Spatie provider + Modulus detection, register-with-Modulus, `shouldBoot()` |
| **`PackageManifest` sync** | Filter effective auto-discovered providers from Laravel’s discovery facilities (modules by default; optionally all packages) |
| **Lifecycle / hooks** | Activate ≠ deactivate ≠ uninstall; migrate-on-enable/upgrade |
| **Event bus integration** | Dispatch/listen for module and domain events |
| **Contracts + DTOs** | Shared interfaces for cross-module APIs; DTOs for payloads |
| **Artisan commands** | Thin CLI over the repository API, including **`modulus:doctor`** |
| **Marketplace extension** (not core) | Separate package: browse known module sources, search/require/install via Composer — CLI/UI over Modulus APIs |

**Why:** SOLID — swap storage, discovery sync, or Composer client without rewriting CLI/UI. Test each seam in Pest. Marketplace stays an optional extension so Modulus core stays a runtime, not an app store.

---

## State file + Laravel discovery (how enable/disable works)

### `ModulusStateStore` (source of truth for enabled)

- Path configurable (`config/modulus.php`, e.g. `storage/modulus/modules.json`).  
- Shape roughly: Composer package name → `{ "enabled": bool, "version": "…", … }`.  
- **Writes:** write temp file → `fsync`/flush → atomic rename over the live file (same durability idea as nWidart’s status file, without inventing a second framework).  
- **Policy:** document committed vs gitignored (team-shared defaults vs machine-local overrides).  
- New discovers default to **`enabled: false`**.

### Laravel facilities (effective load set)

Do **not** mutate the host app’s `composer.json` `dont-discover` on every toggle.

Instead:

1. Composer packages keep declaring `extra.laravel.providers`.  
2. Laravel’s `PackageManifest` / `bootstrap/cache/packages.php` remains the framework’s discovery cache.  
3. Modulus **decorates or replaces** `PackageManifest` (or filters the provider list it exposes) so **disabled** packages Modulus manages are omitted from the effective auto-discovered providers.  
4. After enable/disable/discover: refresh discovery cache as Laravel expects so HTTP and Artisan agree.

**CLI/UI listing** always comes from Modulus state + Composer inventory (`list` / `status` / `doctor`), not from “what Laravel currently booted.” Disabled packages remain visible as disabled.

### Discovery scope (config)

| Mode | Behavior |
|------|----------|
| **`modules` (default)** | Only packages that opt into Modulus (base `ModuleServiceProvider` / tracked as modules) are enable/disable gated. Ordinary Laravel packages (Livewire, Debugbar, …) always discover as usual. |
| **`all` (optional)** | Modulus may also track and gate **any** auto-discovered Composer Laravel package via the same PackageManifest filter + state store. |

When `all` is on: never allow disabling Modulus itself or a small hard denylist (e.g. framework-critical packages); support allow/deny lists in config. Plain packages get **manifest filtering only** (they have no Modulus handshake). Modulus modules still get both layers.

**Why:** Default stays safe and narrow; power users can optionally treat Modulus as a full discovery gate without rewriting `composer.json`.

### Defense in depth (required for Modulus modules)

Two independent gates — both required for feature modules:

1. **PackageManifest filter** — disabled module providers are not registered by Laravel.  
2. **Base `ModuleServiceProvider` handshake** — if a provider instance still runs (stale cache, manual register, drift), it **registers with Modulus** and **must not boot** application services unless Modulus reports enabled. If Modulus is absent, boot standalone (third-party reuse).

**Why:** Manifest filtering is the primary load gate; provider-side `shouldBoot()` / enable check is the safety net.

---

## Modulus extensions (sub-modules) — naming and ownership

Anything that is **not** the enable/disable/discovery runtime stays out of `inengine/modulus` and ships as a separate Composer package. Prefer a consistent **`inengine/modulus-{role}`** name (GitHub `InEngine/modulus-{role}`) so operators can see “this extends Modulus” at a glance.

| Package (planned) | Role | Notes |
|-------------------|------|--------|
| **`inengine/modulus`** | Runtime core | State, PackageManifest filter, repository, handshake, `modulus:*` lifecycle CLI |
| **`inengine/modulus-composer`** | Composer client | `composer require` / `remove` / update wrappers; Packagist / private Packagist / Git; optional Composer script/event hooks that notify Modulus after acquire/remove → `discover` (disabled) → explicit enable. **Not** enable/disable itself |
| **`inengine/modulus-marketplace`** | Browse / catalog UX | Search known module sources; may depend on `modulus-composer` for install; CLI first, optional UI later |
| **`inengine/modulus-ui`** | Operator UI | Livewire (or similar) screens that call Modulus + extension APIs; no Composer shelling of its own |
| **`inengine/modulus-inspect`** | Capability catalog | Inspect / compat / integrate docs (Step 14); richer than Composer `suggest` |

**Rules:**

1. Core never requires UI or Composer-shell packages.  
2. **Install ≠ enable** — `modulus-composer` / marketplace end in discover→disabled→`modulus:enable`.  
3. Composer **hooks** (post-require / pre-remove integration with Modulus) live in **`modulus-composer`**, not core.  
4. Feature modules stay `inengine/{feature}` (from `module-skeleton`), not `modulus-*`.  
5. Names are locked as working titles until first publish; rename only if Packagist conflict.

**Why:** Keeps the runtime lean; lets hosts adopt Composer automation and UI independently; clear package names for Composer/UI/inspect concerns.

---

## Guidelines for Modulus-compatible feature modules

Standing rules for authors (and agents) building `inengine/*` feature modules. The skeleton must encode these; do not invent parallel conventions.

1. **Package name:** `inengine/{feature}` (e.g. `inengine/alert`). Do **not** use `inengine/modulus-*` — that prefix is reserved for Modulus **runtime extensions**.  
2. **Scaffold:** generate from **`InEngine/module-skeleton`** (customized Spatie package).  
3. **Provider:** extend Modulus `ModuleServiceProvider`; declare `modulusPackageName()`; boot only when Modulus says enabled; standalone full boot if Modulus is absent.  
4. **Discovery:** only `extra.laravel.providers` (and aliases) in `composer.json` — no parallel `extra.inengine` provider list.  
5. **Install vs enable:** never assume “Composer installed” means enabled; default disabled until `modulus:enable`.  
6. **Composer acquire/remove:** leave to the host or **`inengine/modulus-composer`** / marketplace — feature modules do not shell out to Composer to manage siblings.  
7. **UI for Modulus ops:** optional; if building operator screens for enable/disable/browse, prefer depending on **`inengine/modulus-ui`** patterns / APIs later — do not fork a second admin into each feature module.  
8. **Capabilities:** ship the skeleton’s capability descriptor stub (provides/requires, events, peers) so **`modulus-inspect`** can read metadata without booting the module.  
9. **Quality:** SemVer, Pint, Pest, PHPStan; CLI/API first.  
10. **Data safety:** migrations on enable/upgrade; disable/uninstall non-destructive by default.  
11. **Dependencies:** Composer `require` is the hard graph. Required Modulus modules must be enabled first; non-Modulus packages need only be installed (normal Laravel discovery). Optionally reach down from `activating()` for config/dist/migrations.  
12. **Integration:** Listen to Modulus lifecycle events; domain events via Laravel; contracts + DTOs bound in the container (`resources/INTEGRATION.md`).

---

## Data safety: disable / uninstall / migrate

| Operation | Data / DB |
|-----------|-----------|
| **Disable** | **Non-destructive.** Stop loading provider; leave tables, files, settings intact. |
| **Uninstall** (default) | **Non-destructive of data.** Remove package from Composer + Modulus state + discovery sync. Do **not** drop tables or wipe storage unless an explicit destructive flag/command is used later. |
| **Enable / install / upgrade** | Modules **must** ship migrations (and upgrade paths) for schema/structure. Run migrate (or module migrate hook) on enable/upgrade so DB changes are intentional and versioned. |

Activate hooks: schema/defaults/cache warm. Deactivate hooks: temp cleanup only. Uninstall hooks: optional explicit purge only when requested.

**Why:** Operators can disable safely to debug; data loss must be opt-in.

---

## Inter-module communication

1. **Event bus** — Laravel events for discovery notifications and “something happened” (`ModuleEnabled`, domain events). Loose coupling.  
2. **Contracts / interfaces** — stable PHP APIs modules implement or consume (`BillingGateway`, `CalendarReader`, …).  
3. **DTOs** — typed payloads across boundaries (no ad-hoc associative arrays as the long-term contract).

**Why:** Bus for fan-out/discovery; interfaces+DTOs for compile-time-safe data and substitutable implementations (OOP). Abilities/menus registries can wait.

---

## Quality bar: SemVer, Pint, Pest, PHPStan

Required for **Core**, **Modulus**, and **every feature module**:

| Tool | Expectation |
|------|-------------|
| **SemVer** | `composer.json` `version` / Git tags; Conventional Commits where releases are automated |
| **Pint** | Format on change / CI |
| **Pest** | Lifecycle and public API coverage; modules: CLI/API first |
| **PHPStan** | Level agreed per repo; CI gate |

Skeleton and CI templates enforce this so new modules don’t skip the bar.

---

## Core: require Modulus + early boot

- **`composer.json` require `inengine/modulus`** (not only `composer.local.json`). Local path `@dev` remains for development.  
- **Boot Modulus early** so `PackageManifest` decoration and state are available before feature module providers resolve (bootstrap / application provider order documented and tested).  
- Publish Modulus config; ensure state path writable.  
- No default `User`; no product features in Core.  
- Arch/smoke tests: Modulus present; feature namespaces not coupled into `App\`; stub module enable/disable path.

**Why:** Core without Modulus is not the InEngine host; late registration breaks discovery gating.

---

## Artisan surface (UI parity)

| Command | Purpose |
|---------|---------|
| `modulus:list` | Modules + enabled/disabled |
| `modulus:discover` | Rescan Composer providers; update state (atomic); sync discovery |
| `modulus:enable` | Enable (deps check, hooks, migrate as needed, state swap, discovery sync) |
| `modulus:disable` | Disable (non-destructive; hooks; state swap; discovery sync) |
| `modulus:uninstall` | Non-destructive uninstall by default; state + discovery + Composer remove |
| `modulus:status` | One module: Composer vs state vs discovery |
| **`modulus:doctor`** | Full diagnostics: drift, corrupt/missing state, PHP/Laravel constraints, dependency graph issues |

---

## Implementation plan (stepped)

Ordered work for Modulus, Core, and modules. Each step includes **why** and **considerations**.

### Step 0 — Finish Laravel 13 baseline

- Merge Modulus PR **#7** (L13 / PHP 8.5); tag a SemVer release.  
- **Why:** Hosts and CI need a stable constraint.  
- **Considerations:** Keep testbench matrix (L12+L13) until L12 is dropped deliberately.

### Step 1 — Split Modulus into collaborating services

- Introduce interfaces + classes: `ModulusStateStore`, repository/orchestrator, discovery syncer, base `ModuleServiceProvider`. Facade/CLI call orchestrator only.  
- **Why:** Avoid god object; testable seams.  
- **Considerations:** Keep BC thin while scaffold exists; don’t preserve broken `modules.php` rewrite as long-term API.

### Step 1a — Close test gaps for Steps 1–3 seams

- Pest for `PackageManifestInstaller` early rebind, `FilteringPackageDiscoverySyncer::sync()`, repository enable/disable → filtered `providers()`, deny-list `forget`, and `register` version updates.  
- **Why:** Catch regressions across store, filter, and orchestrator before Step 4 handshake polish.  
- **Considerations:** Full Core boot smoke stays Step 5; Artisan command coverage stays Step 9.

### Step 2 — `ModulusStateStore` (atomic file)

- Implement JSON (or PHP) state file with **temp write + atomic swap**.  
- Config path; default disabled on first see; `isEnabled()`.  
- **Why:** Durable enabled set without DB; nWidart-proven pattern.  
- **Considerations:** Concurrent Artisan; file permissions on deploy; commit vs gitignore policy.

### Step 3 — Laravel `PackageManifest` sync (enable/disable)

- Decorate/filter PackageManifest using state store; refresh packages cache on toggle.  
- **No** host `composer.json` churn for toggles.  
- Config `discovery.scope`: `modules` (default) or `all` (optional full-package gating + denylist).  
- **Why:** Enable/disable through Laravel’s discovery facilities.  
- **Considerations:** Race with `composer dump-autoload` / `package:discover`; doctor must detect drift; default filters only Modulus modules; never disable Modulus itself.

### Step 4 — Register-with-Modulus handshake (defense in depth)

- Base provider: detect Modulus → register → **do not boot** unless Modulus says enabled; if Modulus absent, standalone full boot.  
- **Why:** Required second gate if discovery cache and state diverge.  
- **Considerations:** Provider may not load at all when filtered out — discovery scan must not depend on executing disabled providers (scan Composer metadata / class hierarchy carefully).

### Step 4b — `InEngine/module-skeleton` (was Step 12; pulled forward)

- Separate **GitHub template** repository (Spatie `package-skeleton-laravel` model) — **not** a Packagist package; never `composer require` the skeleton itself.  
- InEngine / Modulus defaults, extends `ModuleServiceProvider` (Step 4 handshake), capability descriptor stub, Pint/Pest/PHPStan/CI, SemVer notes, non-destructive lifecycle docs.  
- Linked as a PHPStorm content root beside other InEngine modules.  
- **Why:** Every Modulus-compatible module starts with the correct contract instead of hand-rolling after the handshake lands.  
- **Considerations:** Generated modules depend on published or path `@dev` `inengine/modulus`; keep feature package name `inengine/{feature}`, not `modulus-*`.

### Step 5 — Core requires Modulus + early boot

- Add `inengine/modulus` to Core `require`; wire early bootstrap; publish config; Pest/arch smoke.  
- **Why:** InEngine host contract.  
- **Considerations:** Herd local `@dev` via merge plugin; CI uses tagged versions when possible.

### Step 6 — Lifecycle hooks + non-destructive rules + migrations

- Activate / deactivate / uninstall hooks; enable runs migrations/upgrades; disable/uninstall do not destroy data by default.  
- **Why:** Safe ops; schema changes are versioned.  
- **Considerations:** Explicit future `modulus:purge` (or `--wipe`) only with confirmation; document module author rules in skeleton.

### Step 7 — Module dependencies

- Cannot enable without required **Modulus modules** enabled; cannot disable/uninstall while enabled dependents still require the package.  
- Hard graph = each package’s Composer `require` (skip platform `php` / `ext-*` / `composer-*`). Non-Modulus requires need only be **installed**; Modulus does not publish/migrate them.  
- Plain Composer deps keep **normal Laravel discovery / `vendor:publish`**. Feature modules may **optionally** reach down in `activating()` / `bootstrapNonModulusDependencies()` to also publish **config**, copy UI **`resources/dist` → `public/vendor/{name}`**, or load migrations (skeleton helpers + `DEPENDENCIES.md`).  
- **Why:** WordPress 6.5 lesson; prevent half-broken apps; allow real Composer libs alongside modules.  
- **Considerations:** No auto-enable cascade; clear exceptions (`UnmetModuleDependencyException`, `ModuleHasDependentsException`); Artisan messaging in Step 9.

### Step 8 — Event bus + contracts + DTOs

- Dispatch module lifecycle events (`ModuleEnabled`, `ModuleDisabled`, `ModuleForgotten`, `ModuleUpgraded`, `ModuleRegistered`, `ModulePublished`, `ModuleSeeded`) with `ModuleStateData` where useful.  
- Document domain-event pattern (normal Laravel `Event` / listeners).  
- Publish/consume **interfaces** and **DTOs** via container bindings (skeleton `src/Contracts` + `src/DataTransferObjects`; Modulus `INTEGRATION.md`). Shared contracts package only if later needed.  
- **Why:** Loose coupling + type-safe integration.  
- **Considerations:** Don’t invent a second plugin API; prefer Laravel container bindings to interfaces; owning-module hooks vs peer events.

### Step 8b — Laravel Boost agent guidelines (Core, Modulus, skeleton)

- Ship Boost-discoverable package guidelines/skills from Modulus (`resources/boost/guidelines`, `resources/boost/skills`) so Core `boost:install` / `boost:update` loads them.  
- Skeleton ships the same for generated feature modules + repo `AGENTS.md` / `.ai/guidelines`.  
- Core: `.ai/guidelines/inengine-core.md`, `.ai/skills/modulus-modules`, `.ai/rules` (path-scoped host rules), `boost.json` includes `inengine/modulus`, `post-update-cmd` runs `boost:update`.  
- **Why:** Cursor / Claude / other agents follow install≠enable, naming, deps, and integration without relying only on Cursor `.mdc` or scattered markdown.  
- **Considerations:** Keep human docs (`DESIGN.md`, `TEMPLATE.md`, Documentation plan) in sync with Boost guidelines; prefer PHPDoc over inline comments per Boost foundation rules.

### Step 9 — Full Artisan surface + `modulus:doctor`

- Implement list/discover/enable/disable/uninstall/status/**doctor**; every UI action maps here.  
- Doctor: state file health, Composer installed set, PackageManifest/cache sync, SemVer/constraint mismatches, dependency violations.  
- **Missing modules (planned remediation):** Doctor must **list** every package tracked in Modulus state that is not present in Composer’s installed set (enabled and disabled), building on runtime `MissingModuleException` / `validateTrackedPackages()`.  
  - **Repair options (interactive CLI; later UI):**  
    1. **Install** — ask `modulus-composer` / Composer to `require` the missing package (then discover → disabled → optional enable).  
    2. **Supply path/VCS** — operator points at a local path, Git URL, or private source; install then re-validate.  
    3. **Substitute** — map the missing package to another installed (or newly installed) module that fulfills the same role; update Modulus state (forget missing / register+enable substitute) and document the replacement for deps/inspect.  
  - Non-interactive / CI: doctor reports missing packages + suggested actions; exit non-zero when any **enabled** package is missing; `--fix` only with explicit flags (no silent Composer writes).  
- **Why:** Operator trust; UI parity; debug discovery drift; recover from removed vendors without hand-editing state.  
- **Considerations:** Doctor exit codes for CI; machine-readable `--json` later; Composer I/O stays in `modulus-composer`, not Modulus core; substitution must respect Step 7 dependency rules.

### Step 10 — `inengine/modulus-composer` (+ marketplace / UI later)

- **`modulus-composer`:** Composer require/remove/update client; Packagist / private / Git; optional Composer hooks that call into Modulus after acquire/remove.  
- **`modulus-marketplace`:** browse/search UX over known sources; depends on composer client for install.  
- **`modulus-ui`:** optional operator UI (separate; after CLI solid).  
- **Why:** Install/Composer I/O and UI stay out of runtime core.  
- **Considerations:** Auth for private Packagist; timeouts; non-interactive CI; always discover→disabled→explicit enable; never put Composer hooks in `inengine/modulus`.

### Step 11 — Quality gates everywhere

- Enforce SemVer + Pint + Pest + PHPStan on Core, Modulus, and module CI; document in skeleton.  
- **Why:** Platform reliability and release discipline.  
- **Considerations:** Shared workflow templates; same PHP 8.5 floor.

### Step 12 — (relocated)

See **Step 4b** — `InEngine/module-skeleton` was pulled forward immediately after the handshake.

### Step 13 — Pilot + product hosts

- Convert one module (e.g. Alert) **from the skeleton**; verify on Core Herd.  
- Provost “Require Modulus” and copy Core early-boot pattern.  
- **Why:** Prove the spine before packs/CRM/LMS.  
- **Considerations:** Migrate existing `extra.laravel.providers` packages onto base provider without breaking standalone use.

### Step 14 — `inengine/modulus-inspect` (capability catalog)

A future **addon** (`inengine/modulus-inspect`) that interrogates Modulus-compatible modules more deeply than Composer “suggest”:

| Question (CLI / API / later UI) | Answer source |
|---------------------------------|---------------|
| What can this module do? | Declared provides: contracts, abilities, Artisan commands, events |
| What should connect to it? | Declared requires / recommends / optional peers (other modules or contracts) |
| What sub-modules or related packages belong with it? | Declared related packages / sub-features |
| How does a custom module integrate? | Documented bind points: interfaces to implement, events to listen/dispatch, container bindings, example Artisan/`modulus:*` flows |

Behavior sketch:

1. Modules from the skeleton expose a **capability descriptor** (PHP class and/or committed manifest generated/validated in CI).  
2. Catalog extension reads descriptors from **installed** (and optionally Packagist-listed) Modulus modules without requiring them to be enabled.  
3. Surfaces `modulus:inspect {module}`, `modulus:compat {module}`, peer suggestions, and machine-readable `--json` for UI.  
4. May collaborate with `modulus-marketplace` / `modulus-composer` for “install recommended peer,” but **does not own** enable/disable or Composer require (those stay Modulus runtime + `modulus-composer`).

**Why:** Operators and module authors need WordPress-like “what plugs into what,” with typed PHP contracts—not only Packagist suggest strings.  
**Considerations:** Descriptors must be readable when the module is disabled (do not depend on booting the module provider); version capability schemas; keep this out of Modulus core so the runtime stays lean.

---

## Non-goals (this spine)

- Plugins / marketplace / Composer-client / inspect / operator UI inside Modulus **core** (`modulus-*` extensions only)  
- Marketplace, Composer hooks, or inspect UI before CLI + discovery sync + doctor are solid  
- Composer require/remove or Composer event hooks inside `inengine/modulus` core  
- Database-backed module registry  
- Destructive uninstall as default  
- Use-case “packs” as first deliverable  
- Users/auth inside Core  
- Non-Composer marketplace install  
- Rewriting host `composer.json` `dont-discover` on every enable/disable  
- Gating every Composer package by default (`discovery.scope=all` is opt-in only)  
- Ad-hoc Modulus modules that skip the skeleton (discouraged; no guarantee of inspect/compat)  

---

## Related paths

| Path | Notes |
|------|--------|
| `InEngine/Core` | Host — https://github.com/InEngine/Core |
| `InEngine/Modules/Modulus` | Runtime — https://github.com/InEngine/Modulus (`DESIGN.md` mirrors this) |
| `InEngine/Documentation` | This file, logos, premise |
| `InEngine/Modules/*` | Feature modules (from module-skeleton) |
| `InEngine/module-skeleton` | **Required** GitHub template (Spatie-fork; not Packagist) — https://github.com/InEngine/module-skeleton |
| `inengine/modulus-composer` | Planned — Composer require/remove + optional Composer hooks |
| `inengine/modulus-marketplace` | Planned — browse/search; uses composer client for install |
| `inengine/modulus-ui` | Planned — operator UI over Modulus + extension APIs |
| `inengine/modulus-inspect` | Planned — capability inspect / compat / integrate docs |

---

## Changelog of design decisions

### 2026-11-03

- Optional **`discovery.scope=all`** so Modulus can gate non-module Laravel packages too; default remains **modules-only**.  
- Modulus **extensions** use `inengine/modulus-{role}` names: `modulus-composer`, `modulus-marketplace`, `modulus-ui`, `modulus-inspect`.  
- **Composer require/remove and Composer hooks** belong in **`modulus-composer`**, not core.  
- Modulus modules **must** refuse to boot unless Modulus reports enabled (handshake + PackageManifest).  
- **All Modulus-compatible modules** should come from **`InEngine/module-skeleton`**.  
- **`modulus-inspect`**: capability catalog (provides/requires, events, peers, integrate guidance).  

### 2026-09-08

- File registry via **`ModulusStateStore`** (atomic swap); not DB.  
- Enable/disable through **Laravel PackageManifest / discovery**, not Composer.json fights.  
- **Core requires Modulus** and boots it **early**.  
- **SemVer + Pint + Pest + PHPStan** for Core, Modulus, modules.  
- Inter-module: **bus + contracts/DTOs**.  
- **Non-destructive** disable/uninstall; migrations on install/enable/upgrade.  
- **`modulus:doctor`** required.  
- Modulus implemented as **multiple classes/services**, not one class.
