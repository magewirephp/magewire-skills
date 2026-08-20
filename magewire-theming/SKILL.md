---
name: magewire-theming
description: "Build, extend, or debug a theme-specific Magewire 3 integration for Magento: standalone compatibility modules, layout nodes, Alpine loading, CSP-safe feature bridges, Hyvä Tailwind sources, and admin integration boundaries. Use for Hyvä, Hyvä Checkout, custom storefront themes, or magewire-admin."
license: MIT
metadata:
  author: Willem Poortman
---

# Magewire Theming

When installed, consult the sibling `magewire`, `magewire-architecture`, `magewire-javascript`, and `magewire-best-practices` skills when the task crosses into those areas.

Magewire core is theme-agnostic. Since 3.2, maintained integrations are standalone Composer packages, not folders below the core repository:

- `magewirephp/magewire-hyva-theme` (`Magewirephp_MagewireHyvaTheme`)
- `magewirephp/magewire-hyva-checkout` (`Magewirephp_MagewireHyvaCheckout`)
- `magewirephp/magewire-admin` (`Magewirephp_MagewireAdmin`)

There is no current core `themes/Hyva`, `themes/Luma`, `themes/Breeze`, or `themes/Backend` tree. Do not copy V1/in-tree paths or the removed `Magewirephp_MagewireCompatibilityWithHyva` module name.

## Workflow

1. Inspect the tag of Magewire and the companion package installed by the project. Companion packages release independently.
2. Keep reusable behavior in an application/service module; keep asset ordering, layout bridges, styles, and theme-specific BC in the compatibility module.
3. Sequence the compatibility module after `Magewirephp_Magewire` and the target theme module.
4. Register Features and Mechanisms only in area-scoped DI.
5. Extend Magewire's named layout tree; never copy the complete core layout.
6. Coordinate Alpine so exactly one runtime starts on pages both with and without Magewire components.
7. Render inline JavaScript through Magewire fragments for CSP.
8. Test a component page, a page without components, production static-content deployment, and the theme CSS build.

## Current boundaries

- Hyvä: let `magewire-hyva-theme` wrap the theme's Alpine loader. Do not remove Hyvä's Alpine globally.
- Hyvä Checkout BC: add `#[HandleBackwardsCompatibility]` explicitly on legacy components with Magewire 3.5. The current container fallback is unreliable because core writes an explicit false before the companion checks for an unset value.
- Admin: use `magewire-admin`; it owns the admin route, session checks, resolver, RequireJS/Prototype compatibility, and head injection. Its currently tagged layout still targets a core rate-limit block removed in Magewire 3.5; do not copy that obsolete override.
- Luma/Breeze: no maintained first-party Magewire 3 companion package exists. Build or select a project/community integration.

## References

- Creating or packaging an integration: read [module structure](references/module-structure.md).
- Selecting an injection point: read [layout nodes](references/layout-containers.md).
- Adding a Feature bridge or coordinating Alpine: read [extension examples](references/extension-examples.md).
- Integrating a Tailwind build: read [Tailwind](references/tailwind.md).
- Reviewing known source/package mismatches: read [current caveats](references/recommendations.md).

## Non-negotiable rules

- Do not register theme Features or Mechanisms in global `etc/di.xml`.
- Do not use `event.detail.magewire`, `Magewire.addon()`, or `Magewire.utility()`. Use `window.Magewire`, `window.MagewireAddons.register()`, and `window.MagewireUtilities.register()`.
- A Magento `referenceBlock` or `referenceContainer` modifies an existing node; neither replaces it merely by being referenced. Follow the current first-party layout pattern and change templates only intentionally.
- `magewire.script` is admin-package-specific, not a core storefront block.
- Do not invent Tailwind paths or CSS-variable contracts. Inspect the installed companion package.
