# Scaffolded modules (no prod code yet)

Created from Skeleton `v0.1.0-beta.1` under `Modules/`. Handshake + tooling only.

## Identity / auth

| Dir | Package | Notes |
|-----|---------|-------|
| `User` | `inengine/user` | Hub (existing) |
| `LocalUser` | `inengine/local-user` | Auth companion — `auth.method:local` |
| `GoogleUser` | `inengine/google-user` | Auth companion — `auth.method:google` |
| `GithubUser` | `inengine/github-user` | Auth companion — `auth.method:github` |
| `Person` | `inengine/person` | Optional demographics |

## Modulus extensions

| Dir | Package | Notes |
|-----|---------|-------|
| `ModulusConfig` | `inengine/modulus-config` | Read/write module config |
| `ModulusConfigUI` | `inengine/modulus-config-ui` | Operator UI (label: Modulus Config) |

## Theme

| Dir | Package | Notes |
|-----|---------|-------|
| `Theme` | `inengine/theme` | Design tokens / colors for UI modules |

Private GitHub repos (default branch `dev`); Packagist not published yet.

| Package | Repo |
|---------|------|
| `inengine/user` | https://github.com/InEngine/User *(existing)* |
| `inengine/local-user` | https://github.com/InEngine/LocalUser |
| `inengine/google-user` | https://github.com/InEngine/GoogleUser |
| `inengine/github-user` | https://github.com/InEngine/GithubUser |
| `inengine/person` | https://github.com/InEngine/Person |
| `inengine/modulus-config` | https://github.com/InEngine/Modulus-Config |
| `inengine/modulus-config-ui` | https://github.com/InEngine/Modulus-Config-UI |
| `inengine/theme` | https://github.com/InEngine/Theme |
