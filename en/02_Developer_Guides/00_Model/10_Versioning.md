---
title: Versioning
summary: Add versioning to your database content through the Versioned extension.
---

# Versioning

Database content in Silverstripe CMS can be "staged" before its publication, as well as track all changes through the
lifetime of a database record.

Versioning in Silverstripe CMS is handled through the [`Versioned`](api:SilverStripe\Versioned\Versioned) extension class. As an [`Extension`](api:SilverStripe\Core\Extension) it is possible to be applied to any [`DataObject`](api:SilverStripe\ORM\DataObject) subclass. The extension class will automatically update read and write operations performed via the ORM because it implements the `augmentSQL` extension hooks defined in [`DataQuery::getFinalisedQuery()`](api:SilverStripe\ORM\DataQuery::getFinalisedQuery()).

The `Versioned` extension is applied to pages in the CMS (the [`SiteTree`](api:SilverStripe\CMS\Model\SiteTree) class) - along with some other core `DataObject` models such as files - by default. Draft content edited in the CMS can be different
from published content shown to your website visitors.

> [!WARNING]
> There are two complementary modules that improve content editor experience around "owned" nested objects (e.g. elemental blocks).
> Those are in experimental status right now, but we would appreciate any feedback and contributions.
>
> You can check them out on GitHub:
>
> - <https://github.com/silverstripe/silverstripe-versioned-snapshots>
> - <https://github.com/silverstripe/silverstripe-versioned-snapshot-admin>
>
> The first one adds extra metadata to versions about object parents at the moment of version creation.
> The second module extends CMS History UI adding control over nested objects.
>
> Here is an example screenshot from `silverstripe/versioned-snapshot-admin`:
> ![a screenshot of the versioned-snapshot-admin module's "history" interface showing version history of data relations](../../_images/snapshot-admin.png)

## Understanding versioning concepts

This section discusses how Silverstripe CMS implements versioning and related high level concepts without digging into technical details. For the technical part, skip to [Implementing a versioned DataObject](#implementing-a-versioned-dataobject) below.

### Stages

In most cases, you'll want to have one polished version of a `Page` visible to the general public while your editors might be making additional changes on a draft version. Silverstripe CMS handles this through the concept of *stages*.

By default, adding the `Versioned` extension to a `DataObject` will create 2 stages:

- "Stage" for tracking draft content (aka "draft")
- "Live" for tracking content publicly visible (aka "published").

> [!NOTE]
> Yes, the draft stage is called the "Stage" stage. In this documentation we'll try to differentiate between the stage named "Stage" and the concept of a stage by giving the named stage a capital S and putting quotes around it - but in some cases we'll just refer to it as "draft" because often that's the more intuitive way to think of it.

Publishing a versioned `DataObject` is equivalent to copying the version from the "Stage" stage to the "Live" stage.

If you just want to keep track of the version history of a model's records but you don't need to separate draft and published versions, you can apply the `Versioned` extension to your `DataObject` without stages. This will allow you to keep track of all changes that have been applied to a DataObject and who made them.

> [!TIP]
> The `Versioned` class has a `Versioned::DRAFT` constant to refer to the "Stage" stage, and `Versioned::LIVE` to refer to the "Live" stage. It can be useful to use those in your PHP code when you need to refer to the stages.

### Ownership and relations between `DataObject` models {#ownership}

Typically when publishing versioned DataObjects, it is necessary to ensure that some linked components
are published along with it. Unless this is done, site content can appear incorrectly published.

For instance, a page which has a list of rotating banners will require them to be published
whenever that page is.

This is solved through the Ownership API, which declares that one model "owns" the models in a given existing relationship for the purposes of versioning and staging.
It relies on a pre-existing relationship to function.

#### Cascade publishing

If an object "owns" other objects, you'll usually want to publish the child objects when the parent object gets published. If those child objects themselves own other objects, you'll want the grand-children to be published along with the parent.

Silverstripe CMS makes this possible by using the concept of *cascade publishing*. You can choose to recursively publish an object. When an object is recursively published – either through a user action or through code – all other records it owns that implement the `Versioned` extension will automatically be published. Publication will also cascade to children of children and so on.

A non-recursive publish operation is also available if you want to publish a new version of a object without cascade publishing all its children.

> [!CAUTION]
> Declaring ownership implies publish permissions on owned objects.
> Built-in controllers using cascading publish operations check `canPublish()`
> on the owner, but not on the owned object.

#### Ownership of unversioned object

An unversioned object can own a versioned object.

An unversioned object can be configured to automatically publish owned versioned objects on save.

An unversioned object can also be owned by a versioned object. This can be used to recursively publish *children-of-children* objects without requiring the intermediate relationship to go through a versioned object. This behavior can be helpful if you wish to group multiple versioned objects together.

#### Ownership through media insertion in content

Images and other files are tracked as versioned objects. If a file is referenced through an HTML text field, it needs to be published for it to be accessible to the public. Silverstripe CMS will automatically pick up when an object references files through an HTML text field and recursively publish those files.

This behavior works both for versioned and unversioned objects.

### Grouping versioned `DataObject` records into a `ChangeSet`

Sometimes, multiple pages or records may be related in organic ways that cannot be properly expressed through an ownership relation. There's still value in being able to publish those as a block.

For example, your editors may be about to launch a new contest through their website. They've drafted a page to promote the contest, another page with the rules and conditions, a registration page for users to sign up, some promotional images, new sponsor records, etc. All this content needs to become visible simultaneously.

Changes to many objects can be grouped together using the [`ChangeSet`](api:SilverStripe\Versioning\ChangeSet) object. By grouping a series of content changes together as a cohesive unit. This can allow bulk publish an entire body of content all at once, which affords them much more power and control over interdependent content types. Note that there is currently no UI to enable this functionality, so it must be done programmatically.

Programmatically, this is done by creating a `ChangeSet` object and invoking its [`addObject(DataObject $record)`](api:SilverStripe\Versioning\ChangeSet::addObject()) method.

> [!NOTE]
> DataObjects can be added to more than one ChangeSet.
> Most of the time, these objects contain changes.
> A ChangeSet can contain unchanged objects as well.

#### Implicit vs. Explicit inclusions

Items can be added to a changeset in two ways -- *implicitly* and *explicitly*.

An *implicit* inclusion occurs when a record is added to a changeset by virtue of another object declaring ownership of it via the `$owns` setting. Implicit inclusion of owned objects ensures that when a changeset is published, the action cascades through not only all of the items explicitly added to the changeset, but also all of the records that each of those items owns.

An *explicit* inclusion is much more direct, occurring only when a user has opted to include a record in a changeset either through the UI or programmatically.

It is possible for an item to be included both implicitly and explicitly in a changeset. For instance, if a page owns a file, and the page gets added to a changeset, the file is implicitly added. That same file, however, can still be added to the changeset explicitly through the file editor. In this case, the file is considered to be *explicitly* added. If the file is later removed from the changeset, it is then considered *implicitly* added, due to its owner page still being in the changeset.

## Implementing a versioned `DataObject`

This section explains how to take a regular `DataObject` and add versioning to it.

### Applying the `Versioned` extension to your `DataObject`

> [!WARNING]
> Versioning only works if you are adding the extension to the base class. That is, the first subclass
> of `DataObject`. Adding this extension to children of the base class will have unpredictable behaviour.

Adding versioning to a `DataObject` model is as easy as applying the [`Versioned`](api:SilverStripe\Versioned\Versioned) extension to it, either via PHP or YAML configuration. This will apply versioning *with stages*, meaning you can have a draft and a published version of your records.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;
use SilverStripe\Versioned\Versioned;

class MyStagedModel extends DataObject
{
    private static $extensions = [
        Versioned::class,
    ];
}
```

```yml
App\Model\MyStagedModel:
  extensions:
    - SilverStripe\Versioned\Versioned
```

Alternatively, staging can be disabled, so that only versioned changes are tracked for your model. This
can be specified by using the `.versioned` service variant that provides only version history, and no
staging.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;
use SilverStripe\Versioned\Versioned;

class VersionedModel extends DataObject
{
    private static $extensions = [
        Versioned::class . '.versioned',
    ];
}
```

```yml
App\Model\MyStagedModel:
  extensions:
    - SilverStripe\Versioned\Versioned.versioned
```

> [!WARNING]
> The `Versioned` extension is automatically applied to the `SiteTree` class. For more information on extensions see
> [extending](/developer_guides/extending/) and the [Configuration](/developer_guides/configuration/) documentation.

#### Versioning a `many_many` relation

If you want to track versions of `many_many` relationships, you can do so using the ["through" setting](/developer_guides/model/relations/#many-many-through) on a `many_many` definition. This setting allows you to specify a custom `DataObject` through which to map the `many_many` relation. As such, it is possible to version your `many_many` data by versioning a "through" `DataObject`. For example:

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class Product extends DataObject
{
    private static $db = [
        'Title' => 'Varchar(100)',
        'Price' => 'Currency',
    ];

    private static $many_many = [
        'Categories' => [
            'through' => 'ProductCategory',
            'from' => 'Product',
            'to' => 'Category',
        ],
    ];
}
```

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;
use SilverStripe\Versioned\Versioned;

class ProductCategory extends DataObject
{
    private static $db = [
        'SortOrder' => 'Int',
    ];

    private static $has_one = [
        'Product' => Product::class,
        'Category' => Category::class,
    ];

    private static $extensions = [
        Versioned::class,
    ];
}
```

### Controlling permissions to versioned `DataObject` models {#permissions}

By default, `Versioned` will come out of the box with security extensions which restrict the visibility of objects in Draft ("Stage") or Archive viewing mode.

> [!CAUTION]
> As is standard practice, user code should always invoke `canView()` on any object before
> rendering it. DataLists do not filter on `canView()` automatically, so this must be
> done via user code. This can be achieved either by wrapping `<% if $canView %>;` in
> your template, or by implementing your visibility check in PHP.

#### Version specific *can* methods {#permission-methods}

Versioned DataObjects get additional permission check methods to verify what operation a `Member` is allowed to perform:

- [`canPublish()`](api:SilverStripe\Versioned\Versioned::canPublish()): Determines if a given `Member` is allowed to publish the record
- [`canUnpublish()`](api:SilverStripe\Versioned\Versioned::canUnpublish()) Determines if a given `Member` is allowed to unpublish the record
- [`canViewStage()`](api:SilverStripe\Versioned\Versioned::canViewStage()) Determines if a given `Member` can view the latest version of this record on a specific stage. Beware that this is *not* invoked when calling `canView()`. If you want to affect the result of regular `canView()` checks, implement `canViewVersioned()` instead.
- [`canViewVersioned()`](api:SilverStripe\Versioned\Versioned::canViewVersioned()) Provides additional can view checks for versioned records. This is called by `canView()` and should not be called directly.

The existing [`canDelete()`](api:SilverStripe\Versioned\Versioned::canDelete()) method is used to check if a given `Member` is allowed to archive the record. The `Versioned` extension enhances this permission check for published content to ensure members that cannot unpublish content also cannot archive it.

These methods accept an optional `Member` argument. If not provided, they will assume you want to check the permission against the current `Member`. When performing a version operation on behalf of a `Member`, you'll probably want to use these methods to confirm they are authorised.

> [!WARNING]
> Like with the base `can` permission checks, these checks are *not* performed automatically when invoking the associated action via PHP. i.e. if you call `publishSingle()` on a record in your own code, Silverstripe CMS will *not* check if the currently authenticated user has permission to publish the record. Make sure you are performing permission checks by calling these `can` methods before invoking the associated actions.

```php
$record = MyRecord::get()->byID(99);
if ($record->canPublish()) {
    $record->publishRecursive();
}
```

The `canViewStage()` method can be used to check if a Member can view a specific stage of a record. It should be invoked by user code to check if a record is visible in the given stage.

```php
use SilverStripe\Versioned\Versioned;

// Check if `$member` can view the Live version of $record.
$record->canViewStage(Versioned::LIVE, $member);

// Check if `$member` can view the "Stage" version of $record.
$record->canViewStage(Versioned::DRAFT, $member);

// Both parameters are optional. This is equivalent to calling the method with Versioned::LIVE and
// Security::getCurrentUser();
$record->canViewStage();
```

For the `can` methods that all `DataObject` models have, see [Model-Level Permissions](permissions).

#### Customising permissions for a versioned `DataObject`

`Versioned` record visibility can be customised in one of the following ways by editing your code:

- Override the `canViewVersioned()` method in your `DataObject` subclass. Make sure that this returns `true`, or
   `false` if the user is not allowed to view this object in the current viewing mode.
- Override the `canView()` method to override the method visibility completely, regardless of what stage is being viewed.

E.g.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;
use SilverStripe\Security\Permission;
use SilverStripe\Versioned\Versioned;

class MyObject extends DataObject
{
    private static $extensions = [
        Versioned::class,
    ];

    public function canViewVersioned($member = null)
    {
        // Check if site is live
        $mode = $this->getSourceQueryParam('Versioned.mode');
        $stage = $this->getSourceQueryParam('Versioned.stage');
        if ($mode === 'stage' && $stage === Versioned::LIVE) {
            return true;
        }

        // Only admins can view non-live records
        return Permission::checkMember($member, 'ADMIN');
    }
}
```

If you want to control permissions of an object in an extension, you can also implement
one of the below extension hook methods in your `Extension` subclass:

- `canView()` to update the record's `canView` permissions
- `canViewNonLive()` to update the visibility of this object only in non-live mode specifically.

Note that unlike `canViewVersioned()`, the `canViewNonLive()` method will
only be invoked if the object is in a non-published state.

E.g.

```php
namespace App\Extension;

use SilverStripe\Core\Extension;
use SilverStripe\Security\Permission;

class MyObjectExtension extends Extension
{
    protected function canViewNonLive($member = null)
    {
        if (!Permission::check($member, 'DRAFT_STATUS')) {
            return false;
        }

        // Defer to the result of the main canViewVersioned() permission checks
        return null;
    }
}
```

If none of the above checks are overridden, visibility will be determined by the
permissions in the `non_live_permissions` configuration on the target model class.

E.g.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;
use SilverStripe\Versioned\Versioned;

class MyObject extends DataObject
{
    private static $extensions = [
        Versioned::class,
    ];

    private static $non_live_permissions = [
        'ADMIN',
    ];
}
```

Versioned applies no additional permissions to `canEdit` or `canCreate`, and such
these permissions should be implemented as per standard unversioned DataObjects.

### Defining ownership between related versioned `DataObject` models

You can use the `owns` configuration property on a `DataObject` to specify which relationships are ownership relationships. The `owns` property should be defined on the *owner* `DataObject`.

For example, let's say you have a `MyPage` page type that displays banners containing an image. Each `MyPage` owns many `Banners`, which in turn owns an `Image`.

```php
namespace App\PageType;

use App\Model\Banner;
use Page;

class MyPage extends Page
{
    private static $has_many = [
        'Banners' => Banner::class,
    ];

    private static $owns = [
        'Banners',
    ];
}
```

```php
namespace App\Model;

use App\PageType\MyPage;
use SilverStripe\Assets\Image;
use SilverStripe\ORM\DataObject;
use SilverStripe\Versioned\Versioned;

class Banner extends DataObject
{
    private static $extensions = [
        Versioned::class,
    ];

    private static $has_one = [
        'Parent' => MyPage::class,
        'Image' => Image::class,
    ];

    private static $owns = [
        'Image',
    ];
}
```

If a `MyPage` record gets published, all its related `Banners` will also be published, which will cause all related `Image` records to be published.

Note that this relationship is for publishing specifically, and is not affected by unpublishing or archiving the owner record. To ensure unpublish and archive actions affect owned records, `cascade_deletes` must be used. See [Cascading deletions](relations/#cascading-deletions) for more information about this interaction.

```php
namespace App\PageType;

class MyPage extends Page
{
    private static $has_many = [
        'Banners' => Banner::class,
    ];

    private static $cascade_deletes = [
        'Banners',
    ];
}
```

You must declare both `owns` and `cascade_deletes` if you want all publish, unpublish, and archive actions to carry through.

> [!NOTE]
> Note that ownership cannot be used with polymorphic relations (i.e. `has_one` to non-type specific `DataObject`).

#### Unversioned `DataObject` ownership

Ownership can be used with non-versioned DataObjects, as the necessary functionality is included by default
by the versioned object through the [`RecursivePublishable`](api:SilverStripe\Versioned\RecursivePublishable) extension which is
applied to all objects.

However, it is important to note that even when saving un-versioned objects, it is necessary to explicitly call
`publishRecursive()` to trigger a recursive publish.

The `owns` feature works the same regardless of whether these objects are versioned, so you can use any combination of
versioned or unversioned dataobjects. You only need to call `publishRecursive()` on the specific record for which you are saving changes.

#### `DataObject` ownership with custom relations

In some cases you might need to apply ownership where there is no underlying database relation, such as
those calculated at runtime based on business logic. In cases where you are not backing ownership
with standard relations (`has_one`, `has_many`, etc) it is necessary to declare ownership on both
sides of the relation.

This can be done by creating methods on both sides of your relation (e.g. parent and child class)
that can be used to traverse between each, and then by ensuring you configure both
`owns` config (on the parent) and `owned_by` (on the child).

For example:

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;
use SilverStripe\Versioned\Versioned;

class MyParent extends DataObject
{
    private static $extensions = [
        Versioned::class,
    ];

    private static $owns = [
        'ChildObjects',
    ];

    public function ChildObjects()
    {
        return MyChild::get();
    }
}
```

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;
use SilverStripe\Versioned\Versioned;

class MyChild extends DataObject
{
    private static $extensions = [
        Versioned::class,
    ];

    private static $owned_by = [
        'Parent',
    ];

    public function Parent()
    {
        return MyParent::get()->first();
    }
}
```

#### Image ownership in HTML content via the WYSIWYG editor {#wysiwyg-image-ownership}

If you are using [`DBHTMLText`](api:SilverStripe\ORM\FieldType\DBHTMLText) or [`DBHTMLVarchar`](api:SilverStripe\ORM\FieldType\DBHTMLVarchar) fields in your `DataObject::$db` definitions,
it's likely that your authors can insert images into those fields via the CMS interface.

These images are usually considered to be owned by the `DataObject`, and should be published alongside it.

The ownership relationship is tracked through an `[image]` [shortcode](/developer-guides/extending/shortcodes),
which is automatically transformed into an `<img>` tag at render time. In addition to storing the image path,
the shortcode references the database identifier of the `Image` object and ensures it's published appropriately.

### Controlling how CMS users interact with versioned `DataObject` records

The versioned module includes a [`VersionedGridfieldDetailForm`](api:SilverStripe\Versioned\VersionedGridFieldDetailForm) extension which provides versioning support for DataObjects edited in a [`GridField`](api:SilverStripe\Forms\GridField\GridField). This is applied to [`GridFieldDetailForm`](api:SilverStripe\Forms\GridField\GridFieldDetailForm) by default.

You can disable this on a per-model basis by setting the `versioned_gridfield_extensions` configuration property to false. You can do that in PHP:

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyBanner extends DataObject
{
    private static $versioned_gridfield_extensions = false;
    // ...
}
```

Or via YAML configuration:

```yml
App\Model\MyBanner:
  versioned_gridfield_extensions: false
```

This can also be manually enabled for a single `GridField` by passing the `VersionedGridFieldItemRequest` class name to the [`setItemRequestClass()`](api:SilverStripe\Forms\GridField\GridFieldConfig::setItemRequestClass()) method on a [`GridFieldConfig`](api:SilverStripe\Forms\GridField\GridFieldConfig) instance.

```php
namespace {

    use SilverStripe\CMS\Model\SiteTree;
    use SilverStripe\Forms\FieldList;
    use SilverStripe\Forms\GridField\GridField;
    use SilverStripe\Forms\GridField\GridFieldConfig_RelationEditor;
    use SilverStripe\Forms\GridField\GridFieldDetailForm;
    use SilverStripe\Versioned\VersionedGridFieldItemRequest;

    class Page extends SiteTree
    {
        public function getCMSFields()
        {
            $this->beforeUpdateCMSFields(function (FieldList $fields) {
                $config = GridFieldConfig_RelationEditor::create();
                $config
                    ->getComponentByType(GridFieldDetailForm::class)
                    ->setItemRequestClass(VersionedGridFieldItemRequest::class);
                $gridField = GridField::create('Items', 'Items', $this->Items(), $config);
                $fields->addFieldToTab('Root.Items', $gridField);
            });
            return parent::getCMSFields();
        }
    }
}
```

## Interacting with versioned `DataObject` records

This section deals with specialised operations that can be performed on versioned DataObjects.

### Reading versions by stage

By default, all records are retrieved from the "Stage" (aka draft) stage, which pulls from the same database table you would be using if there were no stages or versions at all.

You can explicitly request a specific stage through various static methods on the `Versioned` class.

> [!TIP]
> Note that in the below examples we just return the `DataList` without executing it. We don't need to execute the query, the reading mode is attached to the `DataList` as soon as it's created via the [`augmentDataQueryCreation()`](api:SilverStripe\Versioned\Versioned::augmentDataQueryCreation()) extension hook implementation.

```php
use SilverStripe\Versioned\Versioned;

// Fetching multiple records
$stageRecords = Versioned::get_by_stage(MyRecord::class, Versioned::DRAFT);
$liveRecords = Versioned::get_by_stage(MyRecord::class, Versioned::LIVE);

// Fetching a single record
$stageRecord = Versioned::get_by_stage(MyRecord::class, Versioned::DRAFT)->byID(99);
$liveRecord = Versioned::get_by_stage(MyRecord::class, Versioned::LIVE)->byID(99);
```

If you already have a list and want to filter it to only include records in a given stage, you can use [`updateListForStage()`](api:SilverStripe\Versioned\Versioned::updateListForStage()) instead.

```php
use SilverStripe\Versioned\Versioned;

$myList = MyRecord::get()->filter(['Name' => 'Example']);
$stageRecords = Versioned::updateListForStage($myList, Versioned::DRAFT);
```

You can also use [`Versioned::withVersionedMode()`](api:SilverStripe\Versioned\Versioned::withVersionedMode()) in conjunction with [`Versioned::set_stage()`](api:SilverStripe\Versioned\Versioned::set_stage()) to temporarily change what stage is being used for queries.

```php
use SilverStripe\Versioned\Versioned;

$liveRecords = Versioned::withVersionedMode(function () {
    // Set the reading mode we want - note we don't have to set it back afterwards,
    // that will be done for us automatically.
    Versioned::set_stage(Versioned::LIVE);
    // Return the result so we can assign it to the `$liveRecords` variable.
    return MyRecord::get();
});
```

You can use `Versioned::set_stage()` outside of calls to `Versioned::withVersionedMode()`, but you *must* remember to set the reading mode back to what is was before you started, or you risk unexpected side effects. The only exception to this is if you explicitly want the rest of the request execution to be performed with a given reading mode (e.g. if a given action on a controller must be explicitly completely executed ina  given stage).

```php
use SilverStripe\Versioned\Versioned;

// Temporarily store the current mode before setting the mode we want to use
$oldMode = Versioned::get_reading_mode();
Versioned::set_stage(Versioned::LIVE);

$liveRecords = MyRecord::get();

// Don't forget to set the mode back afterwards!
Versioned::set_reading_mode($oldMode);
```

> [!NOTE]
> `Versioned::set_stage(Versioned::LIVE)` is the equivalent of `Versioned::set_reading_mode('Stage.' . Versioned::LIVE)`.

### Reading historical versions

The above commands will just retrieve the latest version of its respective stage for you, but not older versions stored
in the `<originalTable>_versions` tables  (see [How versioned DataObjects are tracked in the database](#versions-in-the-database)).

```php
use SilverStripe\Versioned\Versioned;

// the "id" parameter is the ID of the record, and the "version" parameter is the specific version number to fetch
$historicalRecord = Versioned::get_version(MyRecord::class, id: 5, version: 6);
```

The record is retrieved as a regular `DataObject` record with its values set to the values it had when that version was originally saved.

> [!CAUTION]
> Saving modifications via `write()` will create a *new* version, rather than modifying the existing one.

In order to get a list of all versions for a specific record, we get the record version data as specialized [`Versioned_Version`](api:SilverStripe\Versioned\Versioned_Version)
objects, which expose the same database information as a `DataObject`, but also include information about when and how
a record was published.

```php
// stage doesn't matter here
$record = MyRecord::get()->byID(99);
$versions = $record->allVersions();
// instance of Versioned_Version
$version = $versions->First()->Version;
```

### Reading archived versions

Similarly to using `Versioned::get_by_stage()` and `Versioned::updateListForStage()` to get versions of records in a particular stage, you can also fetch records that have been archived.

There are a few different methods for this, depending on exactly what information you're after:

- [`getRemovedOnDraft()`](api:SilverStripe\Versioned\Versioned::getRemovedOnDraft()) - Returns a new list of records (both published and archived) which have been removed from draft.
- [`updateListToIncludeRemovedOnDraft()`](api:SilverStripe\Versioned\Versioned::updateListToIncludeRemovedOnDraft()) - Gives the same results as `getRemovedOnDraft()`, but you can pass it a list to modify instead of it giving you a new list.
- [`getArchivedOnly()`](api:SilverStripe\Versioned\Versioned::getArchivedOnly()) - Returns a new list of only records which have been archived. This excludes records which are published but removed from draft.
- [`updateListToIncludeArchivedOnly()`](api:SilverStripe\Versioned\Versioned::updateListToIncludeArchivedOnly()) - Gives the same results as `getArchivedOnly()`, but you can pass it a list to modify instead of it giving you a new list.

```php
use SilverStripe\Versioned\Versioned;

// Fetching a new list
$archivedAndOnlyLive = Versioned::getRemovedOnDraft(MyRecord::class);
$archivedOnly = Versioned::getArchivedOnly(MyRecord::class);

// Using an existing list
$myList = MyRecord::get()->filter(['Name' => 'Example']);
$archivedAndOnlyLive = Versioned::updateListToIncludeRemovedOnDraft($myList);
$archivedOnly = Versioned::updateListToIncludeArchivedOnly($myList);
```

### Writing changes to a versioned `DataObject`

When you call the `write()` method on a versioned `DataObject` record, this will transparently create a new version of the record in the "Stage" stage.

To write your changes without creating new version, call [`writeWithoutVersion()`](api:SilverStripe\Versioned\Versioned::writeWithoutVersion()) instead.

```php
// This will retrieve the latest draft version of record ID 99.
$record = MyRecord::get()->byID(99);
// This will output the version ID. Let's assume it's 13.
echo $record->Version;


$record->Title = "Foo Bar";
// This will create a new version of record ID 99.
$record->write();
// Will output 14 (because a new version was created).
echo $record->Version;

$record->Title = "FOO BAR";
// This will edit the latest version of record ID 99.
$record->writeWithoutVersion();
// Will still output 14 (because we edited the existing version).
echo $record->Version;
```

An "unpublish" operation *removes* that record from the "Live" stage.

### Publishing a versioned `DataObject`

There's two main methods used to publish a versioned `DataObject` record:

- [`publishSingle()`](api:SilverStripe\Versioned\Versioned::writeWithoutVersion()) publishes *only* this record to live from the draft
- [`publishRecursive()`](api:SilverStripe\Versioned\RecursivePublishable::publishRecursive()) publishes this record and any dependent objects this record may refer to.

In most regular cases, you'll want to use `publishRecursive()`.

`publishRecursive()` can be called on unversioned `DataObject` as well, since the `RecursivePublishable` extension is applied to `DataObject` by default.

```php
$record = MyRecord::get()->byID(99);
$record->MyField = 'changed';

// Will create a new revision in "Stage". Editors will be able to see this revision,
// but unauthenticated visitors to the website will not see it.
$record->write();

// This will publish the changes so they are visible publicly.
$record->publishRecursive();
```

### Unpublishing and archiving a versioned `DataObject`

Archiving and unpublishing are similar operations, both will prevent a versioned DataObject from being publicly accessible. Archiving will also remove the record from the "Stage" stage; other ORMs may refer to this concept as *soft-deletion*.

Both of these operations create a new entry in the relevant `_Versions` table with the `WasDeleted` column set to `1` (see [How versioned DataObjects are tracked in the database](#versions-in-the-database)).

Call [`doUnpublish()`](api:SilverStripe\Versioned\Versioned::doUnpublish()) to unpublish an item. Either call [`doArchive()`](api:SilverStripe\Versioned\Versioned::doArchive()) or simply call `delete()` to archive an item. The Silverstripe ORM doesn't allow you to *hard-delete* versioned DataObjects. Instead they are simply removed from all stages, but all version history is retained. This allowed you to restore archived records later on, if you want to.

```php
$record = MyRecord::get()->byID(99);

// Visitors to the site won't be able to see this record anymore, but editors can
// still edit it and re-publish it.
$record->doUnpublish();


// Editors won't be able to see this record anymore, but its version history will
// still be in the database and may be restored.
$record->delete();
// or
$record->doArchive();
```

Note that `doUnpublish()` and `doArchive()` do not work recursively. If you wish to unpublish or archive dependants records, you have to do it manually.

### Rolling back to an older version

Rolling back allows you to return a record to a previous state. You can rollback a single record using the [`rollbackSingle()`](api:SilverStripe\Versioned\Versioned::rollbackSingle()) method. You can also rollback all dependent records using the [`rollbackRecursive()`](api:SilverStripe\Versioned\Versioned::rollbackRecursive()) method.

Both `rollbackSingle()` and `rollbackRecursive()` expect a single argument, which may be a specific version ID or a stage name.

```php
use SilverStripe\Versioned\Versioned;

$record = MyRecord::get()->byID(99);

// This will take the current live version of a record - and all it's associated (owned) records - and copy it to the
// "Stage" stage. This is equivalent to dismissing any draft work and reverting to what was last published.
$record->rollbackRecursive(Versioned::LIVE);

// This will restore a specific version of the record to "Stage" without affecting any owned records.
$versionToRestore = 10;
$record->rollbackSingle($versionToRestore);

// The live version of the record won't be affected unless you publish the record again.
$record->publishRecursive();
```

Note that internally, rolling back a record creates a new version identical to the restored version. For example,
if the live version of `$record` is #10 and the staged version is #13, rolling back to live will create a version #14 in "Stage" that is identical to version #10.

### Restoring an archived version

Archived records can still be retrieved using `get_including_deleted()`. This will include archived as well as current records. You can use the `isArchived()` method to determine if a record is archived or not. Calling the `write()` method on an archived record will restore it to the "Stage" stage.

```php
use App\Model\MyRecord;
use SilverStripe\Versioned\Versioned;

// This script will restore all archived entries for MyRecord.
$allMyRecords = Versioned::get_including_deleted(MyRecord::class);
foreach ($allMyRecords as $myRecord) {
    if ($myRecord->isArchived()) {
        $myRecord->write();
    }
}
```

If you already know a specific record was archived and want to restor it, you can also use the `rollbackRecursive()` and `rollbackSingle()` methods - but you still have to get a hold of the archived record using `get_including_deleted()` first.

## Interacting with `ChangeSet`

This section explains how you can interact with ChangeSets.

### Adding and removing `DataObject` records to a change set

- `$myChangeSet->addObject(DataObject $record)`: Add a record and all of its owned records to the changeset (`canEdit()` dependent).
- `$myChangeSet->removeObject(DataObject $record)`: Removes a record and all of its owned records from the changeset (`canEdit()` dependent).

### Performing actions on the `ChangeSet` object

- `$myChangeSet->publish()`: Publishes all items in the changeset that have modifications, along with all their owned records (`canPublish()` dependent). Closes the changeset on completion.
- `$myChangeSet->sync()`: Find all owned records with modifications for each item in the changeset, and include them implicitly.
- `$myChangeSet->validate()`: Ensure all owned records with modifications for each item in the changeset are included. This method should not need to be invoked if `sync()` is being used on each mutation to the changeset.

### Getting information about the state of the `ChangeSet`

ChangeSets can exists in three different states:

- `open` No action has been taken on the ChangeSet. Resolves to `publishing` or `reverting`.
- `published`: The ChangeSet has published changes to all of its items and its now closed.
- `reverted`: The ChangeSet has reverted changes to all of its items and its now closed. (Future API, not supported yet)

### Getting information about items in a `ChangeSet`

Each item in the ChangeSet stores `VersionBefore` and `VersionAfter` fields. As such, they can compute the type of change they are adding to their parent ChangeSet. Change types include:

- `created`: This ChangeSet item is for a record that does not yet exist
- `modified`: This ChangeSet item is for a record that differs from what is on the live stage
- `deleted`: This ChangeSet item will no longer exist when the ChangeSet is published
- `none`: This ChangeSet item is exactly as it is on the live stage

## Advanced versioning topics

These topics are targeted towards more advanced use cases that might require developers to extend the behavior of versioning.

### How versioned `DataObject` records are tracked in the database {#versions-in-the-database}

Depending on whether staging is enabled, one or more new tables will be created for your records. `<originalTable>_Versions`
is always created to track historic versions for your model. If staging is enabled this will also create a new
`<originalTable>_Live` table once you've rebuilt the database.

> [!WARNING]
> Note that the "Stage" stage doesn't get its own table - instead, the original table represents the "Stage" stage.

- `MyRecord` table: Contains "Stage" (draft) data
- `MyRecord_Live` table: Contains "Live" (published) data
- `MyRecord_Versions` table: Contains a version history (new row created on each save, publish, unpublish, archive, and rollback event)

Similarly, any subclass you create of a versioned `DataObject` will trigger the creation of additional tables, which are
automatically joined as required:

- `MyRecordSubclass` table: Contains only "Stage" (draft) data for subclass columns
- `MyRecordSubclass_Live` table: Contains only "Live" (published) data for subclass columns
- `MyRecordSubclass_Versions` table: Contains only version history for subclass columns

### Writing custom queries to retrieve versioned `DataObject`

We generally discourage writing `Versioned` queries from scratch, due to the complexities involved through joining
multiple tables across an inherited table scheme (see [Versioned::augmentSQL()](api:SilverStripe\Versioned\Versioned::augmentSQL())). If possible, try to stick to smaller modifications of the generated `DataList` objects.

Example: Get the first 10 live records, filtered by creation date:

```php
use SilverStripe\Versioned\Versioned;
$records = Versioned::get_by_stage(MyRecord::class, Versioned::LIVE)->limit(10)->sort('Created', 'ASC');
```

### Controlling what stage is displayed in the front end

The current stage for each request is determined by [`VersionedHTTPMiddleware`](api:SilverStripe\Versioned\VersionedHTTPMiddleware) before any controllers initialize, through
[`Versioned::choose_site_stage()`](api:SilverStripe\Versioned\Versioned::choose_site_stage()). It checks for a `stage` GET parameter, so you can force a "Stage" (draft) stage by appending
`?stage=Stage` to your request.

The current stage setting is not "sticky" in the session.
Any links presented on the view produced with `?stage=Stage` need to have the same GET parameters in order
to retain the stage. If you are using the `SiteTree->Link()` and `Controller->Link()` methods,
this is automatically the case for `DataObject` links, controller links and form actions.
Note that this behaviour applies for unversioned objects as well, since the views
these are presented in might still contain dependent objects that are versioned.

You can opt for a session base stage setting through the `Versioned.use_session` configuration property.

> [!WARNING]
> Settin `Versioned.use_session` can lead to leaking unpublished information, e.g. if a live URL is viewed in draft mode,
> and the result is cached due to aggressive cache settings (not varying on cookie values).

```php
// app/src/Model/MyObject.php
namespace App\Model;

use App\Control\MyObjectController;
use SilverStripe\Control\Controller;
use SilverStripe\Core\Injector\Injector;
use SilverStripe\ORM\DataObject;
use SilverStripe\Versioned\Versioned;

class MyObject extends DataObject
{
    private static $extensions = [
        Versioned::class,
    ];
    // ...

    public function Link()
    {
        return $this->getController()->Link($this->ID);
    }

    public function CustomLink()
    {
        $link = Controller::join_links('custom-route', $this->ID, '?rand=' . rand());
        // Calls VersionedStateExtension->updateLink() which ensures the correct stage is included if necessary
        // updates $link by reference
        $this->extend('updateLink', $link);
        return $link;
    }

    public function LiveLink()
    {
        // Force live link even when current view is in draft mode
        return Controller::join_links($this->getController()->Link($this->ID), '?stage=Live');
    }

    public function getController()
    {
        return Injector::inst()->get(MyObjectController::class);
    }
}
```

```php
// app/src/Control/MyObjectController.php
namespace App\Control;

use App\Model\MyObject;
use SilverStripe\Control\Controller;
use SilverStripe\Control\HTTPRequest;

class MyObjectController extends Controller
{
    private static $url_segment = 'my-objects';

    public function index(HTTPRequest $request)
    {
        $obj = MyObject::get()->byID($request->param('ID'));
        if (!$obj) {
            return $this->httpError(404);
        }

        // Construct view
        $html = sprintf('<a href="%s">%s</a>', $obj->Link(), $obj->ID);

        return $html;
    }

    public function Link($action = null)
    {
        // Construct link with graceful handling of GET parameters
        $link = Controller::join_links('my-objects', $action);

        // Allow Versioned and other extension to update $link by reference.
        // Calls VersionedStateExtension->updateLink() which ensures the correct stage is included if necessary
        $this->extend('updateLink', $link, $action);

        return $link;
    }
}
```

```yml
# app/_config/routes.yml
SilverStripe\Control\Director:
  rules:
    'my-objects/$ID': 'App\Control\MyObjectController'
```

> [!CAUTION]
> The `choose_site_stage()` call only deals with setting the default stage, and doesn't check if the user is
> authenticated to view it. As with any other controller logic, please use `DataObject->canView()` to determine
> permissions, and avoid exposing unpublished content to your users.

#### Templates variables

In templates, you don't need to worry about this distinction. The `$Content` variable contains the published content by
default, and previews draft content only if explicitly requested (e.g. by the "preview" feature in the CMS, or by adding ?stage=Stage to the URL). If you want
to force a specific stage, we recommend the `Controller->init()` method for this purpose, for example:

```php
// app/src/Control/MyController.php
namespace App\Control;

use SilverStripe\Control\Controller;

class MyController extends Controller
{
    // ...

    public function init()
    {
        parent::init();
        Versioned::set_stage(Versioned::DRAFT);
    }
}
```

### Low level write and publication methods

Silverstripe CMS will usually call these low level methods for you. However if you have specialised needs, you may call them directly.

To move a saved version from one stage to another, call [`writeToStage()`](api:SilverStripe\Versioned\Versioned::writeToStage()) on the object, passing in the stage you want to write to. This is used internally to publish DataObjects.

[`copyVersionToStage()`](api:SilverStripe\Versioned\Versioned::copyVersionToStage()) allows you to copy a specific version to a specific stage. This is used internally when performing a rollback, copying whichever version you're rolling back to into the "Stage" stage.

The current stage is stored as global state on the `Versioned` object. It is usually modified by controllers, e.g. when a preview is initialized. But it can also be set and reset temporarily to force a specific operation to run on a certain stage.

```php
// save current mode
$origMode = Versioned::get_reading_mode();
// returns 'Live' records
$obj = MyRecord::getComplexObjectRetrieval();
// temporarily overwrite mode
Versioned::set_reading_mode(Versioned::DRAFT);
// returns 'Stage' records
$obj = MyRecord::getComplexObjectRetrieval();
// reset current mode
Versioned::set_reading_mode($origMode);
```

See [Reading versions by stage](#reading-versions-by-stage) for more about using reading modes.

## Using the history viewer

You can add the [`HistoryViewerField`](api:SilverStripe\VersionedAdmin\Forms\HistoryViewerField) to the edit form of any [`DataObject`](api:SilverStripe\ORM\DataObject) with the [`Versioned`](api:SilverStripe\Versioned\Versioned) extension. This will allow CMS users revert to a previous version of the record.

```php
// app/src/Models/MyDataObject.php
namespace App\Models;

use SilverStripe\ORM\DataObject;
use SilverStripe\Versioned\Versioned;
use SilverStripe\VersionedAdmin\Forms\HistoryViewerField;

class MyDataObject extends DataObject
{
    // ...

    private static $extensions = [
        Versioned::class,
    ];

    public function getCMSFields()
    {
        $fields = parent::getCMSFields();
        $fields->addFieldToTab(
            'Root.History',
            HistoryViewerField::create('HistoryViewer')
        );
        return $fields;
    }
}
```

## API documentation

- [Versioned](api:SilverStripe\Versioned\Versioned)
- [HistoryViewerField](api:SilverStripe\VersionedAdmin\Forms\HistoryViewerField)
