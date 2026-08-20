# Tailwind Integration

Tailwind belongs to the compatibility package, not Magewire's theme-agnostic contract.

## Hyvä

The current `magewirephp/magewire-hyva-theme` source uses:

```css
@source "../../../../../src/view";
```

and imports its notifier CSS. That path is relative to the package stylesheet. Never paste it into a project stylesheet without resolving it from the new file location.

Use the inclusion mechanism supported by the installed Hyvä version and inspect the tagged companion package. Do not assume the old in-tree observer/config structure still exists.

## Custom Tailwind theme

Scan only installed packages containing classes needed by the storefront:

- `vendor/magewirephp/magewire/src/view`
- `vendor/magewirephp/magewire-hyva-theme/src/view` when installed
- project compatibility-module view files

Other companion packages may not contain Tailwind sources. Verify their trees rather than adding speculative globs.

Magewire does not publish a stable `--notifier-*` or `--wire-loading-spinner` CSS-variable contract. Use the CSS and rendered selectors from the installed package, isolate project overrides, and recheck them after upgrades.

## Verification

1. Resolve every `@source`/content path from the file that declares it.
2. Build production CSS and confirm a package-specific class survives.
3. Test notifier and loading UI at the intended breakpoints.
4. Deploy Magento static content and clear generated view assets.

Magento admin does not use this storefront Tailwind pipeline.
