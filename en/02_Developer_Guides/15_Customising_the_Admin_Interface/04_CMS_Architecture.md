---
title: CMS Architecture
summary: An overview of the code architecture of the CMS
icon: sitemap
---

# CMS architecture

## Introduction

A lot can be achieved in Silverstripe CMS by adding properties and form fields
to your own page types (via [SiteTree::getCMSFields()](api:SilverStripe\CMS\Model\SiteTree::getCMSFields())), as well as creating
your own data management interfaces through [ModelAdmin](api:SilverStripe\Admin\ModelAdmin). But sometimes
you'll want to go deeper and tailor the underlying interface to your needs as well.
For example, to build a personalized CMS dashboard, or content "slots" where authors
can drag their content into. At its core, Silverstripe CMS is a web application
built on open standards and common libraries, so lots of the techniques should
feel familiar to you. This is just a quick run down to get you started
with some special conventions.

For a more practical-oriented approach to CMS customizations, refer to
[Howto: Extend the CMS Interface](/developer_guides/customising_the_admin_interface/how_tos/extend_cms_interface).

## Pattern library

A pattern library is a collection of user interface design elements, this helps developers and designers collaborate and to provide a quick preview of elements as they were intended without the need to build an entire interface to see it.
Components built in React and used by the CMS are actively being added to the pattern library.
The pattern library can be used to preview React components without including them in the Silverstripe CMS.

### Viewing the latest pattern library

The easiest way to access the pattern library is to view it online. The pattern library for the latest Silverstripe CMS development branch is automatically built and deployed. Note that this may include new components that are not yet available in a stable release.

[Browse the Silverstripe CMS pattern library online](https://silverstripe.github.io/silverstripe-pattern-lib/).

### Running the pattern library

If you're developing a new React component, running the pattern library locally is a good way to interact with it.

The pattern library is built from the `silverstripe/admin` module, but it also requires `silverstripe/asset-admin`, and `silverstripe/cms`.

To run the pattern library locally, you'll need a Silverstripe CMS project based on `silverstripe/recipe-cms` and `yarn` installed locally. The pattern library requires the JS source files so you'll need to use the `--prefer-source` flag when installing your dependencies with Composer.

```bash
composer install --prefer-source
(cd vendor/silverstripe/asset-admin && yarn install)
(cd vendor/silverstripe/cms && yarn install)
cd vendor/silverstripe/admin && yarn install && yarn pattern-lib
```

The pattern library will be available at `http://localhost:6006`. The JS source files will be watched, so every time you make a change to a JavaScript file, the pattern library will automatically update itself.

If you want to build a static version of the pattern library, you can replace `yarn pattern-lib` with `yarn build-storybook`. This will output the pattern library files to a `storybook-static` folder.

The Silverstripe CMS pattern library is built using the [StoryBook JS library](https://storybook.js.org/). You can read the StoryBook documentation to learn about more advanced features and customisation options.

## The admin URL

The CMS interface can be accessed by default through the `/admin` URL. You can change this by setting the `$url_base` configuration property for the [`AdminRootController`](api:SilverStripe\Admin\AdminRootController), creating your own [`Director`](api:SilverStripe\Control\Director) routing rule and clearing the old rule as per the example below:

```yml
---
Name: myadmin
After:
  - '#adminroutes'
---
SilverStripe\Control\Director:
  rules:
    'admin': ''
    'newAdmin': 'SilverStripe\Admin\AdminRootController'

SilverStripe\Admin\AdminRootController:
  url_base: 'newAdmin'
---
```

When extending the CMS or creating modules, you can take advantage of various functions that will return the configured admin URL (by default 'admin' is returned):

> [!WARNING]
> Depending on your configuration, the returned value *may or may not* include a trailing slash. The default is to not include one, but you should take care to not
> explicitly expect one scenario or the other.
>
> In PHP you can use [Controller::join_links()](api:SilverStripe\Control\Controller::join_links()) or pass an argument to
> [AdminRootController::admin_url()](api:SilverStripe\Admin\AdminRootController::admin_url()) to ensure only one `/` character separates the admin URL from the rest of
> your path.
>
> In JavaScript, if you are using [@silverstripe/webpack-config](https://www.npmjs.com/package/@silverstripe/webpack-config), you can use the `joinUrlPaths()` utility
> function.

In PHP you should use:

```php
use SilverStripe\Admin\AdminRootController;

AdminRootController::admin_url()
// This method can take an argument:
AdminRootController::admin_url('more/path/here')
```

When writing templates use:

```ss
$AdminURL
<%-- This is actually a method that can take an argument: --%>
$AdminURL('more/path/here')
```

And in JavaScript, this is available through the `ss` namespace as `ss.config.adminUrl`

```js
// You can use this if you use @silverstripe/webpack-config
import { joinUrlPaths } from 'lib/urls';

joinUrlPaths(ss.config.adminUrl, 'more/path/here');
```

### Multiple admin URL and overrides

You can also create your own classes that extend the [`AdminRootController`](api:SilverStripe\Admin\AdminRootController) to create multiple or custom admin areas, with a `Director.rules` for each one.

## Templates and controllers

The base controller for the CMS is [`AdminController`](api:SilverStripe\Admin\AdminController). This is a simple controller that checks users have the relevant permissions before initialising. `AdminRootController` provides an appropriate routing rule for all non-abstract subclasses of `AdminController`.

The CMS backend UI is primarily handled through the [`LeftAndMain`](api:SilverStripe\Admin\LeftAndMain) controller class,
which contains base functionality like displaying and saving a record.
This is extended through various subclasses, e.g. to add a group hierarchy ([`SecurityAdmin`](api:SilverStripe\Admin\SecurityAdmin)),
a search interface ([`ModelAdmin`](api:SilverStripe\Admin\ModelAdmin)) or an "Add Page" form ([`CMSPageAddController`](api:SilverStripe\CMS\Controllers\CMSPageAddController)).

The controller structure is too complex to document here, a good starting point
for following the execution path in code are [`LeftAndMain::getRecord()`](api:SilverStripe\Admin\LeftAndMain::getRecord()) and [`LeftAndMain::getEditForm()`](api:SilverStripe\Admin\LeftAndMain::getEditForm()).
If you have the `cms` module installed, have a look at [`CMSMain::getEditForm()`](api:SilverStripe\CMS\Controllers\CMSMain::getEditForm()) for a good
example on how to extend the base functionality (e.g. by adding page versioning hints to the form).

CMS templates are inherited based on their controllers, similar to subclasses of
the common `Page` object (a new PHP class `MyPage` will look for a `MyPage` template).
We can use this to create a different base template called `LeftAndMain`
(which corresponds to the `LeftAndMain` PHP controller class).
In case you want to retain the main CMS structure (which is recommended),
just create your own "Content" template (e.g. `MyCMSController_Content`),
which is in charge of rendering the main content area apart from the CMS menu.

Depending on the complexity of your layout, you'll also need to override the
"EditForm" template (e.g. `MyCMSController_EditForm`), e.g. to implement
a tabbed form which only scrolls the main tab areas, while keeping the buttons at the bottom of the frame.
This requires manual assignment of the template to your form instance, see [`CMSMain::getEditForm()`](api:SilverStripe\CMS\Controllers\CMSMain::getEditForm()) for details.

Often its useful to have a "tools" panel in between the menu and your content,
usually occupied by a search form or navigational helper.
In this case, you can either override the full base template as described above.
To avoid duplicating all this template code, you can also use the special [`LeftAndMain::Tools()`](api:SilverStripe\Admin\LeftAndMain::Tools()) and
[`LeftAndMain::EditFormTools()`](api:SilverStripe\Admin\LeftAndMain::EditFormTools()) methods available in `LeftAndMain`.
These placeholders are populated by auto-detected templates,
with the naming convention of `<controller classname>_Tools` and `<controller classname>_EditFormTools`.
So to add or "subclass" a tools panel, simply create this file and it's automatically picked up.

## Layout and panels

The various panels and UI components within them are loosely coupled to the layout engine through the `data-layout-type`
attribute. The layout is triggered on the top element and cascades into children, with a `redraw` method defined on
each panel and UI component that needs to update itself as a result of layouting.

Refer to [Layout reference](/developer_guides/customising_the_admin_interface/cms_layout) for further information.

## Forms

Silverstripe CMS constructs forms and its fields within PHP,
mainly through the [getCMSFields()](api:SilverStripe\ORM\DataObject::getCMSFields()) method.
This in turn means that the CMS loads these forms as HTML via Ajax calls, e.g.
after saving a record (which requires a form refresh), or switching the section in the CMS.

Depending on where in the DOM hierarchy you want to use a form,
custom templates and additional CSS classes might be required for correct operation.
For example, the "EditForm" has specific view and logic JavaScript behaviour
which can be enabled via adding the "CMS-edit-form" class.
In order to set the correct layout classes, we also need a custom template.
To obey the inheritance chain, we use `$this->getTemplatesWithSuffix('_EditForm')` for
selecting the most specific template (so a template named `MyAdmin_EditForm`, if it exists).

The form should use a `LeftAndMainFormRequestHandler`, since it allows the use
of a `PjaxResponseNegotiator` to handle its display.

Basic example form in a CMS controller subclass:

```php
namespace App\Admin;

use SilverStripe\Admin\LeftAndMain;
use SilverStripe\Admin\LeftAndMainFormRequestHandler;
use SilverStripe\Forms\FieldList;
use SilverStripe\Forms\FormAction;
use SilverStripe\Forms\Tab;
use SilverStripe\Forms\TabSet;
use SilverStripe\Forms\TextField;

class MyAdmin extends LeftAndMain
{
    public function getEditForm()
    {
        return Form::create(
            $this,
            'EditForm',
            FieldList::create(
                TabSet::create(
                    'Root',
                    Tab::create(
                        'Main',
                        TextField::create('MyText')
                    )
                )->setTemplate('CMSTabset')
            ),
            FieldList::create(
                FormAction::create('doSubmit')
            )
        )
            // Use a custom request handler
            ->setRequestHandler(
                LeftAndMainFormRequestHandler::create($form)
            )
            // JS and CSS use this identifier
            ->setHTMLID('Form_EditForm')
            // Render correct responses on validation errors
            ->setResponseNegotiator($this->getResponseNegotiator());
            // Required for correct CMS layout
            ->addExtraClass('cms-edit-form')
            ->setTemplate($this->getTemplatesWithSuffix('_EditForm'));
    }
}
```

Note: Usually you don't need to worry about these settings,
and will simply call `parent::getEditForm()` to modify an existing,
correctly configured form.

## JavaScript through jQuery.Entwine

> [!WARNING]
> The following documentation regarding Entwine does not apply to React components or sections powered by React.
> If you're developing new functionality in React powered sections please refer to
> [React and Redux](/developer_guides/customising_the_admin_interface/reactjs_and_redux/).

jQuery.entwine is a library
which allows us to attach behaviour to DOM elements in a flexible and structured manner.
See [jQuery Entwine](/developer_guides/customising_the_admin_interface/jquery_entwine) for more information on how to use it.

In the CMS interface, all entwine rules should be placed in the "ss" entwine namespace.
If you want to call methods defined within these rules outside of entwine logic,
you have to use this namespace, e.g. `$('.cms-menu').entwine('ss').collapse()`.

Note that only functionality that is custom to the CMS application needs to be built
in jQuery.entwine, we're trying to reuse library code wherever possible.
The most prominent example of this is the usage of [jQuery UI](https://jqueryui.com) for
dialogs and buttons.

## JavaScript and CSS dependencies via requirements and ajax

The JavaScript logic powering the CMS is divided into many files,
which typically are included via the [Requirements](api:SilverStripe\View\Requirements) class, by adding
them to [LeftAndMain::init()](api:SilverStripe\Admin\LeftAndMain::init()) and its subclassed methods.
This class also takes care of minification and combination of the files,
which is crucial for the CMS performance (see [Requirements::combine_files()](api:SilverStripe\View\Requirements::combine_files())).

Due to the procedural and selector-driven style of UI programming in jQuery.entwine,
it can be difficult to find the piece of code responsible for a certain behaviour.
Therefore it is important to adhere to file naming conventions.
E.g. a feature only applicable to `ModelAdmin` should be placed in
`vendor/silverstripe/framework/admin/javascript/src/legacy/ModelAdmin.js`, while something modifying all forms (including ModelAdmin forms)
would be better suited in `vendor/silverstripe/framework/admin/javascript/src/legacy/LeftAndMain.EditForm.js`.
Selectors used in these files should mirror the "scope" set by its filename,
so don't place a rule applying to all form buttons inside `ModelAdmin.js`.

The CMS relies heavily on Ajax-loading of interfaces, so each interface and the JavaScript
driving it have to assume its underlying DOM structure is appended via an Ajax callback
rather than being available when the browser window first loads.
jQuery.entwine is effectively an advanced version of [jQuery.on](https://api.jquery.com/on/), so takes care of dynamic event binding.

Most interfaces will require their own JavaScript and CSS files, so the Ajax loading has
to ensure they're loaded unless already present. A custom-built library called
`jQuery.ondemand` (located in `vendor/silverstripe/admin/thirdparty`) takes care of this transparently -
so as a developer just declare your dependencies through the [Requirements](api:SilverStripe\View\Requirements) API.

## Client-side routing

Silverstripe CMS uses the HTML5 browser history to modify the URL without a complete
window refresh. We us the below systems in combination to achieve this:

- [Page.js](https://github.com/visionmedia/page.js) routing library is used for most
    CMS sections, which provides additional Silverstripe CMS specific functionality via the
    `vendor/silverstripe/admin/client/src/lib/Router.js` wrapper.
  The router is available on `window.ss.router` and provides the same API as
  described in the
  [Page.js docs](https://github.com/visionmedia/page.js#api).
- [React router](https://github.com/reactjs/react-router) is used for react-powered
    CMS sections. This provides a native react-controlled bootstrapping and route handling
    system that works most effectively with react components. Unlike page.js routes, these
    may be lazy-loaded or registered during the lifetime of the application on the
    `window.ss.routeRegister` wrapper.

### Registering routes

#### `Page.js` (non-react) CMS sections

CMS sections that rely on entwine, page.js, and normal ajax powered content loading mechanisms
(such as modeladmin) will typically have a single wildcard route that initiates the [pjax loading
mechanism](#pjax).

The main place that routes are registered are via the `LeftAndMain::getClientConfig()` overridden method,
which by default registers a single 'URL' route. This will generate a wildcard route handler for each CMS
section in the form `/admin/<section>(/*)?`, which will capture any requests for this section.

Additional routes can be registered like so `window.ss.router('admin/pages', callback)`, however
these must be registered prior to `window.onload`, as they would otherwise be added with lower priority
than the wildcard routes, as page.js prioritises routes in order of registration, not by specificity.
Once registered, routes can we called with `windw.ss.router.show('admin/pages')`.

Route callbacks are invoked with two arguments, `context` and `next`. The [context object](https://github.com/visionmedia/page.js#context)
can be used to pass state between route handlers and inspect the current
history state. The `next` function invokes the next matching route. If `next`
is called when there is no 'next' route, a page refresh will occur.

#### React router CMS sections

Similarly to page.js powered routing, the main point of registration for react routing
sections is the `LeftAndMain::getClientConfig()` overridden method, which controls the main
routing mechanism for this section. However, there are three major differences:

Firstly, where the page.js router uses the full path (e.g. 'admin/pages') as its route,
react-router uses relative paths with nested routes. The main route is, by default, 'admin',
which means the 'admin/pages' path would declare its route as 'pages'. This is added against the
`reactRoutePath` key in the array returned from `LeftAndMain::getClientConfig()`.

Secondly, `reactRouter` must be passed as a boolean flag to indicate that this section is
controlled by the react section, and thus should suppress registration of a page.js route
for this section.

```php
namespace App\Admin;

use SilverStripe\Admin\LeftAndMain;

class MyAdmin extends LeftAndMain
{
    // ...

    public function getClientConfig(): array
    {
        return array_merge(parent::getClientConfig(), [
            'reactRouter' => true,
        ]);
    }
}
```

Lastly, you should ensure that your react CMS section triggers route registration on the client side
with the `reactRouteRegister` component. This will need to be done on the `DOMContentLoaded` event
to ensure routes are registered before window.load is invoked.

```js
import ConfigHelpers from 'lib/Config';
import reactRouteRegister from 'lib/ReactRouteRegister';
import MyAdmin from './MyAdmin';

document.addEventListener('DOMContentLoaded', () => {
  const sectionConfig = ConfigHelpers.getSection('MyAdmin');

  reactRouteRegister.add({
    path: sectionConfig.reactRoutePath,
    component: MyAdminComponent,
    childRoutes: [
      { path: 'form/:id/:view', component: MyAdminComponent },
    ],
  });
});
```

Child routes can be registered post-boot by using `ReactRouteRegister` in the same way.

```js
// Register a nested url under `sectionConfig.reactRoutePath`
const sectionConfig = ConfigHelpers.getSection('MyAdmin');
reactRouteRegister.add({
  path: 'nested',
  component: NestedComponent,
}, [sectionConfig.reactRoutePath]);
```

## PJAX: partial template replacement through ajax {#pjax}

Many user interactions can change more than one area in the CMS.
For example, editing a page title in the CMS form changes it in the page tree
as well as the breadcrumbs. In order to avoid unnecessary processing,
we often want to update these sections independently from their neighbouring content.

In order for this to work, the CMS templates declare certain sections as "PJAX fragments"
through a `data-pjax-fragment` attribute. These names correlate to specific
rendering logic in the PHP controllers, through the [PjaxResponseNegotiator](api:SilverStripe\Control\PjaxResponseNegotiator) class.

Through a custom `X-Pjax` HTTP header, the client can declare which view they're expecting,
through identifiers like `CurrentForm` or `Content` (see [LeftAndMain::getResponseNegotiator()](api:SilverStripe\Admin\LeftAndMain::getResponseNegotiator())).
These identifiers are passed to `loadPanel()` via the `pjax` data option.
The HTTP response is a JSON object literal, with template replacements keyed by their Pjax fragment.
Through PHP callbacks, we ensure that only the required template parts are actually executed and rendered.
When the same URL is loaded without Ajax (and hence without `X-Pjax` headers),
it should behave like a normal full page template, but using the same controller logic.

Example: Create a bare-bones CMS subclass which shows breadcrumbs (a built-in method),
as well as info on the current record. A single link updates both sections independently
in a single Ajax request.

```php
// app/src/Admin/MyAdmin.php
namespace App\Admin;

use SilverStripe\Admin\LeftAndMain;

class MyAdmin extends LeftAndMain
{
    private static $url_segment = 'myadmin';

    public function getResponseNegotiator()
    {
        $negotiator = parent::getResponseNegotiator();
        $controller = $this;
        // Register a new callback
        $negotiator->setCallback('MyRecordInfo', function () use (&$controller) {
            return $controller->getMyRecordInfo();
        });

        return $negotiator;
    }

    public function getMyRecordInfo()
    {
        return $this->renderWith('MyRecordInfo');
    }
}
```

```ss
<%-- MyAdmin.ss --%>
<% include SilverStripe\\Admin\\CMSBreadcrumbs %>
<div>Static content (not affected by update)</div>
<% include MyRecordInfo %>
<a href="$Link" class="cms-panel-link" data-pjax-target="MyRecordInfo,Breadcrumbs">
    Update record info
</a>
```

```ss
<%-- MyRecordInfo.ss --%>
<div data-pjax-fragment="MyRecordInfo">
    Current Record: $currentRecord.Title
</div>
```

A click on the link will cause the following (abbreviated) ajax HTTP request:

```text
GET /admin/myadmin HTTP/1.1
X-Pjax:MyRecordInfo,Breadcrumbs
X-Requested-With:XMLHttpRequest
```

... and result in the following response:

```json
{"MyRecordInfo": "<div...", "CMSBreadcrumbs": "<div..."}
```

Keep in mind that the returned view isn't always decided upon when the Ajax request
is fired, so the server might decide to change it based on its own logic,
sending back different `X-Pjax` headers and content.

On the client, you can set your preference through the `data-pjax-target` attributes
on links or through the `X-Pjax` header. For firing off an Ajax request that is
tracked in the browser history, use the `pjax` attribute on the state data.

```js
// You can use this if you use @silverstripe/webpack-config
import { joinUrlPaths } from 'lib/urls';

$('.cms-container').loadPanel(joinUrlPaths(ss.config.adminUrl, 'pages'), null, { pjax: 'Content' });
```

### Loading lightweight PJAX fragments

Normal navigation between URLs in the admin section of the Framework occurs through `loadPanel` and `submitForm`.
These calls make sure the HTML5 history is updated correctly and back and forward buttons work. They also take
care of some automation, for example restoring currently selected tabs.

However there are situations when you would like to only update a small area in the CMS, and when this operation should
not trigger a browser's history pushState. A good example here is reloading a dropdown that relies on backend session
information that could have been updated as part of action elsewhere, updates to sidebar status, or other areas
unrelated to the main flow.

In this case you can use the `loadFragment` call supplied by `LeftAndMain.js`. You can trigger as many of these in
parallel as you want. This will not disturb the main navigation.

```js
// You can use this if you use @silverstripe/webpack-config
import { joinUrlPaths } from 'lib/urls';

$('.cms-container').loadFragment(joinUrlPaths(ss.config.adminUrl, 'foobar/'), 'Fragment1');
$('.cms-container').loadFragment(joinUrlPaths(ss.config.adminUrl, 'foobar/'), 'Fragment2');
$('.cms-container').loadFragment(joinUrlPaths(ss.config.adminUrl, 'foobar/'), 'Fragment3');
```

The ongoing requests are tracked by the PJAX fragment name (Fragment1, 2, and 3 above) - resubmission will
result in the prior request for this fragment to be aborted. Other parallel requests will continue undisturbed.

You can also load multiple fragments in one request, as long as they are to the same controller (i.e. URL):

```js
// You can use this if you use @silverstripe/webpack-config
import { joinUrlPaths } from 'lib/urls';

$('.cms-container').loadFragment(joinUrlPaths(ss.config.adminUrl, 'foobar/'), 'Fragment2,Fragment3');
```

This counts as a separate request type from the perspective of the request tracking, so will not abort the singular
`Fragment2` nor `Fragment3`.

Upon the receipt of the response, the fragment will be injected into DOM where a matching `data-pjax-fragment` attribute
has been found on an element (this element will get completely replaced). Afterwards a `afterloadfragment` event
will be triggered. In case of a request error a `loadfragmenterror` will be raised and DOM will not be touched.

You can hook up a response handler that obtains all the details of the XHR request via Entwine handler:

```js
$(someSelector).entwine({
  'from .cms-container': {
    onafterloadfragment(event, data) {
      // The value of the status variable here is 'success'
    }
  }
});
```

Alternatively you can use the jQuery deferred API:

```js
// You can use this if you use @silverstripe/webpack-config
import { joinUrlPaths } from 'lib/urls';

$('.cms-container')
  .loadFragment(joinUrlPaths(ss.config.adminUrl, 'foobar/'), 'Fragment1')
  .success((data, status, xhr) => {
    // Say 'success'!
    // eslint-disable-next-line no-alert
    alert(status);
  });
```

## Ajax redirects

Sometimes, a server response represents a new URL state, e.g. when submitting an "add record" form,
the resulting view will be the edit form of the new record. On non-ajax submissions, that's easily
handled through a HTTP redirection. On ajax submissions, browsers handle these redirects
transparently, so the CMS JavaScript doesn't know about them (or the new URL).
To work around this, we're using a custom `X-ControllerURL` HTTP response header
which can declare a new URL. If this header is set, the CMS JavaScript will
push the URL to its history stack, causing the logic to fetch it in a subsequent ajax request.
Note: To avoid double processing, the first response body is usually empty.

## State through HTTP response metadata

By loading mostly HTML responses, we don't have an easy way to communicate
information which can't be directly contained in the produced HTML.
For example, the currently used controller class might've changed due to a "redirect",
which affects the currently active menu entry. We're using HTTP response headers to contain this data
without affecting the response body.

```php
namespace App\Control;

use SilverStripe\Admin\LeftAndMain;

class MyController extends LeftAndMain
{
    // ...

    public function myaction()
    {
        // ...
        $this->getResponse()->addHeader('X-Controller', MyOtherController::class);
        return $html;
    }
}
```

Built-in headers are:

- `X-Title`: Set window title (requires URL encoding)
- `X-Controller`: PHP class name matching a menu entry, which is marked active
- `X-ControllerURL`: Alternative URL to record in the HTML5 browser history
- `X-Status`: Extended status information, used for an information popover (aka "toast" message).
- `X-Reload`: Force a full page reload based on `X-ControllerURL`

## Special links

Some links should do more than load a new page in the browser window.
To avoid repetition, we've written some helpers for various use cases:

- Load into a PJAX panel: `<a href="..." class="cms-panel-link" data-pjax-target="Content">`
- Load URL as an iframe into a popup/dialog: `<a href="..." class="ss-ui-dialog-link">`
- GridField click to redirect to external link: `<a href="..." class="cms-panel-link action external-link">`

## Buttons

Silverstripe CMS automatically applies a [jQuery UI button style](https://jqueryui.com/button/)
to all elements with the class `.ss-ui-button`. We've extended the jQuery UI widget a bit
to support defining icons via HTML5 data attributes (see `ssui.core.js`).
These icon identifiers relate to icon files in `vendor/silverstripe/framework/admin/images/sprites/src/btn-icons`,
and are sprited into a single file through SCSS and [sprity](https://www.npmjs.com/package/sprity)
(sprites are compiled with `yarn run build`). There are classes set up to show the correct sprite via
background images (see `vendor/silverstripe/framework/admin/scss/_sprites.scss`).

Input: `<a href="..." class="ss-ui-button" data-icon="add" />Button text</a>`

Output: `<a href="..." data-icon="add" class="ss-ui-button ss-ui-action-constructive ui-button ui-widget ui-state-default ui-corner-all ui-button-text-icon-primary" role="button"><span class="ui-button-icon-primary ui-icon btn-icon-add"></span><span class="ui-button-text">Button text</span></a>`

Note that you can create buttons from pretty much any element, although
when using an input of type button, submit or reset, support is limited to plain text labels with no icons.

## Menu

The navigation menu in the CMS is created through the [CMSMenu](api:SilverStripe\Admin\CMSMenu) API,
which auto-detects all subclasses of `LeftAndMain`. This means that your custom
`ModelAdmin` subclasses will already appear in there without any explicit definition.
To modify existing menu entries or create new ones, see [CMSMenu::add_menu_item()](api:SilverStripe\Admin\CMSMenu::add_menu_item())
and [CMSMenu::remove_menu_item()](api:SilverStripe\Admin\CMSMenu::remove_menu_item()).

New content panels are typically loaded via Ajax, which might change
the current menu context. For example, a link to edit a file might be clicked
within a page edit form, which should change the currently active menu entry
from "Page" to "Files & Images". To communicate this state change, a controller
response has the option to pass along a special HTTP response header,
which is picked up by the menu:

```php
namespace App\Control;

use SilverStripe\Control\Controller;

class MyController extends Controller
{
    // ...

    public function mycontrollermethod()
    {
        // ... logic here
        $this->getResponse()->addHeader('X-Controller', 'AssetAdmin');
        return 'my response';
    }
}
```

This is usually handled by the existing [LeftAndMain](api:SilverStripe\Admin\LeftAndMain) logic,
so you don't need to worry about it. The same concept applies for
'X-Title' (change the window title) and 'X-ControllerURL' (change the URL recorded in browser history).
Note: You can see any additional HTTP headers through the web developer tools in your browser of choice.

## Tree

The CMS tree for viewing hierarchical structures (mostly pages) is powered
by the [jstree](https://www.jstree.com) library. It is configured through
`client/src/legacy/LeftAndMain.Tree.js` in the `silverstripe/admin` module, as well as some
HTML5 metadata generated on its container (see the `data-hints` attribute).
For more information, see the [Howto: Customise the CMS tree](/developer_guides/customising_the_admin_interface/how_tos/customise_cms_tree).

Note that a similar tree logic is also used for the
form fields to select one or more entries from those hierarchies
([TreeDropdownField](api:SilverStripe\Forms\TreeDropdownField) and [TreeMultiselectField](api:SilverStripe\Forms\TreeMultiselectField)).

## Tabs

We're using [jQuery UI tabs](https://jqueryui.com/tabs/), but in a customised fashion.
HTML with tabs can be created either directly through HTML templates in the CMS,
or indirectly through a [TabSet](api:SilverStripe\Forms\TabSet) form field. Since tabsets are useable
outside of the CMS as well, the baseline application of tabs happens via
a small wrapper around `jQuery.tabs()` stored in `TabSet.js`.

In the CMS however, tabs need to do more: They memorize their active tab
in the user's browser, and lazy load content via ajax once they're activated.

They also need to work across different "layout containers" (see above),
meaning a tab navigation might be in a layout header, while the tab
content is occupied by the main content area. jQuery assumes a common
parent in the DOM for both the tab navigation and its target DOM elements.
In order to achieve this level of flexibility, most tabsets in the CMS
use a custom template which leaves rendering the tab navigation to
a separate template named `CMSMain`. See the "Forms" section above
for an example form.

Here's how you would apply this template to your own tabsets used in the CMS.
Note that you usually only need to apply it to the outermost tabset,
since all others should render with their tab navigation inline.

Form template with custom tab navigation (trimmed down):

```ss
<form $FormAttributes data-layout-type="border">

    <div class="cms-content-header north">
        <% if Fields.hasTabset %>
            <% with Fields.fieldByName('Root') %>
            <div class="cms-content-header-tabs">
                <ul>
                <% loop Tabs %>
                    <li><a href="#$id">$Title</a></li>
                <% end_loop %>
                </ul>
            </div>
            <% end_with %>
        <% end_if %>
    </div>

    <div class="cms-content-fields center">
        <fieldset>
            <% loop Fields %>$FieldHolder<% end_loop %>
        </fieldset>
    </div>

</form>
```

Tabset template without tab navigation (e.g. a template named `CMSTabset`)

```ss
<div $AttributesHTML>
    <% loop Tabs %>
        <% if Tabs %>
            $FieldHolder
        <% else %>
            <div $AttributesHTML>
                <% loop Fields %>
                    $FieldHolder
                <% end_loop %>
            </div>
        <% end_if %>
    <% end_loop %>
</div>
```

Lazy loading works based on the `href` attribute of the tab navigation.
The base behaviour is applied through adding a class `.cms-tabset` to a container.
Assuming that each tab has its own URL which is tracked in the HTML5 history,
the current tab display also has to work when loaded directly without Ajax.
This is achieved by template conditionals (see "MyActiveCondition").
The `.cms-panel-link` class will automatically trigger the ajax loading,
and load the HTML content into the main view. Example:

```ss
<div id="my-tab-id" class="cms-tabset" data-ignore-tab-state="true">
    <ul>
        <li class="<% if MyActiveCondition %> ui-tabs-active<% end_if %>">
            <a href="$AdminURL('mytabs/tab1')" class="cms-panel-link">
                Tab1
            </a>
        </li>
        <li class="<% if MyActiveCondition %> ui-tabs-active<% end_if %>">
            <a href="$AdminURL('mytabs/tab2')" class="cms-panel-link">
                Tab2
            </a>
        </li>
    </ul>
</div>
```

The URL endpoints `$AdminURL('mytabs/tab1')` and `$AdminURL('mytabs/tab2')`
should return HTML fragments suitable for inserting into the content area,
through the `PjaxResponseNegotiator` class (see above).

### Lazy loading fields on tab activation

When a tab is not lazy loaded via ajax, it might still be necessary to
delay some work (for example when doing HTTP requests) until the tab is activated. This is how, for example, the
[`GridFieldLazyLoader`](api:SilverStripe\Forms\GridField\GridFieldLazyLoader) works.

In order to open up the same kind of features to other fields, a custom event is fired on all nodes with the `lazy-loadable` class inside the activated tab panel.
They will receive a `lazyload` event that can be listened to in the following way (you will have to implement your own logic for "loading" the content):

```js
jQuery('input.myfield.lazy-loadable').entwine({
  // Use onmatch so we apply the event handler as soon as the element enters the DOM
  onmatch(e) {
    // Use the one() function so the lazyload only happens once for this field
    this.one('lazyload', () => {
      // Some init code here
    });
  },
});
```

> [!NOTE]
> The `myfield` CSS class isn't strictly necessary here (nor is the input for that matter) - it's just being used so we have a more specific selector. That way we know our JavaScript code will only trigger for the relevant element, and not for every lazy-loadable element in the DOM.

If you apply the `myfield` and `lazy-loadable` CSS classes to some form field on a tab other than main, then when you swap to the tab containing that field it will trigger the lazyload event for that element.

```php
use SilverStripe\Forms\TextField;
$fields->addFieldToTab('Root.AnyTab', TextField::create('MyField')->addExtraClass('myfield lazy-loadable'));
```

## Related

- [Howto: Extend the CMS Interface](/developer_guides/customising_the_admin_interface/how_tos/extend_cms_interface)
- [Howto: Customise the CMS tree](/developer_guides/customising_the_admin_interface/how_tos/customise_cms_tree)
- [ModelAdmin API](api:SilverStripe\Admin\ModelAdmin)
- [Reference: Layout](/developer_guides/customising_the_admin_interface/cms_layout)
- [Rich Text Editing](/developer_guides/forms/field_types/htmleditorfield)
