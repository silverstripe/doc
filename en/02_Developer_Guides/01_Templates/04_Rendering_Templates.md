---
title: Rendering data to a template
summary: Call and render Silverstripe CMS templates manually.
icon: code
---

# Rendering data to a template

> [!NOTE]
> The template syntax, file extensions, and specifics about which templates are chosen from a set as described on this page are specific to the default [`SSTemplateEngine`](api:SilverStripe\View\SSTemplateEngine) - but many of the concepts here (especially the PHP code) should work with any template engine you choose to use.

Templates do nothing on their own. Rather, they are used to generate markup - most typically they are used to generate HTML markup, using variables from some `ModelData` object.
All of the `<% if %>`, `<% loop %>` and other variables are methods or parameters that are called on the current object in
scope (see [scope](syntax#scope) in the syntax section).

The following will render the given data into a template. Given the template:

```ss
<%-- app/templates/Coach_Message.ss --%>
<strong>$Name</strong> is the $Role on our team.
```

Our application code can render into that view using the [`renderWith()`](api:SilverStripe\Model\ModelData) method provided by `ModelData`. Call this method on any instance of `ModelData` or its subclasses, passing in a template name or an array of templates to render.

> [!IMPORTANT]
> Don't include the `.ss` file extension when referencing templates.

```php
namespace App\Model;

use SilverStripe\Model\ArrayData;
use SilverStripe\ORM\DataObject;

class MyModel extends DataObject
{
    // ...

    public function getRenderedMessage()
    {
        $arrayData = ArrayData::create([
            'Name' => 'John',
            'Role' => 'Head Coach',
        ]);

        // renders "<strong>John</strong> is the Head Coach on our team."
        return $arrayData->renderWith('Coach_Message');
    }
}
```

If you want to render an arbitrary template into the `$Layout` section of a page, you need to render your layout template and pass that as the `Layout` parameter to the Page template.

These examples assume you have moved the `templates/Coach_Message.ss` template file to `templates/Layout/Coach_Message.ss`

> [!WARNING]
> While a lot of the concepts on this page apply for any template engine, the `$Layout` functionality is specific to the default [`SSTemplateEngine`](api:SilverStripe\View\SSTemplateEngine).

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyModel extends DataObject
{
    // ...

    public function getRenderedMessageAsPage()
    {
        $data = [
            'Title' => 'Message from the Head Coach',
        ];

        return $this->customise([
            'Layout' => $this
                        ->customise($data)
                        ->renderWith(['Layout/Coach_Message']),
        ])->renderWith(['Page']);
    }
}
```

In this case it may be better to use an *implicit* `Layout` type template, and rely on template inheritance to figure out which templates to use.

```php
$this->customise($data)->renderWith(['Coach_Message', 'Page']);
```

This will look for a global `templates/Coach_Message.ss` template, and if it doesn't find one, will use `templates/Page.ss` as the main template. Then, when it encounters `$Layout` in that template, it will find and use the `templates/Layout/Coach_Message.ss` file to substitute that variable.

> [!TIP]
> You will often have templates named after specific classes, as discussed in [template types and locations](template_inheritance/#template-types-and-locations). In that case, you can simply use `MyClass::class` syntax here. e.g:
>
> ```php
> use App\Model\Coach;
>
> $this->customise($data)->renderWith([Coach::class . '_Message', Page::class]);
> ```
>
> This will search for the following templates:
>
> - `templates/App/Model/Coach_Message.ss`
> - `templates/Page.ss`
> - `templates/App/Model/Layout/Coach_Message.ss`
> - `templates/Layout/Page.ss`

See [template types and locations](template_inheritance/#template-types-and-locations) for more information.

> [!NOTE]
> Most classes in Silverstripe CMS you want in your template extend `ModelData` and allow you to call `renderWith`. This
> includes [Controller](api:SilverStripe\Control\Controller), [FormField](api:SilverStripe\Forms\FormField) and [DataObject](api:SilverStripe\ORM\DataObject) instances.
>
> ```php
> $controller->renderWith([MyController::class, MyBaseController::class]);
>
> use SilverStripe\Security\Security;
> Security::getCurrentUser()->renderWith('Member_Profile');
> ```

## Advanced use cases

`renderWith()` can be used to override the default template process. For instance, to provide an ajax version of a
template.

```php
namespace App\PageType;

use PageController;
use SilverStripe\Control\Director;

class MyPageController extends PageController
{
    private static $allowed_actions = [
        'iwantmyajax',
    ];

    public function iwantmyajax()
    {
        if (Director::is_ajax()) {
            return $this->renderWith('AjaxTemplate');
        } else {
            return $this->httpError(404);
        }
    }
}
```

> [!TIP]
> `Controller` already has a shortcut for the above scenario. Instead of explicitly calling `renderWith()` above, you can declare a template with the following naming convension: `[modelOrControllerClass]_[action].ss` e.g. `Page_iwantmyajax.ss`, `HomePage_iwantmyajax.ss`, or `PageController_iwantmyajax.ss`.
>
> With a template that follows that naming convention in place, the PHP for the `iwantmyajax()` becomes:
>
> ```php
> public function iwantmyajax()
> {
>     if (!Director::is_ajax()) {
>         return $this->httpError(400);
>     }
>     // will feed $this into $this->prepareResponse(), which will render $this using templates defined
>     // in $this->getViewer()
>     return $this;
> }
> ```
>
> This ultimately uses [`SSViewer::get_templates_by_class()`](api::SilverStripe\View\SSViewer::get_templates_by_class()) to find the templates for the class or its parent classes with the action as a suffix.
>
> If you don't have any logic to add in the action, you can forego implementing a method altogether - all you need is to add the action name in the `$allowed_actions` configuration array and make sure you have an appropriately named template.

## Rendering arbitrary data in templates

While `ModelData` has some methods on it you may find useful for reprensenting complex data, you should be able to use just about anything as a model in a template.

To actually render the data, you can use the `customise()` method to add your arbitrary data on top of an existing model:

```php
namespace App\PageType;

use PageController;
use SilverStripe\Control\Director;

class MyPageController extends PageController
{
    // ...

    public function iwantmyajax()
    {
        if (Director::is_ajax()) {
            return $this->customise([
                'Name' => 'John',
                'Role' => 'Head Coach',
                'Experience' => [
                    [
                        'Title' => 'First Job',
                    ],
                    [
                        'Title' => 'Second Job',
                    ],
                ],
            ])->renderWith('AjaxTemplate');
        } else {
            return $this->httpError(400);
        }
    }
}
```

Or wrap the data in a `ModelData` subclass such as `ArrayList`:

```php
namespace App\PageType;

use PageController;
use SilverStripe\Model\ArrayData;

class MyPageController extends PageController
{
    // ...

    public function getMyRenderedData()
    {
        return ArrayData::create([
            'Name' => 'John',
            'Role' => 'Head Coach',
            'Experience' => [
                [
                    'Title' => 'First Job',
                ],
                [
                    'Title' => 'Second Job',
                ],
            ],
        ])->renderWith('MyTemplate');
    }
}
```

Or you can hand the data to `SSViewer` directly:

```php
namespace App\PageType;

use PageController;
use SilverStripe\View\SSViewer;

class MyPageController extends PageController
{
    // ...

    public function getMyRenderedData()
    {
        return SSViewer::create('MyTemplate')->process([
            'Name' => 'John',
            'Role' => 'Head Coach',
            'Experience' => [
                [
                    'Title' => 'First Job',
                ],
                [
                    'Title' => 'Second Job',
                ],
            ],
        ]);
    }
}
```
