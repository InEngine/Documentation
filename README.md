![InEngine Logo](logo/InEngine-Logo 800X496.png)

# InEngine documentation

| Doc | Purpose |
|-----|---------|
| [Plan and Design.md](Plan%20and%20Design.md) | Platform architecture, Modulus, modules; **Step 10 Composer done** |
| [Distribution and Installation.md](Distribution%20and%20Installation.md) | `create-project` vs `composer require`; Core, modules, products; **licensing split** |
| [Packagist.md](Packagist.md) | Packagist checklist (**hold** until intentional release); Skeleton = template only |
| [Git workflow.md](Git%20workflow.md) | **`dev` → `main`/`master` PR** standard for all InEngine repos |
| [Identity and Auth.md](Identity%20and%20Auth.md) | User hub, social companions, Person, registry / optional modules |
| [Module naming and UI.md](Module%20naming%20and%20UI.md) | UI display labels vs Composer package names; CLI shortcuts |
| [ci/](ci/) | Step 11 workflow templates (Pest / PHPStan / Pint) — not UI |

**Licensing (2026-11-19):** Core / Modulus / Modulus Composer / Skeleton = **MIT**. `modulus-ui` = **private Pro**.

**Core:** opinionated Laravel-based modular application builder framework — built on Laravel; requires Modulus; Modulus Composer opt-in. **Skeleton:** public GitHub template (not Packagist), attributed to Spatie `package-skeleton-laravel`.

---

# The Premise Behind InEngine
## Managing Context and Content
You have a website, you use a content management system of some sort... but...





It is time to manage both the Context and Content of our Websites.
Your content is the reason you have a website, but content without context it looses meaning, emphases, and specificity. On the web the context for your content is what makes up the rest of your website. It is the layout and design of the site from images to typography. It is the data relationship between pieces of content. It is the features supplied, such as search, indexes, commenting. If your content needs context then why do we only have content management systems? Why do we not have website management systems. A system that manages both the content and the context. A system that makes it easy to have a consistent context for your content regardless of the viewing context (mobile, app, desktop). A system that integrates well with the platforms and networks you use that makes it easy to natively share your content with the appropriate context for the given platform or network. This idea of managing content and context is why we are building InEngine.

## Features
- flexible with pluggable architecture that allows you to have only what you need but always what you need
- easy for developers by using best practices, oss packages, and standards, and the latest versions of php and js at your finger tips
- extensive documentation
- code review and testing for all contributions
- integration with social media platforms and networks
- oath integration for users
- multi-site management from a single admin interface
- easy management for non-technical users
- full site management from design to content
- open source for self installations, competitively priced for hosted managed solution
