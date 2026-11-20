# Identity and auth (InEngine)

Design notes for composing **User**, optional **social companions** (Google, GitHub, …), optional **Person**, and Modulus capability discovery.

**Canonical copy for implementation:** [InEngine/User `docs/IDENTITY_ARCHITECTURE.md`](https://github.com/InEngine/User/blob/dev/docs/IDENTITY_ARCHITECTURE.md) (also under `Modules/User/docs/` in the monorepo).

## One-line summary

**One User hub**; Google/GitHub as **optional side modules** that link into User; **Person** optional demographics; **Registry in Modulus** for soft discovery; **contracts + default bindings** so apps work when modules are omitted.

## See also

- [Plan and Design.md](Plan%20and%20Design.md) — Step 14 Capability Registry  
- [Git workflow.md](Git%20workflow.md)  
- Composer [`provide`](https://getcomposer.org/doc/04-schema.md#provide)  
- Symfony [service tags](https://symfony.com/doc/current/service_container/tags.html)  
