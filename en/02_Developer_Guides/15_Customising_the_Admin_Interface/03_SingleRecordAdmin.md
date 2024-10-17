---
title: Single-record Admin Sections
summary: Create admin UI's for managing a single record with SingleRecordAdmin
---

# Single-record admin sections

The [`SingleRecordAdmin`](api:SilverStripe\Admin\SingleRecordAdmin) class lets you create an admin section which presents an edit form for a single record. Unlike [`ModelAdmin`](api:SilverStripe\Admin\ModelAdmin), there's no UI mechanism in a `SingleRecordAdmin` to swap what record you're editing.

The main use cases for using `SingleRecordAdmin` are for a settings section (where a single record holds all the settings needed for that section), or editing a record that is unique for each signed-in user (such as that user's profile).

```php
namespace App\Admin;

use App\Model\MySettingsModel;
use SilverStripe\Admin\SingleRecordAdmin;

class MySettingsAdmin extends SingleRecordAdmin
{
    private static $url_segment = 'my-settings';

    private static $menu_title = 'My Settings';

    private static $model_class = MySettingsModel::class;
}
```

This admin section will fetch a record of the `MySettingsModel` class using the [`get_one()`](api:SilverStripe\ORM\DataObject::get_one()) method.

If you don't want the admin section to fetch your record in this way, you can set the [`restrict_to_single_record`](api:SilverStripe\Admin\SingleRecordAdmin->restrict_to_single_record) configuration property to false. In this case you must provide another way for the admin section to know which record to edit. This could be in the form of a separate action on the controller (e.g. `edit/$ID`), or by calling [`setCurrentRecordID()`](api:SilverStripe\Admin\LeftAndMain::setCurrentRecordID()) in the [`init()`](api:SilverStripe\Admin\LeftAndMain::init()) method of the controller.

If there's no record to edit, by default it will create one for you. To disable that behaviour, set the [`allow_new_record`](api:SilverStripe\Admin\SingleRecordAdmin->allow_new_record) configuration property to false.

> [!NOTE]
> If you need more complex behaviour to fetch your record, for example you use the `silverstripe/subsites` or `tractorcow/silverstripe-fluent` module, you could either override the [`getSingleRecord()`](api:SilverStripe\Admin\SingleRecordAdmin::getSingleRecord()) method, or you could create [an extension](/developer_guides/extending/extensions/) that implements the `augmentSQL()` extension hook.
>
> The latter option is the most flexible, as it affects all high-level ORM queries for your model which ensures consistency when fetching the record from other areas of your code.

For records that hold settings, it's common to provide a static method to get the current settings record. Below is a basic example.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MySettingsModel extends DataObject
{
    // ...

    /**
     * Get the current settings record
     */
    public static function currentRecord(): MySettingsModel
    {
        $record = MySettingsModel::get_one();
        if (!$record) {
            $record = MySettingsModel::create();
            $record->write(skipValidation: true);
        }
        return $record;
    }
}
```
