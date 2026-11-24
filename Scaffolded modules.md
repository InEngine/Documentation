# Scaffolded modules

Created from Skeleton under `Modules/`. Private GitHub repos; default branch `dev`. Packagist not published yet.

## Contracts (Modulus library modules — ports / tokens only)

Composer `"type": "library"` + Modulus capability `kind: library` (always boot when installed; contribute to registry without enable). User hard-requires both.

| Dir | Package | Status |
|-----|---------|--------|
| `IdentityContracts` | `inengine/identity-contracts` | Library; `DisplayNameResolver`, `PersonResolver`, `UserType`, `EntityMorph` |
| `AuthContracts` | `inengine/auth-contracts` | Library; `AuthMethod`, `LocalCredentialsContract`, `AuthMethodList` |

## Identity / auth

| Dir | Package | Notes |
|-----|---------|-------|
| `User` | `inengine/user` | UUID hub — `auth_methods` JSON; loose Authenticatable |
| `AuthLocal` | `inengine/auth-local` | Local credentials + `LocalCredentialsContract` bind |
| `AuthGoogle` | `inengine/auth-google` | Scaffold — `auth.method:google` |
| `AuthGithub` | `inengine/auth-github` | Scaffold — `auth.method:github` |
| `Person` | `inengine/person` | Next — optional demographics |

## Modulus extensions

| Dir | Package | Notes |
|-----|---------|-------|
| `ModulusConfig` | `inengine/modulus-config` | Read/write module config |
| `ModulusConfigUI` | `inengine/modulus-config-ui` | Operator UI (label: Modulus Config) |
| `ModulusUI` | `inengine/modulus-ui` | Operator UI (label: **Modulus**); private Pro; scaffold from Skeleton — UI implementation next |

## Theme

| Dir | Package | Notes |
|-----|---------|-------|
| `Theme` | `inengine/theme` | Design tokens / colors for UI modules |

| Package | Repo |
|---------|------|
| `inengine/identity-contracts` | https://github.com/InEngine/Identity-Contracts |
| `inengine/auth-contracts` | https://github.com/InEngine/Auth-Contracts |
| `inengine/user` | https://github.com/InEngine/User |
| `inengine/auth-local` | https://github.com/InEngine/Auth-Local |
| `inengine/auth-google` | https://github.com/InEngine/Auth-Google |
| `inengine/auth-github` | https://github.com/InEngine/Auth-Github |
| `inengine/person` | https://github.com/InEngine/Person |
| `inengine/modulus-config` | https://github.com/InEngine/Modulus-Config |
| `inengine/modulus-config-ui` | https://github.com/InEngine/Modulus-Config-UI |
| `inengine/modulus-ui` | https://github.com/InEngine/Modulus-UI |
| `inengine/theme` | https://github.com/InEngine/Theme |
