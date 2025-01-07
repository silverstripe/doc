---
title: Extending an existing ModelAdmin
summary: ModelAdmin interfaces that come with the core can be customised easily
---

# Extending existing `ModelAdmin`

Sometimes you'll work with ModelAdmins from other modules. To customise these interfaces, you can always subclass. But there's
also another tool at your disposal: The [Extension](api:SilverStripe\Core\Extension) API.

```php
namespace App\Extension;

use SilverStripe\Core\Extension;

class MyAdminExtension extends Extension
{
    protected function updateEditForm($form)
    {
        $form->Fields()->push(/* ... */)
    }
}
```

Now enable this extension through your `config.yml` file (see the [configuration documentation](/configuration/configuration/)).

```yml
MyAdmin:
  extensions:
    - App\Extension\MyAdminExtension
```

The following extension points are available: `updateEditForm()`, `updateList()`, `updateImportForm`.
