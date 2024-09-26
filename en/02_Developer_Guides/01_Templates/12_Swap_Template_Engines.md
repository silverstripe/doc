---
title: Swap Template Engines
summary: Using a third-party template rendering engine with Silverstripe CMS
icon: hdd
---

# Swap template engines

All other sections in the documentation on this website assume you're using the default template rendering engine. That said, Silverstripe CMS does allow you to use other template rendering engines if you want to.

## How to swap out a template engine

Depending on what your end goal is, you can swap out the template engine at a few different points.

### Globally override the default engine via dependency injection {#swap-globally}

This is the lowest effort in terms of setting the engine, but it means *every single template* used in your project (including in the CMS backend) must be in the syntax your template engine understands. If you don't provide copies of all templates in your syntax of choice, things will fail.

```yml
---
After: '#view-config'
---
SilverStripe\Core\Injector\Injector:
  SilverStripe\View\TemplateEngine:
    class: 'MyModule\MyTemplateEngine'
```

> [!WARNING]
> There are a few places in core code that take a string template from configuration and render it using the default template engine. You'll need to find and update the configuration to provide a string template in your chosen syntax.

### Define the template engine for a specific controller or model {#swap-controller-or-model}

#### Controllers {#swap-controller}

If you want all templates rendered by a given controller to use a different template engine, you need to override the [`getTemplateEngine()`](api:SilverStripe\Control\Controller::getTemplateEngine()) method on your controller.

```php
namespace App\Controller;

use MyModule\MyTemplateEngine;
use SilverStripe\Control\Controller;
use SilverStripe\View\TemplateEngine;

class MyController extends Controller
{
    // ...
    protected function getTemplateEngine(): TemplateEngine
    {
        if (!$this->templateEngine) {
            $this->templateEngine = MyTemplateEngine::create();
        }
        return $this->templateEngine;
    }
}
```

When this controller is rendered (e.g. via [`handleAction()`](api:SilverStripe\Control\Controller::handleAction()) or directly calling [`render()`](api:SilverStripe\Control\Controller::handleAction())), your template engine will be used to render it. You must make sure you have an appropriate template available.

Note that if a different controller is invoked (e.g. rendering an elemental area from inside a page template) or a model gets rendered directly (e.g. using `$Me` in a template) the default template engine will be used for that if you haven't changed it there as well.

#### Models {#swap-model}

There's no `getTemplateEngine()` method in [`ModelData`](api:SilverStripe\Model\ModelData). Instead, you'll need to override the [`renderWith()`](api:SilverStripe\Model\ModelData::renderWith()) method like so:

```php
namespace App\Model;

use MyModule\MyTemplateEngine;
use SilverStripe\Model\ModelData;
use SilverStripe\ORM\FieldType\DBHTMLText;
use SilverStripe\View\SSViewer;

class MyModel extends ModelData
{
    // ...
    public function renderWith($template, ModelData|array $customFields = []): DBHTMLText
    {
        // In this case if an SSViewer has been explicitly instantiated, that will be used instead.
        // You can choose to override that as well, if it suits your use case to do so.
        if (!($template instanceof SSViewer)) {
            $template = SSViewer::create($template, MyTemplateEngine::create());
        }
        return parent::renderWith($template, $customFields);
    }
}
```

### For a given instance of `SSViewer` {#swap-ssviewer}

If you're instantiating [`SSViewer`](api:SilverStripe\View\SSViewer) instances directly, you can pass the template engine as the second constructor argument.

```php
use MyModule\MyTemplateEngine;
use SilverStripe\View\SSViewer;

// ...
$viewer = SSViewer::create($templates, MyTemplateEngine::create());
```

## Implementing a template engine

If you want to use a specific third-party template rendering solution in Silverstripe CMS and there's no module for it, you may need to create your own class that implements the [`TemplateEngine`](api:SilverStripe\View\TemplateEngine) interface.

When a template needs to be rendered, usually `SSViewer` will be told which template candidates to choose from and what data to inject into the template. This will be passed through from `SSViewer` to your `TemplateEngine` class, which will need to:

- Determine which actual template file to use
- Render the template, using the given data
- Output the rendered markup as a string

`SSViewer` is then responsible for normalising the markup (e.g. by rewriting anchor links and injecting resources from the Requirements API).

For consistency, your template engine should follow these guidelines:

1. Respect the `SSViewer` cascading themes list and the template priority order.
   This ensures that projects and modules can refer to themes and templates generically, and don't need to know the specifics of your template engine for things to work correctly.
1. Respect [`TemplateGlobalProvider`](api:SilverStripe\View\TemplateGlobalProvider) data.
   Some modules and project code will provide data that should be available in all templates. Ensuring that data is available will mean you don't have to re-implement ways to access that data when you need it.
1. Respect casting as defined in the [casting](/developer_guides/model/data_types_and_casting/#casting) docs.
   This may mean turning off default casting or escaping functionality provided by the third-party template rendering solution. It ensures that the data can be displayed in ways intended by whoever set up the data model.
1. Respect Requirements API.
   As mentioned above, `SSViewer` will inject resources from the Requirements API into your final rendered result for you. These can come from PHP code, but you should ideally provide a way to interact with the Requirements API from your templates as well.
1. Respect the base tag generated with [`SSViewer::getBaseTag()`](api:SilverStripe\View\SSViewer::getBaseTag()).
   As mentioned in the [common variables](/developer_guides/templates/common_variables/#base-tag) section, there are assumptions in the Silverstripe CMS codebase about this base tag being present.
1. Respect the i18n system.
   The localisation system used in Silverstripe CMS is designed to allow you to use the exact same localisation files in PHP code *and* in templates. Ideally your template engine should hook into this same system to avoid duplicating localisation strings.

Note that when `SSViewer` provides data to your template engine, it will be wrapped in an instance of [`ViewLayerData`](api:SilverStripe\View\ViewLayerData). `ViewLayerData` makes sure data is cast appropriately according to the model that provides it (where possible). When you request values from it, they also come wrapped in `ViewLayerData` unless they're null, or you specifically call the [`getRawDataValue()`](api:SilverStripe\View\ViewLayerData::getRawDataValue()) method.

### Gotchas

- You will likely need to use `ViewLayerData::getRawDataValue()` to get values that can be used in conditional statements, since for example an object will always be truthy even if it represents a boolean `false`.
  Note that in some cases a `DBField` will be returned from a method on your model or controller directly, in which case `ViewLayerData::getRawDataValue()` will return the `DBField` instance instead of the truly *raw* value.
- Values in the `$overlay` argument for the [`TemplateEngine::render()`](api:SilverStripe\View\TemplateEngine::render()) and [`TemplateEngine::renderString()`](api:SilverStripe\View\TemplateEngine::renderString()) methods, and values returned from `TemplateGlobalProvider`, will not be wrapped in `ViewLayerData`. You should wrap those yourself before using them.
