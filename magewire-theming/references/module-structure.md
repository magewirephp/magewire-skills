# Standalone Compatibility Module Structure

Create a standard standalone Magento module. A project-local integration can live under `app/code`; a reusable integration should be its own `magento2-module` Composer package.

```text
src/
├── registration.php
├── etc/
│   ├── module.xml
│   ├── frontend/di.xml
│   └── frontend/events.xml
├── Magewire/Features/
├── Observer/
└── view/frontend/
    ├── layout/
    ├── templates/
    └── tailwind/
```

Add only directories the integration needs.

## Registration

```php
<?php

use Magento\Framework\Component\ComponentRegistrar;

ComponentRegistrar::register(
    ComponentRegistrar::MODULE,
    'Vendor_MagewireMyTheme',
    __DIR__,
);
```

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
    <module name="Vendor_MagewireMyTheme">
        <sequence>
            <module name="Magewirephp_Magewire"/>
            <module name="Vendor_MyTheme"/>
        </sequence>
    </module>
</config>
```

Reusable package baseline:

```json
{
  "name": "vendor/magewire-my-theme",
  "type": "magento2-module",
  "require": {
    "php": ">=8.2",
    "magento/framework": "*",
    "magewirephp/magewire": "^3.5"
  },
  "autoload": {
    "files": ["src/registration.php"],
    "psr-4": {"Vendor\\MagewireMyTheme\\": "src/"}
  }
}
```

Choose the Magewire constraint from APIs actually used; do not copy `^3.5` when the package supports older releases.

## Area ownership

- Storefront: `etc/frontend/di.xml` and `view/frontend/`.
- Admin: `etc/adminhtml/di.xml` and `view/adminhtml/`, normally in a package depending on `magewirephp/magewire-admin`.
- Global `etc/di.xml`: ordinary Magento service preferences are allowed, but never register Magewire Features or Mechanisms there.

Do not add custom update controllers, admin-session logic, or RequireJS patches to a thin storefront theme adapter. Those are infrastructure responsibilities; depend on `magewire-admin` for the backend or create a deliberately versioned infrastructure package.
