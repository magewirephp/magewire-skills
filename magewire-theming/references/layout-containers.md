# Layout Nodes

Read the installed Magewire tag's `src/view/base/layout/default.xml` before targeting a node. Magewire 3.5 exposes this relevant tree:

```text
magewire
├── magewire.global
│   ├── magewire.global.before
│   │   ├── magewire.alpinejs.load
│   │   ├── magewire.alpinejs
│   │   └── magewire.alpinejs.components
│   ├── magewire.utilities
│   │   └── magewire.utilities.after
│   ├── magewire.addons
│   │   └── magewire.addons.after
│   └── magewire.global.after
├── magewire.before
│   ├── magewire.alpinejs.directives
│   ├── magewire.ui-components
│   └── magewire.alpinejs.after
├── magewire.internal
│   └── magewire.internal.backwards-compatibility
├── magewire.directives
├── magewire.features
├── magewire.after.internal
├── magewire.disabled
├── magewire.after
└── magewire.legacy
```

## Selection

| Need | Target |
|---|---|
| Theme Alpine loader | `magewire.alpinejs.load` |
| Reusable Alpine data | `magewire.alpinejs.components` |
| Utility | `magewire.utilities.after` |
| Stateful addon | `magewire.addons.after` |
| Rendered Alpine UI | `magewire.ui-components` |
| V1 browser shim | `magewire.internal.backwards-compatibility` |
| Magewire directive | `magewire.directives` |
| Feature browser bridge | `magewire.features` |
| General late output | `magewire.after` |

Do not override the `magewire` root or `magewire.internal`. `magewire.script` belongs to `magewire-admin` and is not a storefront target.

## References are not replacements

`referenceBlock` and `referenceContainer` address an existing node. Replacement occurs when the reference changes a template or arguments; adding a child remains additive. First-party companion packages sometimes use `referenceContainer` against nodes rendered by block templates, so inspect the merged layout rather than relying on a simplistic tag rule.

```xml
<referenceContainer name="magewire.features">
    <block name="vendor.magewire.features.my-bridge"
           template="Vendor_Module::magewire-features/my-bridge.phtml"/>
</referenceContainer>
```

Use `before`/`after` only against siblings guaranteed by a sequenced dependency. Prefer dedicated `.after` containers for custom utilities and addons.
