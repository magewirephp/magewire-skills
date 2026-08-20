---
name: magewire-backwards-compatibility
description: "Reference for Magewire's framework-level backwards compatibility system. Use when migrating Magewire v1 (Livewire v2) code to v3, enabling BC on components, understanding wire:model / entangle / hook behavioral changes between v2 and v3, or working with the BC memo flag, deprecated v1 component APIs, and the #[HandleBackwardsCompatibility] attribute. Theme-agnostic — BC applies to any Magewire component in any Magento theme. For theme-specific BC JS implementations (e.g. Hyvä Checkout), see the matching theme's BC skill."
license: MIT
metadata:
  author: Willem Poortman
---

# Magewire Backwards Compatibility

Magewire v3 is built on Livewire v3, which introduced breaking changes from v2. The BC system lets existing v1 components keep working without code changes while providing a migration path to v3 conventions.

Magewire is **not tied to any specific theme**. The BC system is a framework concern — any component, in any theme, can opt into BC. Individual themes may ship their own JS-layer BC implementations on top of this core (see the theme's BC skill for details).

---

## The Two BC Layers

### 1. Core BC (`lib/MagewireBc/`) — theme-agnostic

Framework-level compatibility that applies to all Magewire components regardless of theme.

**PHP trait** — `HandlesComponentBackwardsCompatibility` is mixed into the base `Component` class. It provides deprecated v1 APIs: `$this->id` (public), `getPublicProperties()`, and concerns for Emit, Error, BrowserEvent, and Request. The `Component\Form` base class uses this trait via `HandlesFormComponentBackwardsCompatibility` (a deprecated pass-through that exists for potential form-specific BC, currently empty).

**Feature** — `SupportMagewireBackwardsCompatibility` (sort order 99100):
- Pushes BC effects into the snapshot during `dehydrate()`: property path mappings (`data` → `$wire`, `__livewire` → `queuedUpdates`) and preferences
- These effects are read by JS to proxy old property access patterns

### 2. Theme-specific BC layers

Some themes need additional BC on top of core—wire directive rewriting, entangle live-by-default, and deprecated JS hook/event re-triggering. Those live in standalone companion packages. The pattern is:

- A theme-scoped Feature manages a `memo.bc.enabled` flag on the snapshot
- JS templates read the flag and apply migrations to components that have it set
- The flag is resolvable per-component via the `#[HandleBackwardsCompatibility]` attribute, data store, or theme-defined layout rules

---

## The `memo.bc.enabled` Flag

The BC flag is the single signal a theme's BC JS uses to decide whether to apply v2→v3 shims to a component. Resolution priority:

1. `#[HandleBackwardsCompatibility]` attribute on the component class
2. Previously hydrated value from the component's data store
3. Theme-specific defaults (e.g. "all components under layout container X")

On `hydrate()`, the flag is restored from memo into the component's data store. On `dehydrate()`, it is pushed back into the snapshot memo.

---

## Wire Directive Changes (v2 → v3)

| Livewire v2 | Livewire v3 | Behavior |
|---|---|---|
| `wire:model` | `wire:model.live` | Syncs on every input change (instant) |
| `wire:model.defer` | `wire:model` | Syncs on form submit / next request (now the default) |
| `wire:model.lazy` | `wire:model.blur` | Syncs when the field loses focus |

### Migrating your own code

Remove the old modifiers and use v3 syntax:

```html
<!-- Before (v1/v2) -->
<input wire:model="name">              <!-- instant sync -->
<input wire:model.defer="email">       <!-- deferred -->
<input wire:model.lazy="phone">        <!-- on blur -->

<!-- After (v3) -->
<input wire:model.live="name">         <!-- instant sync (opt-in) -->
<input wire:model="email">             <!-- deferred (now default) -->
<input wire:model.blur="phone">        <!-- on blur -->
```

A theme's BC JS may rewrite these directives automatically for components with `memo.bc.enabled` — see that theme's BC skill.

---

## Entangle Changes (v2 → v3)

| Livewire v2 | Livewire v3 |
|---|---|
| `$wire.entangle('prop')` — **live** by default | `$wire.entangle('prop')` — **deferred** by default |
| N/A | `$wire.entangle('prop').live` — opt-in to live |

### Migrating your own code

If you want instant sync (v2 default), add `.live` explicitly:

```javascript
// Before (v1/v2) — live by default
couponCode: this.$wire.entangle('couponCode'),

// After (v3) — must opt in to live
couponCode: this.$wire.entangle('couponCode').live,

// Or keep deferred (v3 default) — no .live needed
couponCode: this.$wire.entangle('couponCode'),
```

A theme's BC JS may restore the v2 live-by-default semantic for BC-enabled components via a `$wire` proxy — see that theme's BC skill.

---

## Hook / Event Changes (v2 → v3)

| Deprecated (v2) | Replacement (v3) |
|---|---|
| `component.initialized` | `component.init` |
| `element.initialized` | `element.init` |
| `element.updating` | `morph.updating` |
| `element.removed` | `morph.removed` |
| `message.sent` | `commit` |
| `message.failed` | `commit` → `fail()` |
| `message.received` | `commit` → `succeed()` |
| `message.processed` | `commit` → `succeed()` → `queueMicrotask` |

### Component property aliases

Old property names aliased on the component object by BC JS layers:

```javascript
// component.deferredActions → component.queuedUpdates
// component.data → component.$wire
```

---

## Enabling BC for a Component

### Explicit via PHP attribute

```php
use Magewirephp\Magewire\Features\SupportMagewireBackwardsCompatibility\HandleBackwardsCompatibility;

#[HandleBackwardsCompatibility]
class MyLegacyComponent extends Component
{
    // This component gets BC behavior
}

#[HandleBackwardsCompatibility(enabled: false)]
class MyModernComponent extends Component
{
    // Explicitly opt out, even if a theme would normally enable it
}
```

### Via data store (programmatic)

```php
use function Magewirephp\Magewire\store;

// In a Feature or hook:
store($component)->set('magewire:bc', true);
```

### Via theme defaults

A theme can attempt to BC-enable components automatically based on its own rules. For Magewire 3.5 with `magewirephp/magewire-hyva-checkout`, do not rely on the `hyva-checkout-main` fallback: the core resolver writes an explicit false before the fallback checks for an unset value. Use the attribute explicitly and verify the snapshot.

---

## Key Files

| File | Purpose |
|------|---------|
| `lib/MagewireBc/Features/SupportMagewireBackwardsCompatibility/SupportMagewireBackwardsCompatibility.php` | Core BC feature — pushes path mappings into effects |
| `lib/MagewireBc/Features/SupportMagewireBackwardsCompatibility/HandleBackwardsCompatibility.php` | PHP attribute for explicit BC opt-in/out |
| `lib/MagewireBc/Features/SupportMagewireBackwardsCompatibility/HandlesComponentBackwardsCompatibility.php` | Trait with deprecated v1 component APIs |

---

## Migration Checklist

When upgrading a v1 component to v3:

1. **wire:model** — replace `wire:model` with `wire:model.live` if you need instant sync, remove `.defer` (it's now the default), replace `.lazy` with `.blur`
2. **entangle** — add `.live` if you need instant sync, otherwise leave as-is (deferred is now default)
3. **JS hooks** — replace deprecated event names (see table above)
4. **Component properties** — replace `component.data` with `component.$wire`, `component.deferredActions` with `component.queuedUpdates`
5. **PHP APIs** — replace `$this->getPublicProperties()` with `$this->all()`, avoid public `$this->id` (use `$this->id()` or `$this->getId()`)
6. **Remove BC attribute** — once migrated, remove `#[HandleBackwardsCompatibility]` if present
