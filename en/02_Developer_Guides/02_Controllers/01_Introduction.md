---
title: Introduction to a Controller
summary: A brief look at the definition of a Controller, creating actions and how to respond to requests.
---

# Introduction to controllers

The following example is for a simple [`Controller`](api:SilverStripe\Control\Controller) class. When building off the Silverstripe CMS you will
subclass the base `Controller` class.

> [!NOTE]
> If you're using the `cms` module and dealing with [`SiteTree`](api:SilverStripe\CMS\Model\SiteTree) records then for your custom page controllers you
> would extend [`ContentController`](api:SilverStripe\CMS\Controllers\ContentController) or `PageController`.

```php
// app/src/Control/TeamController.php
namespace App\Control;

use App\Model\Team;
use SilverStripe\Control\Controller;
use SilverStripe\Control\HTTPRequest;

class TeamController extends Controller
{
    private static $allowed_actions = [
        'players',
    ];

    public function players(HTTPRequest $request)
    {
        $this->renderWith(Team::class . '_PlayerList');
    }
}
```

> [!WARNING]
> When choosing names for actions, avoid using the same name you've used for relations on the model the controller represents. If you have relations with the same name as controller actions, templates rendered for that controller which refer to the relation won't render as expected - they will attempt to render the action where you expect to be using the relation.
>
> For example if the controller above was for a `Team` model which had a `Players` relation, the action should not also be named `players`. Something like `showPlayers` would be more appropriate.

## Routing

We need to define the URL that this controller can be accessed on. In our case, the `TeamsController` should be visible
at `https://www.example.com/teams/` and the `players` custom action is at `https://www.example.com/team/players/`.

> [!NOTE]
> If you're extending `ContentController` or `PageController` for your `SiteTree` records you don't need to define the routes value as the `cms` handles
> routing for those.

```yml
# app/_config/routes.yml
---
Name: approutes
After: '#coreroutes'
---
SilverStripe\Control\Director:
  rules:
    'teams//$Action/$ID/$Name': 'App\Control\TeamController'
```

> [!CAUTION]
> Make sure that after modifying the `routes.yml` file you clear your Silverstripe CMS caches using `?flush=1`.

For more information about creating custom routes, see the [Routing](routing) documentation.

For information about how to get a URL (e.g. to use in templates) for a given controller action, see [Getting the URL for a controller action](#link) below.

## Actions

Actions in controllers are specific routes which are accessible via HTTP or CLI requests. They are often backed by a method, though a method does not need to be declared so long as there
is a template available to render for the given action.

Controllers respond by default to an `index` action. You don't need to implement this action as a method nor declare it as in `allowed_actions`, but you
can implement the `index()` method and return custom data to be used in the appropriate template (see [Template and Views](../templates)).

Action methods can return one of four things:

1. an array. The appropriate template will be rendered, using the values in the array you provided. The rendered result will be set as the body for the current [`HTTPResponse`](api:SilverStripe\Control\HTTPResponse) object.
1. a string (e.g. JSON or HTML markup). The string will be set as the body for the current `HTTPResponse` object.
1. an `HTTPResponse`. This can either be a new response or `$this->getResponse()`.
1. `$this` or `$this->customise()`. This will render the controller using the appropriate template and set the rendered result as the body for the current `HTTPResponse`.

> [!TIP]
> There are a couple of things to note here:
>
> - returning `$this` is the equivalent of returning an empty array.
> - returning `$this->customise()` is the equivalent of returning an array with data.

See [templates](#templates) below for information about declaring what template to use in the above scenarios.

A controller action can also throw an [`HTTPResponse_Exception`](api:SilverStripe\Control\HTTPResponse_Exception).
This is a special exception that indicates that a specific error HTTP code should be used in the response.
By throwing this exception, the execution pipeline can adapt and use any error handlers (e.g. via the [silverstripe/errorpage](https://github.com/silverstripe/silverstripe-errorpage/) module).

```php
// app/src/Control/TeamController.php
namespace App\Control;

use SilverStripe\Control\Controller;
// ...

class TeamController extends Controller
{
    // ...

    /**
     * Return some additional data to the current response that is waiting to go out, this makes $Title set to
     * 'My Team Name' and continues on with generating the response.
     */
    public function index(HTTPRequest $request)
    {
        // ...
    }

    /**
     * We can manually create a response and return that to ignore any previous data or modifications to the request.
     */
    public function someaction(HTTPRequest $request)
    {
        $this->setResponse(new HTTPResponse());
        $this->getResponse()->setStatusCode(400);
        $this->getResponse()->setBody('invalid');

        return $this->getResponse();
    }

    /**
     * Or, we can modify the response that is waiting to go out.
     */
    public function anotheraction(HTTPRequest $request)
    {
        $this->getResponse()->setStatusCode(400);

        return $this->getResponse();
    }

    /**
     * We can render HTML and leave Silverstripe CMS to set the response code and body.
     */
    public function htmlaction()
    {
        return $this->customise(ArrayData::create([
            'Title' => 'HTML Action',
        ]))->renderWith('MyCustomTemplate');
    }

    /**
     * We can send stuff to the browser which isn't HTML
     */
    public function ajaxaction()
    {
        $this->getResponse()->addHeader('Content-type', 'application/json');

        return json_encode([
            'json' => true,
        ]);
    }
}
```

For more information on how a URL gets mapped to an action see the [Routing](routing) documentation.

## Security

See the [Access Control](access_control) documentation.

## Templates

The template to use for a given action is determined in the following order:

1. If a template has been explicitly declared for the action in the `templates` property, it will be used.
1. If a template has been explicitly declared for the "index" action in the `templates` property, it will be used (regardless of what action is being rendered).
1. If the `template` property has been set at all, its value will be used.
1. If a template exists with the name of this class or any of its ancestors, suffixed with the name of the action name, it will be used.
    - e.g. for the `App\Control\TeamController` example, the "showPlayers" action would look for templates named `templates/App/Control/TeamController_showPlayers` and `templates/SilverStripe/Control/Controller_showPlayers` with the relevant file extension.
    - Note that the "index" action skips this step.
1. If a template exists with the name of this class or any of its ancestors (with no suffix), it will be used.
    - e.g. for the `App\Control\TeamController` example, it would look for templates named `templates/App/Control/TeamController` and `templates/SilverStripe/Control/Controller` with the relevant file extension.

> [!NOTE]
> Subclasses of `ContentController` additionally check for templates named similarly to the model the controller represents - for example a `HomePageController` class which represents a `HomePage` model will look for a `HomePage_{action}` template after checking `HomePageController_{action}`.

You can declare templates to be used for an action by setting the `templates` array. The key should be the name of the action,
and the value should be a template name, or array of template names in cascading precedence.

```php
namespace App\Control;

use SilverStripe\Control\Controller;

class TeamController extends Controller
{
    protected $templates = [
        'showPlayers' => 'TemplateForPlayers',
    ];

    private static $allowed_actions = [
        'showPlayers',
    ];
}
```

> [!WARNING]
> The `templates` property is *not* a configuration property, so if you declare it directly as in the above example you will
> override any templates declared in parent classes. If you want to keep template declarations from parent classes, you could
> apply new templates in a constructor like so:
>
> ```php
> namespace App\Control;
>
> class TeamController extends SomeParentController
> {
>     // ...
>
>     public function __construct()
>     {
>         parent::__construct();
>         $this->templates['showPlayers'] => 'TemplateForPlayers';
>     }
> }
> ```

As mentioned in [Actions](#actions) above, controller actions can return a string or `HTTPResponse` to bypass this template selection process.

For more information about templates, inheritance and how to render into views, See the
[Templates and Views](../templates) documentation.

## Getting the URL for a controller action {#link}

Each controller should declare the `url_segment` configuration property, using the non-variable portion of that controller's routing rule.

```php
namespace App\Control;

use SilverStripe\Control\Controller;

class TeamController extends Controller
{
    private static $url_segment = 'teams';
    // ...
}
```

You can then use the [`Link()`](api:SilverStripe\Control\RequestHandler::Link()) method to get a relative URL for your controller:

```php
$indexLink = $teamController::Link();
$playersActionLink = $teamController::Link('players');
```

You can of course also use `$Link` in a template.

> [!WARNING]
> If you have more complex logic for determining the link for your controller, you can override the `Link()` method - in that case you should
> be sure to invoke the `updateLink` extension method so that extensions can make changes as necessary: `$this->extend('updateLink', $link, $action);`

## Connecting pages to controllers

By default, a `SiteTree` subclass will be automatically associated with a controller which is in the same
namespace, and is named the same but suffixed with `Controller`. For example, `App\PageType\HomePage`
will be associated with a `App\PageType\HomePageController` if such a class exists.

If there is no controller for a specific page class, that page's ancestors will be checked until a suitable
controller is found.

If you find that your controllers are in a different namespace then you'll need to define the correct
controller in the `controller_name` configuration property.

Example controller:

```php
namespace App\Control;

use SilverStripe\Control\Controller;

class TeamPageController extends Controller
{
    private static $url_segment = 'teams';

    // ...
}
```

Calling `$this->Link()` in the above controller will now give a valid relative URL for accessing the controller on your site. If this is a subcontroller or otherwise has some part of its route that is dynamic, you will need to override the `Link()` method to resolve the correct URL dynamically.

Example page:

```php
namespace App\PageType;

use App\Control\TeamPageController;
use Page;

class TeamPage extends Page
{
    private static $controller_name = TeamPageController::class;
}
```

You'd now be able to access methods of the controller in the page's template

```ss
<%-- TeamPage.ss --%>
<p>{$getExample}</p>
```

## Related documentation

- [Execution Pipeline](../execution_pipeline)
- [Templates and Views](../templates)

## API documentation

- [Controller](api:SilverStripe\Control\Controller)
- [Director](api:SilverStripe\Control\Director)
