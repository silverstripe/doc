---
title: Building Model and Search Interfaces around Scaffolding
summary: A Model-driven approach to defining your application UI.
icon: hammer
---

# Scaffolding

The ORM already has a lot of information about the data represented by a `DataObject` through its `$db` property, so
Silverstripe CMS will use that information to scaffold some interfaces. This is done though [FormScaffolder](api:SilverStripe\Forms\FormScaffolder)
to provide reasonable defaults based on the property type (e.g. a checkbox field for booleans). You can then further
customise those fields as required.

## Form fields

An example is `DataObject`, Silverstripe CMS will automatically create your CMS interface so you can modify what you need, without having to define all of your form fields from scratch.

Note that the [`SiteTree`](api:SilverStripe\CMS\Model\SiteTree) edit form does not use scaffolded fields.

```php
namespace App\Model;

use SilverStripe\Forms\FieldList;
use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...
    private static $db = [
        'IsActive' => 'Boolean',
        'Title' => 'Varchar',
        'Content' => 'Text',
    ];

    public function getCMSFields()
    {
        $this->beforeUpdateCMSFields(function (FieldList $fields) {
            $fields->dataFieldByName('IsActive')->setTitle('Is active?');
        });

        // parent::getCMSFields() does all the hard work and creates the fields for Title, IsActive and Content.
        return parent::getCMSFields();
    }
}
```

> [!TIP]
> It is typically considered a good practice to wrap your modifications in a call to [`beforeUpdateCMSFields()`](api:SilverStripe\ORM\DataObject::beforeUpdateCMSFields()) - the `updateCMSFields()` extension hook is triggered by `parent::getCMSFields()`, so this is how you ensure any new fields are added before extensions update your fieldlist.

To define the form fields yourself without using scaffolding, use the `mainTabOnly` option in [`DataObject.scaffold_cms_fields_settings`](api:SilverStripe\ORM\DataObject->scaffold_cms_fields_settings). See [scaffolding options](#scaffolding-options) for details.

```php
namespace App\Model;

use SilverStripe\Forms\CheckboxSetField;
use SilverStripe\Forms\FieldList;
use SilverStripe\Forms\TextField;
use SilverStripe\Forms\TextareaField;
use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...

    private static array $scaffold_cms_fields_settings = [
        'mainTabOnly' => true,
    ];

    public function getCMSFields()
    {
        $this->beforeUpdateCMSFields(function (FieldList $fields) {
            $fields->addFieldsToTab('Root.Main', [
                CheckboxSetField::create('IsActive', 'Is active?'),
                TextField::create('Title'),
                TextareaField::create('Content')->setRows(5),
            ]);
        });

        return parent::getCMSFields();
    }
}
```

You can also alter the fields of built-in and module `DataObject` classes by implementing `updateCMSFields()` in [your own Extension](/developer_guides/extending/extensions).

> [!NOTE]
> `FormField` scaffolding takes [`$field_labels` config](#field-labels) into account as well.

## Scaffolding options

`FormScaffolder` has several options that modify the way it scaffolds form fields.

|option|description|
|---|---|
|`tabbed`|Use tabs for the scaffolded fields. All database fields and `has_one` fields will be in a "Root.Main" tab. Fields representing `has_many` and `many_many` relations will either be in "Root.Main" or in "Root.`<relationname>`" tabs.|
|`mainTabOnly`|Only set up the "Root.Main" tab, but skip scaffolding actual form fields or relation tabs. If `tabbed` is false, the `FieldList` will be empty.|
|`restrictFields`|Allow list of field names. If populated, any database fields and fields representing `has_one` relations not in this array won't be scaffolded.|
|`ignoreFields`|Deny list of field names. If populated, database fields and fields representing `has_one` relations which *are* in this array won't be scaffolded.|
|`fieldClasses`|Optional mapping of field names to subclasses of `FormField`.|
|`includeRelations`|Whether to include `has_many` and `many_many` relations.|
|`restrictRelations`|Allow list of field names. If populated, form fields representing `has_many` and `many_many` relations not in this array won't be scaffolded.|
|`ignoreRelations`|Deny list of field names. If populated, form fields representing `has_many` and `many_many` relations which *are* in this array won't be scaffolded.|

You can set these options for the scaffolding of the fields in your model's `getCMSFields()` field list by setting the [`DataObject.scaffold_cms_fields_settings`](api:SilverStripe\ORM\DataObject->scaffold_cms_fields_settings) configuration property.

```php
namespace App\Model;

use SilverStripe\Forms\HiddenField;
use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...

    private static array $scaffold_cms_fields_settings = [
        'includeRelations' => false,
        'ignoreFields' => [
            'MyDataOnlyField',
        ],
        'fieldClasses' => [
            'MyHiddenField' => HiddenField::class,
        ],
    ];
}
```

You can also set this configuration in [extensions](/developer_guides/extending/extensions), for example if your extension is adding new database fields that you don't want to be edited via form fields in the CMS.

## Scaffolding for relations

Form fields are also automatically scaffolded for `has_one`, `has_many`, and `many_many` relations. These have sensible default implementations, and you can also customise what form field will be used for any given `DataObject` model.

> [!NOTE]
> Polymorphic `has_one` relations do not have scaffolded form fields. Usually these are managed via a `has_many` relation which points at the `has_one` relation.

With the below example, the following form fields will be scaffolded:

|relation|form field|
|---|---|
|`Child`|`MyCustomField`|
|`HasManyChildren`|`SearchableMultiDropdownField`|
|`ManyManyChildren`|`GridField`|

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...
    private static array $has_one = [
        'Child' => MyChild::class,
    ];

    private static array $has_many = [
        'HasManyChildren' => MyChild::class . '.Parent',
    ];

    private static array $many_many = [
        'ManyManyChildren' => MyChild::class,
    ];
}
```

```php
namespace App\Model;

use App\Form\MyCustomField;
use SilverStripe\Forms\FormField;
use SilverStripe\Forms\GridField\GridFieldAddExistingAutocompleter;
use SilverStripe\Forms\SearchableMultiDropdownField;
use SilverStripe\ORM\DataObject;

class MyChild extends DataObject
{
    // ...
    public function scaffoldFormFieldForHasOne(
        string $fieldName,
        ?string $fieldTitle,
        string $relationName,
        DataObject $ownerRecord
    ): FormField {
        // Return a form field that should be used for selecting this model type for has_one relations.
        return MyCustomField::create($fieldName, $fieldTitle);
    }

    public function scaffoldFormFieldForHasMany(
        string $relationName,
        ?string $fieldTitle,
        DataObject $ownerRecord,
        bool &$includeInOwnTab
    ): FormField {
        // If this should be in its own tab, set $includeInOwnTab to true, otherwise set it to false.
        $includeInOwnTab = false;
        // Return a form field that should be used for selecting this model type for has_many relations.
        return SearchableMultiDropdownField::create($relationName, $fieldTitle, static::get());
    }

    public function scaffoldFormFieldForManyMany(
        string $relationName,
        ?string $fieldTitle,
        DataObject $ownerRecord,
        bool &$includeInOwnTab
    ): FormField {
        // The default implementation for this method returns a GridField, which we can modify.
        $gridField = parent::scaffoldFormFieldForManyMany($relationName, $fieldTitle, $ownerRecord, $includeInOwnTab);
        $gridField->getConfig()->removeComponentsByType(GridFieldAddExistingAutocompleter::class);
        return $gridField;
    }
}
```

## Searchable fields

The `$searchable_fields` property uses a mixed array format that can be used to further customise your generated admin
system. The default is a set of array values listing the fields.

> [!NOTE]
> `$searchable_fields` will default to use the [`$summary_fields` config](#summary-fields), excluding anything that isn't a database field (such as method calls) if not explicitly defined.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...
    private static $searchable_fields = [
        'Name',
        'ProductCode',
    ];
}
```

> [!WARNING]
> If you define a `searchable_fields` configuration, *do not* specify fields that are not stored in the database (such as methods), as this will cause an error.

### General search field

Tabular views such as `GridField` or `ModelAdmin` include a search bar. The search bar will search across all of your searchable fields by default. It will return a match if the search terms appear in any of the searchable fields.

#### Exclude fields from the general search

If you have fields which you do *not* want to be searched with this general search (e.g. date fields which need special consideration), you can mark them as being explicitly excluded by setting `general` to false in the searchable fields configuration for that field:

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...
    private static $searchable_fields = [
        'Name',
        'BirthDate' => [
            'general' => false,
        ],
    ];
}
```

#### Customise the general search field name

By default the general search field uses the name "q". If you already use that field name or search query in your [SearchContext](/developer_guides/search/searchcontext), you can change this to whatever name you prefer either globally or per class:

> [!TIP]
> If you set `general_search_field_name` to any empty string, general search will be disabled entirely. Instead, the first field in your searchable fields configuration will be used.

##### Globally change the general search field name via YAML config {#general-field-name-yaml}

```yml
SilverStripe\ORM\DataObject:
  general_search_field_name: 'my_general_field_name'
```

##### Customise the general search field name via YAML *or* PHP config {#general-field-name-php}

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...
    private static string $general_search_field_name = 'my_general_field_name';
}
```

#### Specify a search filter for general search

By default, the general search will search across your fields using a [PartialMatchFilter](api:SilverStripe\ORM\Filters\PartialMatchFilter) regardless of what filters you have [specified for those fields](#specify-a-form-field-or-search-filter).

You can configure this to be a specific filter class, or else disable the general search filter. Disabling the filter will result in the filters you have specified for each field being used when searching against that field in the general search.

Like the general search field name, you can set this either globally or per class.

##### Globally change the general search filter via YAML config {#general-field-filter-yaml}

```yml
# use a specific filter
SilverStripe\ORM\DataObject:
  general_search_field_filter: 'SilverStripe\ORM\Filters\EndsWithFilter'

# or disable the filter to fall back on individual fields' filters
SilverStripe\ORM\DataObject:
  general_search_field_filter: ''
```

##### Customise the general search filter via YAML *or* PHP config {#general-field-filter-php}

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;
use SilverStripe\ORM\Filters\EndsWithFilter;

class MyDataObject extends DataObject
{
    // ...
    private static string $general_search_field_filter = EndsWithFilter::class;
}
```

> [!WARNING]
> You may get unexpected results using some filters if you don't disable splitting the query into terms - for example if you use an [ExactMatchFilter](api:SilverStripe\ORM\Filters\ExactMatchFilter), each term in the query *must* exactly match the value in at least one field to get a match. If you disable splitting terms, the whole query must exactly match a field value instead.

#### Splitting search queries into individual terms

By default the general search field will split your search query on spaces into individual terms, and search across your searchable field for each term. At least one field must match each term to get a match.

For example: with the search query "farm house" at least one field must have a match for the word "farm", and at least one field must have a match for the word "house". There does *not* need to be a field which matches the full phrase "farm house".

You can disable this behaviour by setting `DataObject.general_search_split_terms` to false. This would mean that for the example above a `DataObject` would need a field that matches "farm house" to be included in the results. Simply matching "farm" or "house" alone would not be sufficient.

Like the general search field name, you can set this either globally or per class.

##### Globally disable splitting terms via YAML config {#general-field-split-yaml}

```yml
SilverStripe\ORM\DataObject:
  general_search_split_terms: false
```

##### Disable splitting terms via YAML *or* PHP config {#general-field-split-php}

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...
    private static bool $general_search_split_terms = false;
}
```

#### Use a specific single field

If you disable the global general search functionality, the general seach field will revert to searching against the *first
field* in your `searchableFields` list.

As an example, let's look at a definition like this:

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...
    private static $searchable_fields = [
        'Name',
        'JobTitle',
    ];
}
```

That `Name` comes first in that list is actually quite a good thing. The user will likely want the
single search input to target the `Name` field rather something with a more predictable value,
like `JobTitle`.

By contrast, let's look at this definition:

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...
    private static $searchable_fields = [
        'Price',
        'Description',
        'Title',
    ];
}
```

It's unlikely that the user will want to search on `Price`. A better candidate would be `Title` or `Description`. Rather than reorder the array, which may be counter-intuitive, you can use the `general_search_field` configuration property.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...
    private static $general_search_field = 'Title';
}
```

##### Customise the field per `GridField`

You can customise the search field for a specific `GridField` by calling `setSearchField()` on its `GridFieldFilterHeader` component instance.

```php
$myGrid->getConfig()->getComponentByType(GridFieldFilterHeader::class)->setSearchField('Title');
```

This is useful if you have disabled the global general search functionality, if you have [customised the SearchContext](/developer_guides/search/searchcontext), or if you (for whatever reason) want to use a single specific search field for this `GridField`.

### Specify a form field or search filter

Searchable fields will appear in the search interface with a default form field (usually a [TextField](api:SilverStripe\Forms\TextField)) and a
default search filter assigned (usually a [PartialMatchFilter](api:SilverStripe\ORM\Filters\PartialMatchFilter)). To override these defaults, you can specify
additional information on `$searchable_fields`:

```php
namespace App\Model;

use SilverStripe\Forms\NumericField;
use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...
    private static $searchable_fields = [
        'Name' => 'PartialMatchFilter',
        'ProductCode' => [
            'field' => NumericField::class,
        ],
    ];
}
```

If you assign a single string value, you can set it to be a [`SearchFilter`](api:SilverStripe\ORM\Filters\SearchFilter) class. To specify a specific [`FormField`](api:SilverStripe\Forms\FormField) to use or
specify both a form field *and* a filter - or to combine this with other configuration - you can assign an array:

```php
namespace App\Model;

use SilverStripe\Forms\NumericField;
use SilverStripe\Forms\TextField;
use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...
    private static $searchable_fields = [
       'Name' => [
          'field' => TextField::class,
          'filter' => 'PartialMatchFilter',
       ],
       'ProductCode' => [
           'title' => 'Product code #',
           'field' => NumericField::class,
           'filter' => 'PartialMatchFilter',
       ],
    ];
}
```

#### Using `WithinRangeFilter` {#searchable-fields-withinrangefilter}

If you want users to be able to filter by a field using a range, specify the [`WithinRangeFilter`](api:SilverStripe\ORM\Filters\WithinRangeFilter). This works out of the box with the numeric, date, datetime, and time fields that come in Silverstripe framework.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;
SilverStripe\ORM\Filters\WithinRangeFilter

class MyDataObject extends DataObject
{
    // ...
    private static array $db = [
        'Price' => 'Currency',
    ];

    private static array $searchable_fields = [
        'Price' => [
            'filter' => WithinRangeFilter::class,
        ],
    ];
}
```

This configuration will duplicate the form field, providing one form field for the "from" value, and another for the "to" value. Users can then filter within a range using the filters in the CMS.

![filter by price within a range](../../_images/withinrangefilter.png)

If a user fills in only the "from" or "to" field, the other will be populated with the minimum or maximum value defined by the relevant `DBField` class in [`getMinValue()`](api:SilverStripe\ORM\FieldType\DBField::getMinValue()) or [`getMaxValue()`](api:SilverStripe\ORM\FieldType\DBField::getMaxValue())

This can also be used with other field types, but you need to define what the default "from" and "to" values should be. You can do this with the `rangeFromDefault` and `rangeToDefault` keys as shown below.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;
SilverStripe\ORM\Filters\WithinRangeFilter

class MyDataObject extends DataObject
{
    // ...
    private static array $db = [
        'Title' => 'Varchar',
    ];

    private static array $searchable_fields = [
        'Title' => [
            'filter' => WithinRangeFilter::class,
            'rangeFromDefault' => 'a',
            'rangeToDefault' => 'z',
        ],
    ];
}
```

### Searching on relations

To include relations (`$has_one`, `$has_many` and `$many_many`) in your search, you can use a dot-notation.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class Team extends DataObject
{
    // ...
    private static $db = [
        'Title' => 'Varchar',
    ];

    private static $many_many = [
        'Players' => 'Player',
    ];

    private static $searchable_fields = [
        'Title',
        'Players.Name',
    ];
}
```

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class Player extends DataObject
{
    // ...
    private static $db = [
        'Name' => 'Varchar',
        'Birthday' => 'Date',
    ];

    private static $belongs_many_many = [
        'Teams' => 'Team',
    ];
}
```

### Searching many db fields on a single search field

Use a single search field that matches on multiple database fields with `'match_any'`. This also supports specifying a `FormField` and a filter, though it is not necessary to do so.

> [!CAUTION]
> If you don't specify `field` or `dataType`, you must use the name of a real database field as the array key instead of a custom name so that a default field class can be determined.

```php
namespace App\Model;

use SilverStripe\Forms\TextField;

class Order extends DataObject
{
    // ...
    private static $db = [
        'Name' => 'Varchar',
    ];

    private static $has_one = [
        'Customer' => Customer::class,
        'ShippingAddress' => Address::class,
    ];

    private static $searchable_fields = [
        'CustomName' => [
            'title' => 'First Name',
            'field' => TextField::class,
            // Instead of defining "field" above, you could set "dataType" to a DBField instance like so:
            // 'dataType' => DBVarchar::class,
            'match_any' => [
                // Searching with the "First Name" field will show Orders matching either
                // Name, Customer.FirstName, or ShippingAddress.FirstName
                'Name',
                'Customer.FirstName',
                'ShippingAddress.FirstName',
            ],
        ],
    ];
}
```

You can also allow users to filter `match_any` with a range, using the configuration specified in [using `WithinRangeFilter`](#searchable-fields-withinrangefilter).

## Summary fields

Summary fields can be used to show a quick overview of the data for a specific [DataObject](api:SilverStripe\ORM\DataObject) record. The most common use
is their display as table columns, e.g. in the search results of a [ModelAdmin](api:SilverStripe\Admin\ModelAdmin) CMS interface.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...
    private static $db = [
        'Name' => 'Text',
        'OtherProperty' => 'Text',
        'ProductCode' => 'Int',
    ];

    private static $summary_fields = [
        'Name',
        'ProductCode',
    ];
}
```

### Relations in summary fields

To include relations or field manipulations in your summaries, you can use a dot-notation.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class OtherObject extends DataObject
{
    // ...
    private static $db = [
        'Title' => 'Varchar',
    ];
}
```

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...
    private static $db = [
        'Name' => 'Text',
        'Description' => 'HTMLText',
    ];

    private static $has_one = [
        'OtherObject' => 'OtherObject',
    ];

    private static $summary_fields = [
        'Name' => 'Name',
        'Description.Summary' => 'Description (summary)',
        'OtherObject.Title' => 'Other Object Title',
    ];
}
```

### Images in summary fields

Non-textual elements (such as images and their manipulations) can also be used in summaries.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...
    private static $db = [
        'Name' => 'Text',
    ];

    private static $has_one = [
        'HeroImage' => 'Image',
    ];

    private static $summary_fields = [
        'Name' => 'Name',
        'HeroImage.CMSThumbnail' => 'Hero Image',
    ];
}
```

## Field labels

In order to re-label any summary fields, you can use the `$field_labels` static. This will also affect the output of `$object->fieldLabels()` and `$object->fieldLabel()`.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...
    private static $db = [
        'Name' => 'Text',
    ];

    private static $has_one = [
        'HeroImage' => Image::class,
    ];

    private static $summary_fields = [
        'Name',
        'HeroImage.CMSThumbnail',
    ];

    private static $field_labels = [
        'Name' => 'Name',
        'HeroImage.CMSThumbnail' => 'Hero',
    ];
}
```

### Localisation {#field-label-localisation}

For any fields *not* defined in `$field_labels`, labels can be localised by defining the name prefixed by the type of field (e.g `db_`, `has_one_`, etc) in your localisation YAML files:

> [!NOTE]
> The class name should be the class that defined the field or relationship.

```yml
# app/lang/en.yml
en:
  App\Model\MyDataObject:
    db_Name: "Name"
    has_one_HeroImage: "Hero Image"
```

> [!WARNING]
> For relations (such as `has_one_HeroImage` above), this field label applies to the scaffolded form field (an `UploadField` for files, a tab for `has_many`/`many_many`, etc). It does *not* apply to summary or searchable fields with dot notation.

Labels you define in `$field_labels` *won't* be overridden by localisation strings. To make those localisable, you will need to override the [`fieldLabels()`](api:SilverStripe\ORM\DataObject) method and explicitly localise those labels yourself:

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyDataObject extends DataObject
{
    // ...

    public function fieldLabels($includerelations = true)
    {
        $labels = parent::fieldLabels($includerelations);
        $customLabels = static::config()->get('field_labels');

        foreach ($customLabels as $name => $label) {
            $labels[$name] = _t(__CLASS__ . '.' . $name, $label);
        }

        return $labels;
    }
}
```

See the [i18n section](/developer_guides/i18n) for more about localisation.

## Related documentation

- [SearchFilters](searchfilters)

## API documentation

- [FormScaffolder](api:SilverStripe\Forms\FormScaffolder)
- [DataObject](api:SilverStripe\ORM\DataObject)
