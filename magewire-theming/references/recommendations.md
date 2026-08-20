# Current Source Caveats

Treat these as version-specific audit findings, not general Magewire design rules. Recheck them when tags change.

## Magewire 3.5 and current companion tags

- Core is theme-agnostic and has no `themes/` directory. Old in-tree package paths and `Magewirephp_MagewireCompatibilityWithHyva` names are obsolete.
- `magewire-hyva-checkout` intends to enable BC under `hyva-checkout-main`, but the core resolver writes false before the fallback tests for an unset value. Use explicit `#[HandleBackwardsCompatibility]` until corrected and tested.
- `magewire-admin` still targets `magewire.features.support-magewire-rate-limiting`; core 3.5 replaced it with `magewire.features.support-magewire-request-filters`.
- `magewire.features` is declared as a block in core while first-party integrations may reference it using `referenceContainer`. Avoid claims that one reference tag is inherently additive and the other inherently replacing; inspect merged layout behavior.
- `magewire.script` is created by the admin package, not by the core storefront layout.
- The Hyvä Tailwind package scans its `src/view` tree and does not define the CSS variables older guidance claimed.

When a caveat affects an implementation, report the mismatch separately from the project change and avoid silently encoding the broken behavior as an API.
