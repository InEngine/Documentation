# Scaffolded modules

Created from Skeleton under `Modules/`. Private GitHub repos; default branch `dev`. Packagist not published yet.

## Contracts (Modulus modules — ports / tokens only)

| Dir | Package | Status |
|-----|---------|--------|
| `IdentityContracts` | `inengine/identity-contracts` | Modulus module; `DisplayNameResolver`, `PersonResolver`, `IdentityCapability` |
| `AuthContracts` | `inengine/auth-contracts` | Modulus module; `AuthMethod`, `LocalCredentialsContract`, `AuthMethodList` |

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
| `inengine/theme` | https://github.com/InEngine/Theme |
