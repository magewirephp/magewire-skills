# Extension Examples

Use these as shapes, then verify names and APIs against the installed tags.

## Area-scoped Feature

```xml
<!-- src/etc/frontend/di.xml -->
<type name="Magewirephp\Magewire\Features">
    <arguments>
        <argument name="items" xsi:type="array">
            <item name="vendor_theme_bridge" xsi:type="array">
                <item name="type" xsi:type="string">Vendor\MagewireMyTheme\Magewire\Features\SupportThemeBridge</item>
                <item name="sort_order" xsi:type="number">6000</item>
            </item>
        </argument>
    </arguments>
</type>
```

```php
<?php

namespace Vendor\MagewireMyTheme\Magewire\Features;

use Magewirephp\Magewire\ComponentHook;
use function Magewirephp\Magewire\on;

final class SupportThemeBridge extends ComponentHook
{
    public function provide(): void
    {
        on('dehydrate', function ($component, $context): void {
            // Push a small, JSON-safe effect when the theme needs it.
        });
    }
}
```

Check the actual callback signature at the tagged `trigger()` call site. Do not guess lifecycle arguments.

## CSP-safe browser bridge

```xml
<referenceContainer name="magewire.features">
    <block name="vendor.magewire.features.theme-bridge"
           template="Vendor_MagewireMyTheme::magewire-features/theme-bridge.phtml"/>
</referenceContainer>
```

```php
<?php
$viewModel = $block->getData('view_model');
$script = $viewModel->utils()->fragment()->make()->script()->start();
?>
<script>
    document.addEventListener('magewire:init', () => {
        window.Magewire.hook('commit', ({ succeed }) => {
            succeed(({ effects }) => {
                // Theme bridge.
            })
        })
    }, { once: true })
</script>
<?php $script->end(); ?>
```

Use the globals directly. `event.detail.magewire` and `Magewire.addons` are not current APIs.

## Addon or utility

```javascript
window.MagewireAddons.register('theme', () => ({ open: false }), true)
window.MagewireUtilities.register('format', () => ({ value: value => String(value) }))
```

Place the templates below `magewire.addons.after` or `magewire.utilities.after`. The third addon argument enables Alpine reactivity.

## Alpine coordination

Do not implement a blanket “remove the theme's Alpine” rule. A compatibility package must preserve theme Alpine behavior on pages without Magewire while ensuring Magewire's bundled runtime is the only instance started on component pages.

For Hyvä, use `magewirephp/magewire-hyva-theme`; its layout wraps Hyvä's `script-alpine-js` block and selects the appropriate child. Copying one old template from this package without its complete layout relationship is unsafe.

Test:

1. a page containing a Magewire component;
2. a page containing Alpine behavior but no Magewire component;
3. a full-page-cache hit for each;
4. production static content after cache clearing.

## Hyvä Checkout BC

```php
use Magewirephp\Magewire\Features\SupportMagewireBackwardsCompatibility\HandleBackwardsCompatibility;

#[HandleBackwardsCompatibility]
final class LegacyCheckoutComponent extends \Magewirephp\Magewire\Component
{
}
```

Use the explicit attribute with Magewire 3.5. The companion package's `hyva-checkout-main` fallback currently checks too late to override the core resolver's explicit false reliably.

## Admin

Depend on `magewirephp/magewire-admin`; do not recreate its route and head injection inside an ordinary theme adapter. Admin components still bind the component object through the `magewire` layout argument; the resolver's `layout_admin` accessor is internal snapshot metadata, not the value application layout XML should put in that argument.

The currently tagged admin layout references `magewire.features.support-magewire-rate-limiting`, removed from core in 3.5. Do not copy that reference; core now renders `magewire.features.support-magewire-request-filters`.
