---
title: Managing Multiple Records
summary: Create admin UI's for managing multiple records at once with ModelAdmin
---

# Managing multiple records

[ModelAdmin](api:SilverStripe\Admin\ModelAdmin) provides a simple way to manage multiple records at once with your own data models. It can create
searchables list and edit views of [DataObject](api:SilverStripe\ORM\DataObject) subclasses, and even provides import and export of your data.

It uses the framework's knowledge about the model to provide sensible defaults, allowing you to get started in a couple
of lines of code, while still providing a solid base for customization.

> [!NOTE]
> The interface is mainly powered by the [GridField](api:SilverStripe\Forms\GridField\GridField) class ([documentation](../forms/field_types/gridfield)), which can
> also be used in other areas of your application.

Let's assume we want to manage a simple product listing as a sample data model: A product can have a name, price, and
a category.

```php
// app/src/Model/Product.php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class Product extends DataObject
{
    private static $db = [
        'Name' => 'Varchar',
        'ProductCode' => 'Varchar',
        'Price' => 'Currency',
    ];

    private static $has_one = [
        'Category' => Category::class,
    ];
}
```

```php
// app/src/Model/Category.php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class Category extends DataObject
{
    private static $db = [
        'Title' => 'Text',
    ];

    private static $has_many = [
        'Products' => Product::class,
    ];
}
```

To create your own `ModelAdmin`, simply extend the base class, and edit the `$managed_models` property with the list of
DataObject's you want to scaffold an interface for. The class can manage multiple models in parallel, if required.

We'll name it `MyAdmin`, but the class name can be anything you want.

```php
// app/src/Admin/MyAdmin.php
namespace App\Admin;

use App\Model\Category;
use App\Model\Product;
use SilverStripe\Admin\ModelAdmin;

class MyAdmin extends ModelAdmin
{
    private static $managed_models = [
        Product::class,
        Category::class,
    ];

    private static $url_segment = 'products';

    private static $menu_title = 'My Product Admin';
}
```

This will automatically add a new menu entry to the Silverstripe CMS UI entitled `My Product Admin` and logged in
users will be able to upload and manage `Product` and `Category` instances through `https://www.example.com/admin/products`.

> [!CAUTION]
> After defining these classes, make sure you have rebuilt your Silverstripe CMS database and flushed your cache.

## Defining the `ModelAdmin` models

The `$managed_models` configuration supports additional formats allowing you to customise
the URL and tab label used to access a specific model. This can also be used to display
the same model more than once with different filtering or display options.

```php
namespace App\Admin;

use App\Model\Category;
use App\Model\Product;
use SilverStripe\Admin\ModelAdmin;

class MyAdmin extends ModelAdmin
{
    private static $managed_models = [
        // This is the most basic format. URL for this Model will use the fully
        // qualified namespace of `Product`. The label for this tab will be determined
        // by the `i18n_plural_name` on the `Product` class.
        Product::class,

        // This format can be used to customise the tab title.
        Category::class => [
            'title' => 'All categories',
        ],

        // This format can be used to customise the URL segment for this Model. This can
        // be useful if you do not want the fully qualified class name of the Model to
        // appear in the URL. It can also be used to have the same Model appear more than
        // once, allowing you to create custom views.
        'product-category' => [
            'dataClass' => Category::class,
            'title' => 'Product categories',
        ],
    ];

    private static $url_segment = 'products';

    private static $menu_title = 'My Product Admin';

    public function getList()
    {
        $list = parent::getList();
        // Only show Categories specific to Products When viewing the product-category tab
        if ($this->modelTab === 'product-category') {
            $list = $list->filter('IsProductCategory', true);
        }
        return $list;
    }
}
```

### Edit links for records

It is trivial to get links to the edit form for managed records.

> [!NOTE]
> The [getLinkForModelClass()](api:SilverStripe\Admin\ModelAdmin::getLinkForModelClass()) method returns a link
> for the first tab defined for that class. If you have multiple tabs for a given class (as in the example above)
> it is better to use [getLinkForModelTab()](api:SilverStripe\Admin\ModelAdmin::getLinkForModelTab()) which will
> give you a link for the specific tab you pass in.

```php
$admin = MyAdmin::singleton();
if ($admin->isManagedModel(Product::class)) {
    // Get the link to the tab holding the record's gridfield
    $tabLink = $admin->getLinkForModelClass(Product::class);
    // Get the link to edit the record itself
    $editLink = $admin->getCMSEditLinkForManagedDataObject($someProduct);
}
// Get the link for a specific tab in the model admin
$tabLink = $admin->getLinkForModelTab('product-category');
```

> [!TIP]
> If you want `getLinkForModelClass()` to return the link for a specific tab, you can override the
> [getModelTabForModelClass()](api:SilverStripe\Admin\ModelAdmin::getModelTabForModelClass()) method
> for your `ModelAdmin` subclass.

You can also use the new [CMSEditLinkExtension](api:SilverStripe\Admin\CMSEditLinkExtension) to update the value of the `getCMSEditLink()` method on the record - see [Managing Records](../model/managing_records#getting-an-edit-link).

## Permissions

Each new `ModelAdmin` subclass creates its' own [permission code](../security), for the example above this would be
`CMS_ACCESS_MyAdmin`. Users with access to the Admin UI will need to have this permission assigned through
`admin/security/` or have the `ADMIN` permission code in order to gain access to the controller.

> [!WARNING]
> For more information on the security and permission system see the [Security Documentation](../security)

The [DataObject](api:SilverStripe\ORM\DataObject) API has more granular permission control, which is enforced in [ModelAdmin](api:SilverStripe\Admin\ModelAdmin) by default.
Available checks are `canEdit()`, `canCreate()`, `canView()` and `canDelete()`. Models check for administrator
permissions by default. For most cases, less restrictive checks make sense, e.g. checking for general CMS access rights.

```php
// app/src/Model/Category.php
namespace App\Model;

use SilverStripe\ORM\DataObject;
use SilverStripe\Security\Permission;

class Category extends DataObject
{
    // ...

    public function canView($member = null)
    {
        return Permission::check('CMS_ACCESS_Company\Website\MyAdmin', 'any', $member);
    }

    public function canEdit($member = null)
    {
        return Permission::check('CMS_ACCESS_Company\Website\MyAdmin', 'any', $member);
    }

    public function canDelete($member = null)
    {
        return Permission::check('CMS_ACCESS_Company\Website\MyAdmin', 'any', $member);
    }

    public function canCreate($member = null)
    {
        return Permission::check('CMS_ACCESS_Company\Website\MyAdmin', 'any', $member);
    }
}
```

## Custom `ModelAdmin` CSS menu icons using built in icon font

An extended ModelAdmin class supports adding a custom menu icon to the CMS.

```php
namespace App\Admin;

use SilverStripe\Admin\ModelAdmin;

class NewsAdmin extends ModelAdmin
{
    private static $menu_icon_class = 'font-icon-news';
    // ...
}
```

A complete list of supported font icons is available to view in the [Silverstripe CMS Design System Manager](https://projects.invisionapp.com/dsm/silver-stripe/silver-stripe/section/icons/5a8b972d656c91001150f8b6)

## Searching records

[ModelAdmin](api:SilverStripe\Admin\ModelAdmin) uses the [SearchContext](../search/searchcontext) class to provide a search form, as well as get the
searched results. Every [DataObject](api:SilverStripe\ORM\DataObject) can have its own context, based on the fields which should be searchable. The
class makes a guess at how those fields should be searched, e.g. showing a checkbox for any boolean fields in your
`$db` definition.

To remove, add or modify searchable fields, define a new [DataObject::$searchable_fields](api:SilverStripe\ORM\DataObject::$searchable_fields) static on your model
class (see [Searchable Fields](/developer_guides/model/scaffolding#searchable-fields) and [SearchContext](../search/searchcontext) docs for details).

```php
// app/src/Model/Product.php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class Product extends DataObject
{
    // ...

    private static $searchable_fields = [
      'Name',
      'ProductCode',
    ];
}
```

> [!TIP]
> [SearchContext](../search/searchcontext) documentation has more information on providing the search functionality.

## Displaying results

The results are shown in a tabular listing, powered by the [GridField](../forms/field_types/gridfield), more specifically
the [GridFieldDataColumns](api:SilverStripe\Forms\GridField\GridFieldDataColumns) component. This component looks for a [DataObject::$summary_fields](api:SilverStripe\ORM\DataObject::$summary_fields) static on your
model class, where you can add or remove columns. To change the title, use [DataObject::$field_labels](api:SilverStripe\ORM\DataObject::$field_labels).
See [Summary Fields](/developer_guides/model/scaffolding#summary-fields) and [Field labels](/developer_guides/model/scaffolding#field-labels) for details.

```php
// app/src/Model/Product.php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class Product extends DataObject
{
    // ...

    private static $field_labels = [
      // renames the column to "Cost"
      'Price' => 'Cost',
    ];

    private static $summary_fields = [
      'Name',
      'Price',
    ];
}
```

The results list are retrieved from [SearchContext::getResults()](api:SilverStripe\ORM\Search\SearchContext::getResults()), based on the parameters passed through the search
form. If no search parameters are given, the results will show every record. Results are a [DataList](api:SilverStripe\ORM\DataList) instance, so
can be customized by additional SQL filters, joins.

For example, we might want to exclude all products without prices in our sample `MyAdmin` implementation.

```php
// app/src/Admin/MyAdmin.php
namespace App\Admin;

use App\Model\Product;
use SilverStripe\Admin\ModelAdmin;
use SilverStripe\ORM\DataObject;

class MyAdmin extends ModelAdmin
{
    // ...

    public function getList()
    {
        $list = parent::getList();

        // Always limit by model class, in case you're managing multiple
        if ($this->modelClass == Product::class) {
            $list = $list->exclude('Price', '0');
        }

        return $list;
    }

    public function getCMSEditLinkForManagedDataObject(DataObject $obj): string
    {
        if (!$obj->Price) {
            // We don't manage models without a price here, so we can't provide an edit link for them.
            return '';
        }
        return parent::getCMSEditLinkForManagedDataObject($obj);
    }
}
```

You can also customize the search behavior directly on your `ModelAdmin` instance. For example, we might want to have a
checkbox which limits search results to expensive products (over $100).

```php
// app/src/Admin/MyAdmin.php
namespace App\Admin;

use App\Model\Product;
use SilverStripe\Admin\ModelAdmin;
use SilverStripe\Forms\CheckboxField;

class MyAdmin extends ModelAdmin
{
    // ...

    public function getSearchContext()
    {
        $context = parent::getSearchContext();

        if ($this->modelClass == Product::class) {
            $context->getFields()->push(CheckboxField::create('q[ExpensiveOnly]', 'Only expensive stuff'));
        }

        return $context;
    }

    public function getList()
    {
        $list = parent::getList();

        // use this to access search parameters
        $params = $this->getRequest()->requestVar('q');

        if ($this->modelClass == Product::class && isset($params['ExpensiveOnly']) && $params['ExpensiveOnly']) {
            $list = $list->exclude('Price:LessThan', '100');
        }

        return $list;
    }
}
```

## Altering the `ModelAdmin`, `GridField`, or `Form`

If you wish to provided a tailored esperience for CMS users, you can directly interact with the ModelAdmin form or gridfield. Override the following method:

- `getEditForm()` to alter the Form object
- `getGridField()` to alter the GridField field
- `getGridFieldConfig()` to alter the GridField configuration.

Extensions applied to a ModelAdmin can also use the `updateGridField` and `updateGridFieldConfig` hooks.

To alter how the results are displayed (via [`GridField`](api:SilverStripe\Forms\GridField\GridField)), you can also override the `getEditForm()` method. For
example, to add a new component.

### Overriding the methods on `ModelAdmin`

```php
// app/src/Admin/MyAdmin.php
namespace App\Admin;

use App\Model\Category;
use App\Model\Product;
use SilverStripe\Admin\ModelAdmin;
use SilverStripe\Forms\GridField\GridFieldConfig;
use SilverStripe\Forms\GridField\GridFieldFilterHeader;

class MyAdmin extends ModelAdmin
{
    private static $managed_models = [
        Product::class,
        Category::class,
    ];

    private static $url_segment = 'my-admin';

    protected function getGridFieldConfig(): GridFieldConfig
    {
        $config = parent::getGridFieldConfig();

        $config->addComponent(GridFieldFilterHeader::create());

        return $config;
    }
}
```

The above example will add the component to all `GridField`s (of all managed models). Alternatively we can also add it
to only one specific `GridField`:

```php
// app/src/Admin/MyAdmin.php
namespace App\Admin;

use App\Model\Category;
use App\Model\Product;
use SilverStripe\Admin\ModelAdmin;
use SilverStripe\Forms\GridField\GridFieldConfig;
use SilverStripe\Forms\GridField\GridFieldFilterHeader;

class MyAdmin extends ModelAdmin
{
    private static $managed_models = [
        Product::class,
        Category::class,
    ];

    private static $url_segment = 'my-admin';

    protected function getGridFieldConfig(): GridFieldConfig
    {
        $config = parent::getGridFieldConfig();

        // modify the list view.
        if ($this->modelClass === Product::class) {
            $config->addComponent(GridFieldFilterHeader::create());
        }

        return $config;
    }
}
```

### Using an extension to customise a `ModelAdmin`

You can use an Extension to achieve the same results. Extensions have the advantage of being reusable in many contexts.

```php
// app/src/Extension/ModelAdminExtension.php
namespace App\Extension;

use SilverStripe\Core\Extension;
use SilverStripe\Forms\GridField\GridFieldConfig;
use SilverStripe\Forms\GridField\GridFieldFilterHeader;

/**
 * You can apply this extension to a GridField.
 */
class ModelAdminExtension extends Extension
{
    protected function updateGridFieldConfig(GridFieldConfig &$config)
    {
        $config->addComponent(GridFieldFilterHeader::create());
    }
}
```

```yml
# app/_config/extensions.yml
App\Admin\MyAdmin:
  extensions:
    - App\Extension\ModelAdminExtension
```

### Altering a `ModelAdmin` using only `getEditForm()`

This requires a bit more work to access the GridField and GridFieldConfig instances, but it can be useful for advanced modifications for the edit form.

```php
// app/src/Admin/MyAdmin.php
namespace App\Admin;

use App\Model\Category;
use App\Model\Product;
use SilverStripe\Admin\ModelAdmin;
use SilverStripe\Forms\GridField\GridFieldFilterHeader;

class MyAdmin extends ModelAdmin
{
    private static $managed_models = [
        Product::class,
        Category::class,
    ];

    private static $url_segment = 'my-admin';

    public function getEditForm($id = null, $fields = null)
    {
        $form = parent::getEditForm($id, $fields);

        // $gridFieldName is generated from the ModelClass, eg if the Class 'Product'
        // is managed by this ModelAdmin, the GridField for it will also be named 'Product'
        $gridFieldName = $this->sanitiseClassName($this->modelClass);
        $gridField = $form->Fields()->fieldByName($gridFieldName);

        // modify the list view.
        $gridField->getConfig()->addComponent(GridFieldFilterHeader::create());

        return $form;
    }
}
```

## Data import

The `ModelAdmin` class provides import of CSV files through the [CsvBulkLoader](api:SilverStripe\Dev\CsvBulkLoader) API. which has support for column
mapping, updating existing records, and identifying relationships - so its a powerful tool to get your data into a
Silverstripe CMS database.

By default, each model management interface allows uploading a CSV file with all columns auto detected. To override
with a more specific importer implementation, use the [ModelAdmin::$model_importers](api:SilverStripe\Admin\ModelAdmin::$model_importers) static.

## Data export

Export is available as a CSV format through a button at the end of a results list. You can also export search results.
This is handled through the [GridFieldExportButton](api:SilverStripe\Forms\GridField\GridFieldExportButton) component.

To customize the exported columns, create a new method called `getExportFields` in your `ModelAdmin`:

```php
namespace App\Admin;

use SilverStripe\Admin\ModelAdmin;

class MyAdmin extends ModelAdmin
{
    // ...

    public function getExportFields()
    {
        return [
            'Name' => 'Name',
            'ProductCode' => 'Product Code',
            'Category.Title' => 'Category',
        ];
    }
}
```

## Related documentation

- [GridField](../forms/field_types/gridfield)
- [Permissions](../security/permissions)
- [SearchContext](../search/searchcontext)

## API documentation

- [ModelAdmin](api:SilverStripe\Admin\ModelAdmin)
- [LeftAndMain](api:SilverStripe\Admin\LeftAndMain)
- [GridField](api:SilverStripe\Forms\GridField\GridField)
- [DataList](api:SilverStripe\ORM\DataList)
- [CsvBulkLoader](api:SilverStripe\Dev\CsvBulkLoader)
