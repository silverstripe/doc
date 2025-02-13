---
title: Preview
summary: How content previews work in the CMS
---

# CMS preview

## Overview

With the addition of side-by-side editing, the preview has the ability to appear
within the CMS window when editing content in the CMS. This is enabled by default
in the *Pages* section for `SiteTree` models, but as outlined below can be enabled
in other sections and for other models as well.

Within the preview panel, the site is rendered into an iframe. It will update
itself whenever the content is saved, and relevant pages will be loaded for editing
when the user navigates around in the preview.

The root element for preview is `.cms-preview` which maintains the internal
states necessary for rendering within the entwine properties. It provides
function calls for transitioning between these states and has the ability to
update the appearance of the option selectors.

In terms of backend support, it relies on [`SilverStripeNavigator`](api:SilverStripe\Admin\Navigator\SilverStripeNavigator) to be rendered
into the form. `LeftAndMain` will automatically take care of generating it as long
as the `*_SilverStripeNavigator` template is found - first segment has to match the
current *LeftAndMain*-derived class (e.g. `LeftAndMain_SilverStripeNavigator`).

## PHP

For a DataObject to be previewed using the preview panel there are a few prerequisites:

- The class must implement (or [have an extension](/developer_guides/extending/extensions/) which implements) the [CMSPreviewable](api:SilverStripe\ORM\CMSPreviewable) interface
- At least one preview state must be enabled for the class
- There must be some valid URL to use inside the preview panel

### `CMSPreviewable`

The `CMSPreviewable` interface has three methods: `PreviewLink`, `getCMSEditLink`, and
`getMimeType`.

#### `PreviewLink`

The `PreviewLink` method is what determines the URL used inside the preview panel. If
your `DataObject` is intended to always belong to a page, you might want to preview the
item in the context of where it sits on the page using an anchor. You can also provide
some route specific for previewing this object, for example an action on the ModelAdmin
that is used to manage the object.

#### `getCMSEditLink`

This method exists so that when a user clicks on a link in the preview panel, the CMS
edit form for the page the link leads to can be loaded. Unless your `DataObject` is
acting like a page this will likely not apply.

You can leave the default implementation from `DataObject` which returns `null` if you want.

If your object belongs to [a custom ModelAdmin](./modeladmin), the edit URL for the
object is predictable enough to construct and return from this method as you'll see below.
The format for that situation is always the same, with increasing complexity if you're
nesting `GridField`s. For the below examples it is assumed you aren't using nested
`GridField`s - though [CMSEditLinkExtension](/developer_guides/model/managing_records#getting-an-edit-link)
will handle those situations for you if you use it.

> [!TIP]
> The easiest way to get a correct return value for `getCMSEditLink()` is by
> [using CMSEditLinkExtension](/developer_guides/model/managing_records#getting-an-edit-link).
> That extension implements `updateCMSLink()` so you don't have to touch the `getCMSEditLink()`
> method directly at all.
>
> ```php
> namespace App\Model;
>
> use App\Admin\MyModelAdmin;
> use SilverStripe\Admin\CMSEditLinkExtension;
> use SilverStripe\ORM\CMSPreviewable;
> use SilverStripe\ORM\DataObject;
>
> class MyParentModel extends DataObject implements CMSPreviewable
> {
>     private static string $cms_edit_owner = MyModelAdmin::class;
>
>     private static $extensions = [
>         CMSEditLinkExtension::class,
>     ];
>
>     // ...
> }
> ```
>
> For completeness, the other examples below show alternative implementations.

#### `GetMimeType`

In ~90% of cases will be 'text/html', but note it is also possible to display (for example)
an inline PDF document in the preview panel.

### Preview states {#preview-states-php}

The preview state(s) you apply to your `DataObject` will depend primarily on whether it uses
the [Versioned](api:SilverStripe\Versioned\Versioned) extension or not.

#### Versioned `DataObject` models

If your class does use the `Versioned` extension, there are two different states available
to you. It is generally recommended that you enable both, so that content authors can toggle
between viewing the draft and the published content.

To enable the draft preview state, use the `$show_stage_link` configuration variable.

```php
namespace App\Model;

use SilverStripe\ORM\CMSPreviewable;
use SilverStripe\ORM\DataObject;

class MyModel extends DataObject implements CMSPreviewable
{
    private static $show_stage_link = true;

    // ...
}
```

To enable the published preview state, use the `$show_live_link` configuration variable.

```php
namespace App\Model;

use SilverStripe\ORM\CMSPreviewable;
use SilverStripe\ORM\DataObject;

class MyModel extends DataObject implements CMSPreviewable
{
    private static $show_live_link = true;

    // ...
}
```

#### Unversioned `DataObject` models

If you are not using the `Versioned` extension for your class, there is only one preview
state you can use. This state will always be active once you enable it.

To enable the unversioned preview state, use the `$show_unversioned_preview_link`
configuration variable.

```php
namespace App\Model;

use SilverStripe\ORM\CMSPreviewable;
use SilverStripe\ORM\DataObject;

class MyModel extends DataObject implements CMSPreviewable
{
    private static $show_unversioned_preview_link = true;

    // ...
}
```

### Enabling preview for `DataObject` records in a `ModelAdmin`

For this example we will take the `Product` and `MyAdmin` classes from the
[ModelAdmin documentation](./modeladmin).

#### The `DataObject` implementation {#modeladmin-dataobject-implementation}

As mentioned above, your `Product` class must implement the `CMSPreviewable` interface.
It also needs at least one preview state enabled. This example assumes we aren't using
the `Versioned` extension.

```php
namespace App\Model;

use SilverStripe\ORM\CMSPreviewable;
use SilverStripe\ORM\DataObject;

class Product extends DataObject implements CMSPreviewable
{
    private static $show_unversioned_preview_link = true;
    // ...

    public function PreviewLink($action = null)
    {
        $link = null;
        $this->extend('updatePreviewLink', $link, $action);
        return $link;
    }

    public function getMimeType()
    {
        return 'text/html';
    }
}
```

We will need to add a new action to the `ModelAdmin` to provide the actual preview itself.
For now, assume that action will be called `cmsPreview`. We can very easily craft a valid
URL using the `Link` method on the `MyAdmin` class.

Note that if you had set up this model to act like a page,
you could simply `return $this->Link($action)`. In that case the new action would not need
to be added to your `ModelAdmin`.

> [!WARNING]
> Note: The `if (!$this->isInDB())` check below is important! Without this, the preview panel will redirect you to a 404 page when creating a new object.

`ModelAdmin` provides methods for generating a link for the correct model:

```php
namespace App\Model;

use App\Admin\MyAdmin;
use SilverStripe\Control\Controller;
use SilverStripe\ORM\CMSPreviewable;
use SilverStripe\ORM\DataObject;

class Product extends DataObject implements CMSPreviewable
{
    // ...

    public function PreviewLink($action = null)
    {
        if (!$this->isInDB()) {
            return null;
        }
        $admin = MyAdmin::singleton();
        $link = Controller::join_links(
            $admin->getLinkForModelClass(static::class),
            'cmsPreview',
            $this->ID
        );
        $this->extend('updatePreviewLink', $link, $action);
        return $link;
    }
}
```

The `getCMSEditLink()` method is also very easy to implement, because the edit link used by
`ModelAdmin` is predictable.

If you aren't using [CMSEditLinkExtension](/developer_guides/model/managing_records#getting-an-edit-link),
you can simply call `getCMSEditLinkForManagedDataObject()` on a singleton of the `ModelAdmin` subclass:

```php
namespace App\Model;

use App\Admin\MyAdmin;
use SilverStripe\ORM\CMSPreviewable;
use SilverStripe\ORM\DataObject;

class Product extends DataObject implements CMSPreviewable
{
    // ...

    public function getCMSEditLink(): ?string
    {
        $admin = MyAdmin::singleton();
        return $admin->getCMSEditLinkForManagedDataObject($this);
    }
}
```

> [!TIP]
> Remember, if you're implementing this [in an extension](/developer_guides/extending/extensions/), you'll
> need to replace any `$this->` with `$this->owner->` to get the values from the actual record.

Let's assume when you display this object on the front end you're just looping through a
list of items and indirectly calling `forTemplate` using the [`$Me` template variable](../templates/syntax#fortemplate).
This method will be used by the `cmsPreview` action in the `MyAdmin` class to tell the
CMS what to display in the preview panel.

#### The `ModelAdmin` implementation

We need to add the `cmsPreview` action to the `MyAdmin` class, which will output the
content which should be displayed in the preview panel.

Because this is a public method called on a `ModelAdmin`, which will often be executed
in a back-end context using admin themes, it pays to ensure we're loading the front-end
themes whilst rendering out the preview content.

> [!TIP]
> If the `ModelAdmin` you want to do this on is in some vendor module, you can apply
> this action in an extension as well! Just remember to use the public methods where
> protected properties are used below (e.g. `$this->urlParams['ID']` would become
> `$this->owner->getUrlParams()['ID']`).

```php
namespace App\Admin;

use App\Model\Product;
use SilverStripe\Admin\ModelAdmin;
use SilverStripe\View\Requirements;
use SilverStripe\View\SSViewer;

class MyAdmin extends ModelAdmin
{
    private static $managed_models = [
        Product::class,
    ];

    private static $url_segment = 'products';

    private static $menu_title = 'Products';

    private static $allowed_actions = [
        'cmsPreview',
    ];

    private static $url_handlers = [
        '$ModelClass/cmsPreview/$ID' => 'cmsPreview',
    ];

    public function cmsPreview()
    {
        $id = $this->urlParams['ID'];
        $obj = $this->modelClass::get_by_id($id);
        if (!$obj || !$obj->exists()) {
            return $this->httpError(404);
        }

        // Include use of a front-end theme temporarily and clear any CMS requirements.
        $oldThemes = SSViewer::get_themes();
        SSViewer::set_themes(SSViewer::config()->get('themes'));
        Requirements::clear();

        // Render the preview content
        // Note that if your template is an include and relies on global css or js, you should
        // use the Requirements API here to include those
        $preview = $obj->forTemplate();

        // Make sure to set back to backend themes and restore CMS requirements.
        SSViewer::set_themes($oldThemes);
        Requirements::restore();

        return $preview;
    }
}
```

> [!TIP]
> If the CSS or JS you have added via [the Requirements API](/developer_guides/templates/requirements/#php-requirements-api)
> aren't coming through, you may need to add `<head>` and `<body>` tags to the markup. It may not be appropriate to do this in
> your main template (you don't want two `<body>` tags on a page that includes the template), so you might need a preview wrapper
> template, like so:
>
> ```ss
> <%-- themes/mytheme/templates/PreviewBase.ss --%>
> <!DOCTYPE html>
> <html>
> <%-- head tag is needed for css to be injected --%>
> <head></head>
> <%-- body tag is needed for javascript to be injected --%>
> <body>
>     <%-- these HTML elements are just here to comply with styling from the default theme, replace them with your own theme markup --%>
>     <main class="container container--page"><div class="page"><div class="page__content>
>         $Preview
>     </div></div></main>
> </body>
> </html>
> ```
>
> ```php
> // app/src/Admin/MyAdmin.php
> namespace App\Admin;
>
> use SilverStripe\Admin\ModelAdmin;
> use SilverStripe\View\Requirements;
> use SilverStripe\View\SSViewer;
>
> class MyAdmin extends ModelAdmin
> {
>     // ...
>
>     public function cmsPreview()
>     {
>         // ... ommitted for brevity
>
>         // Add in global css/js that would normally be added in the page base template (as needed)
>         Requirements::themedCSS('client/dist/css/style.css');
>         // Render the preview content
>         $preview = $obj->forTemplate();
>         // Wrap preview in proper html, body, etc so Requirements are used
>         $preview = SSViewer::create('PreviewBase')->process(['Preview' => $preview]);
>
>         // ... ommitted for brevity
>     }
> }
> ```

### Enabling preview for `DataObject` models which belong to a page

If the `DataObject` you want to preview belongs to a specific page, for example
through a `has_one` or `has_many` relation, you will most likely want to preview
it in the context of the page it belongs to.

#### The page implementation

For this example we will assume the `Product` class is `Versioned`.

As discussed above, the `getCMSEditLink` method is used to load the correct edit form
in the CMS when you click on a link within the preview panel. This uses the
`x-page-id` and `x-cms-edit-link` meta tags in the head of the page (assuming your
page template calls `$MetaTags` in the `<head>` element). When a page loads,
these meta tags are checked and the appropriate form is loaded.

When rendering a full page in the preview panel to preview a `DataObject` on that
page, the meta tags for that page are present. When a content author toggles between
the draft and published preview states, those meta tags are checked and the page's
edit form would be loaded instead of the `DataObject`'s form. To avoid this
unexpected behaviour, you can include an extra GET parameter in the value returned
by `PreviewLink`. Then in the `MetaTags` method, when the extra parameter is
detected, omit the relevant meta tags.

Note that this is not necessary for unversioned `DataObjects` as they only have
one preview state.

```php
namespace App\PageType;

use App\Model\Product;
use Page;
use SilverStripe\Control\Controller;
use SilverStripe\View\Parsers\HTMLValue;

class ProductPage extends Page
{
    // ...

    private static $has_many = [
        'Products' => Product::class,
    ];

    public function MetaTags($includeTitle = true)
    {
        $tags = parent::MetaTags($includeTitle);
        $controller = Controller::curr();
        if ($controller === null) {
            return;
        }
        // If the 'DataObjectPreview' GET parameter is present, remove 'x-page-id' and 'x-cms-edit-link' meta tags.
        // This ensures that toggling between draft/published states doesn't revert the CMS to the page's edit form.
        $request = $controller->getRequest();
        if ($request->getVar('DataObjectPreview') !== null) {
            $html = HTMLValue::create($tags);
            $xpath = "//meta[@name='x-page-id' or @name='x-cms-edit-link']";
            $removeTags = $html->query($xpath);
            $body = $html->getBody();
            foreach ($removeTags as $tag) {
                $body->removeChild($tag);
            }
            $tags = $html->getContent();
        }
        return $tags;
    }
}
```

#### The `DataObject` implementation {#page-dataobject-implementation}

Make sure the Versioned `Product` class implements `CMSPreviewable` and enables
the draft and published preview states.

```php
namespace App\Model;

use App\PageType\ProductPage;
use SilverStripe\ORM\CMSPreviewable;
use SilverStripe\ORM\DataObject;
use SilverStripe\Versioned\Versioned;

class Product extends DataObject implements CMSPreviewable
{
    private static $show_stage_link = true;

    private static $show_live_link = true;

    private static $extensions = [
        Versioned::class,
    ];

    private static $has_one = [
        'ProductPage' => ProductPage::class,
    ];
    // ...

    public function PreviewLink($action = null)
    {
        $link = null;
        $this->extend('updatePreviewLink', $link, $action);
        return $link;
    }

    public function getMimeType()
    {
        return 'text/html';
    }
}
```

Implement a method which gives you a unique repeatable anchor for each
distinct `Product` object.

```php
namespace App\Model;

use SilverStripe\ORM\CMSPreviewable;
// ...

class Product extends DataObject implements CMSPreviewable
{
    // ...

    /**
     * Used to generate the id for the product element in the template.
     */
    public function getAnchor()
    {
        return 'product-' . $this->getUniqueKey();
    }
}
```

For the `PreviewLink`, append the `DataObjectPreview` GET parameter to the
page's frontend URL.

```php
namespace App\Model;

use SilverStripe\ORM\CMSPreviewable;
// ...

class Product extends DataObject implements CMSPreviewable
{
    // ...

    public function PreviewLink($action = null)
    {
        $link = null
        if (!$this->isInDB()) {
            return $link;
        }
        // Let the page know it's being previewed from a DataObject edit form (see Page::MetaTags())
        $action = $action . '?DataObjectPreview=' . mt_rand();

        // Scroll the preview straight to where the object sits on the page.
        $page = $this->ProductPage()
        if ($page && $page->exists()) {
            $link = $page->Link($action) . '#' . $this->getAnchor();
        }

        $this->extend('updatePreviewLink', $link, $action);
        return $link;
    }
}
```

The `getCMSEditLink()` method doesn't matter so much for this implementation, so you can let
it return the default value of `null`.

#### The page template

In your page template, make sure the anchor is used where you render the objects.
This allows the preview panel to be scrolled automatically to where the object
being edited sits on the page.

```ss
<%-- ... --%>
<% loop $Products %>
  <div id="$Anchor">
    <%-- ... --%>
  </div>
<% end_loop %>
```

## JavaScript

### Configuration and defaults

We use `ss.preview` entwine namespace for all preview-related entwines.

Like most of the CMS, the preview UI is powered by
[jQuery entwine](/developer_guides/customising_the_admin_interface/jquery_entwine). This means
its defaults are configured through JavaScript, by setting entwine properties.
In order to achieve this, create a new file `app/javascript/MyLeftAndMain.Preview.js`.

In the following example we configure three aspects:

- Set the default mode from "split view" to a full "edit view"
- Make a wider mobile preview
- Increase minimum space required by preview before auto-hiding

Note how the configuration happens in different entwine namespaces
("ss.preview" and "ss"), as well as applies to different selectors
(".CMS-preview" and ".CMS-container").

```js
jQuery.entwine('ss.preview', ($) => {
  $('.cms-preview').entwine({
    DefaultMode: 'content',
    getSizes() {
      const sizes = this._super();
      sizes.mobile.width = '400px';
      return sizes;
    }
  });
});

jQuery.entwine('ss', ($) => {
  $('.cms-container').entwine({
    getLayoutOptions() {
      const opts = this._super();
      opts.minPreviewWidth = 600;
      return opts;
    }
  });
});
```

Load the file in the CMS via setting adding `app/javascript/MyLeftAndMain.Preview.js`
to the `LeftAndMain.extra_requirements_javascript` [configuration value](../configuration)

```yml
SilverStripe\Admin\LeftAndMain:
  extra_requirements_javascript:
    - app/javascript/MyLeftAndMain.Preview.js
```

In order to find out which configuration values are available, the source code
is your best reference at the moment - have a look in `LeftAndMain.Preview.js`
in the `silverstripe/admin` module.
To understand how layouts are handled in the CMS UI, have a look at the
[CMS Architecture](cms_architecture) guide.

### Enabling preview

The frontend decides on the preview being enabled or disabled based on the
presence of the `.cms-previewable` class. If this class is not found the preview
will remain hidden, and the layout will stay in the *content* mode.

If the class is found, frontend looks for the `SilverStripeNavigator` structure
and moves it to the `.cms-preview-control` panel at the bottom of the preview.
This structure supplies preview options such as state selector.

If the navigator is not found, the preview appears in the GUI, but is shown as
"blocked" - i.e. displaying the "preview unavailable" overlay.

The preview can be affected by calling `enablePreview` and `disablePreview`. You
can check if the preview is active by inspecting the `IsPreviewEnabled` entwine
property.

### Preview states {#preview-states-js}

States are the site stages: *live*, *stage* etc. Preview states are picked up
from the `SilverStripeNavigator`. You can invoke the state change by calling:

```js
$('.cms-preview').entwine('.ss.preview').changeState('StageLink');
```

Note the state names come from [SilverStripeNavigatorItem](api:SilverStripe\Admin\Navigator\SilverStripeNagivatorItem) class names - thus
the `Link` in their names. This call will also redraw the state selector to fit
with the internal state. See `AllowedStates` in `.cms-preview` entwine for the
list of supported states.

You can get the current state by calling:

```js
$('.cms-preview').entwine('.ss.preview').getCurrentStateName();
```

### Preview sizes

This selector defines how the preview iframe is rendered, and try to emulate
different device sizes. The options are hardcoded. The option names map directly
to CSS classes applied to the `.cms-preview` and are as follows:

- *auto*: responsive layout
- *desktop*
- *tablet*
- *mobile*

You can switch between different types of display sizes programmatically, which
has the benefit of redrawing the related selector and maintaining a consistent
internal state:

```js
$('.cms-preview').entwine('.ss.preview').changeSize('auto');
```

You can find out current size by calling:

```js
$('.cms-preview').entwine('.ss.preview').getCurrentSizeName();
```

### Preview modes

Preview modes map to the modes supported by the *threeColumnCompressor* layout
algorithm, see [layout reference](cms_layout) for more details. You
can change modes by calling:

```js
$('.cms-preview').entwine('.ss.preview').changeMode('preview');
```

Currently active mode is stored on the `.cms-container` along with related
internal states of the layout. You can reach it by calling:

```js
$('.cms-container').entwine('.ss').getLayoutOptions().mode;
```

> [!WARNING]
> Caveat: the `.preview-mode-selector` appears twice, once in the preview and
> second time in the CMS actions area as `#preview-mode-dropdown-in-cms`. This is
> done because the user should still have access to the mode selector even if
> preview is not visible. Currently CMS Actions are a separate area to the preview
> option selectors, even if they try to appear as one horizontal bar.

### Preview API

Namespace `ss.preview`, selector `.cms-preview`:

- **getCurrentStateName**: get the name of the current state (e.g. *LiveLink* or *StageLink*).
- **getCurrentSizeName**: get the name of the current device size.
- **getIsPreviewEnabled**: check if the preview is enabled.
- **changeState**: one of the `AllowedStates`.
- **changeSize**: one of *auto*, *desktop*, *tablet*, *mobile*.
- **changeMode**: maps to *threeColumnLayout* modes - *split*, *preview*, *content*.
- **enablePreview**: activate the preview and switch to the *split* mode. Try to load the relevant URL from the content.
- **disablePreview**: deactivate the preview and switch to the *content* mode. Preview will re-enable itself when new
previewable content is loaded.

### Related

- [Reference: Layout](cms_layout)
