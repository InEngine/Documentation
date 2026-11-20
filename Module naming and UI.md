# Module naming: UI display vs Composer packages

Notes so operators are not forced to think in “backwards” stack names (e.g. installing “UI” that then needs “Modulus”, “Modulus Core”, “Modulus Composer” as separate mental products).

## Problem to avoid

If the operator UI is labeled **Modulus UI** and the runtime is also **Modulus**, people install or toggle “UI” and still hunt for “Modulus”, or the product reads as Modulus-UI → Modulus-Core → Modulus-Composer from the outside in.

## Convention

| Layer | What the human sees | What Composer / CLI use |
|-------|---------------------|-------------------------|
| **Operator UI label** | Product / feature name (e.g. **Modulus**) | — |
| **Composer package** | — | `inengine/modulus-ui`, `inengine/modulus`, `inengine/modulus-composer` |
| **Modulus state key** | — | Always the Composer package name (`inengine/modulus-ui`) |

### UI modules declare a display `Module` name

In the UI module’s provider / capability descriptor (and later registry metadata), set a **display name** (and optional short description) used by operator UI lists and toggles:

- Package: `inengine/modulus-ui`  
- **Display module name:** `Modulus`  
- Soft/hard depends on: `inengine/modulus` (and typically `inengine/modulus-composer` for install flows)

Enabling **Modulus** in the UI means enabling the **UI package** (and pulling/enabling recommended deps). Operators are not asked to “enable Modulus UI” as a second product next to Modulus.

Same pattern for other UI skins later (e.g. a CRM UI package might display as **CRM** while the package is `inengine/crm-ui`).

### Dependencies

UI packages **require** the runtime they surface. Installing/enabling the UI module is enough for the operator story; Composer + Modulus dependency guards pull `inengine/modulus` (and friends) automatically. Do not make humans install “core” then “UI” as two unrelated catalog entries when one product surface is intended.

### CLI stays package-accurate

Artisan and docs aimed at developers keep **Composer package names**:

```bash
php artisan modulus:enable inengine/modulus-ui
php artisan modulus-composer:require inengine/modulus-ui
```

Optional **shortcuts / recommends** (composer or modulus-composer):

- `modulus-composer:require modulus` → resolves to recommended stack (e.g. `modulus-ui` + ensures `modulus` / `modulus-composer`)  
- Or `modulus:enable Modulus` as a **display-name alias** that maps to `inengine/modulus-ui` in UI-capable hosts only  

CLI primary path = package names; aliases are sugar for recommended sub-modules, not a second naming system in state files.

## Registry / descriptor fields (planned)

When Capability Registry lands in Modulus, persist something like:

```text
package:     inengine/modulus-ui
label:       Modulus          # UI display name
summary:     Operator UI for modules…
provides:    […]
requires:    [inengine/modulus, …]
recommends:  [inengine/modulus-composer, …]
```

Lists in **modulus-ui** show `label`. Doctor, enable, and Composer keep `package`.

## Feature modules (non-UI)

Feature packages (`inengine/user`, `inengine/alert`) usually use a label close to the feature (`User`, `Alert`). The special case is **runtime extension UIs** (`modulus-ui`, future `*-ui`) whose label is the **product** they operate, not the suffix “UI”.

## Related

- Extensions table: Documentation [Plan and Design.md](Plan%20and%20Design.md), Modulus `DESIGN.md`  
- Private Pro UI: [InEngine/Modulus-UI](https://github.com/InEngine/Modulus-UI)  
