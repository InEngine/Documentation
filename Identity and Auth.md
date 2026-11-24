# Identity and auth (InEngine)

Design notes for composing **User**, optional **auth companions** (Local, Google, GitHub, …), optional **Person**, and Modulus capability discovery.

**Canonical copy for implementation:** [InEngine/User `IDENTITY_ARCHITECTURE.md`](https://github.com/InEngine/User/blob/dev/IDENTITY_ARCHITECTURE.md) (also under `Modules/User/IDENTITY_ARCHITECTURE.md`).

## One-line summary

**One User hub**; optional **auth** companions (`auth-local` / `auth-google` / `auth-github`) that link into User; **Person** optional demographics; **Registry in Modulus** for soft discovery; **`users.auth_methods` JSON** allow-list of short method ids; **contracts + default bindings** so apps work when modules are omitted.

## Package split

| Package | Role |
|---------|------|
| `inengine/identity-contracts` | Modulus **library** — ports/tokens (`DisplayNameResolver`, `UserType`, `EntityMorph`, …); Composer-required by User |
| `inengine/auth-contracts` | Modulus **library** — auth ports/tokens (`AuthMethod`, `LocalCredentialsContract`, …); Composer-required by User |
| `inengine/user` | UUID hub; `auth_methods` / `types` JSON via `EmbeddedShortIdRelation`, entity morph, `enabled`, soft deletes |

Companions extend User with `Model::resolveRelationUsing()` / `User::macro()` via `UserExtensionGate` — only when Modulus registry has `identity.user` + `identity.user.config`, User is enabled, and `config('user.model')` resolves. Never traits that patch User source. See User `IDENTITY_ARCHITECTURE.md`.
| `inengine/auth-local` | `auth_local_credentials` by `user_id`; implements `LocalCredentialsContract` |
| `inengine/auth-google` / `auth-github` | OAuth companions (scaffold) |
| `inengine/person` | Optional demographics (next) |

Login UI lives in future `auth-*-ui` packages — not in companions.

## Scaffold / implementation status

| Area | Status |
|------|--------|
| IdentityContracts + AuthContracts | Library modules implemented (private GitHub; path-only) |
| User (UUID, EmbeddedShortId relations, gate) | Foundation implemented + CI green |
| Auth-Local (credentials + UserExtensionGate) | Foundation implemented + CI green |
| Modulus library kind / ModuleExtensionGate | Implemented (libraries always-on like Modulus core) |
| Modulus-UI | Scaffold from Skeleton; wired in Core + PhpStorm; CI green — UI next |
| Auth-Google / Auth-Github | Scaffold only |
| Person | Scaffold only — **next** for identity stack |

Session handoff: User `.cursor/SESSION_HANDOFF.md`. See [Scaffolded modules.md](Scaffolded%20modules.md).

## See also

- [Plan and Design.md](Plan%20and%20Design.md) — Step 14 Capability Registry  
- [Git workflow.md](Git%20workflow.md)  
- Composer [`provide`](https://getcomposer.org/doc/04-schema.md#provide)  
- Symfony [service tags](https://symfony.com/doc/current/service_container/tags.html)  
