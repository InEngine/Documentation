# Identity and auth (InEngine)

Design notes for composing **User**, optional **auth companions** (Local, Google, GitHub, …), optional **Person**, and Modulus capability discovery.

**Canonical copy for implementation:** [InEngine/User `IDENTITY_ARCHITECTURE.md`](https://github.com/InEngine/User/blob/dev/IDENTITY_ARCHITECTURE.md) (also under `Modules/User/IDENTITY_ARCHITECTURE.md`).

## One-line summary

**One User hub**; optional **auth** companions (`auth-local` / `auth-google` / `auth-github`) that link into User; **Person** optional demographics; **Registry in Modulus** for soft discovery; **`users.auth_methods` JSON** allow-list of short method ids; **contracts + default bindings** so apps work when modules are omitted.

## Package split

| Package | Role |
|---------|------|
| `inengine/identity-contracts` | Modulus module — ports/tokens for User↔Person (`DisplayNameResolver`, `PersonResolver`, `IdentityCapability`) — no migrations |
| `inengine/auth-contracts` | Modulus module — auth-method ports/tokens (`AuthMethod`, `LocalCredentialsContract`, `AuthMethodList`) |
| `inengine/user` | UUID authenticatable hub; owns `users` + `auth_methods`; binds Authenticatable loosely |
| `inengine/auth-local` | `auth_local_credentials` by `user_id`; implements `LocalCredentialsContract` |
| `inengine/auth-google` / `auth-github` | OAuth companions (scaffold) |
| `inengine/person` | Optional demographics (next) |

Login UI lives in future `auth-*-ui` packages — not in companions.

## Scaffold / implementation status

| Area | Status |
|------|--------|
| IdentityContracts + AuthContracts | Implemented + GitHub |
| User (UUID, helpers, Authenticatable) | Foundation implemented |
| Auth-Local (credentials + contract bind) | Foundation implemented |
| Auth-Google / Auth-Github | Scaffold only |
| Person | Scaffold only — next |

See [Scaffolded modules.md](Scaffolded%20modules.md).

## See also

- [Plan and Design.md](Plan%20and%20Design.md) — Step 14 Capability Registry  
- [Git workflow.md](Git%20workflow.md)  
- Composer [`provide`](https://getcomposer.org/doc/04-schema.md#provide)  
- Symfony [service tags](https://symfony.com/doc/current/service_container/tags.html)  
