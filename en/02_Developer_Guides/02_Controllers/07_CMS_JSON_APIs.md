---
title: CMS JSON APIs
summary: Creating standardised JSON APIs for authenticated users in the CMS.
---

# CMS JSON APIs

This document contains a standard set of conventions to be used when creating JSON APIs in the CMS that are used in conjunction with AJAX requests from authenticated CMS users.

To view an example of a controller that follows these standards see [`LinkFieldController`](api:SilverStripe\LinkField\Controllers\LinkFieldController).

## Making the controller "REST-like" and its relation with `FormSchema`

It's recommend you design your API with a "REST-like" approach for JSON requests in your application, acknowledging that certain aspects may not strictly adhere to pure REST principles. This means that you should use the HTTP methods GET, POST, and DELETE, though typically not others such as PUT or PATCH.

The reason for not implementing PUT or PATCH is because `FormSchema` and react `FormBuilder` should be used whenever you need to render a form that uses the standard set of react form fields. `FormSchema` and `FormBuilder` will handle data submission, data validation and showing validation errors on the frontend.

`FormSchema` diverges from REST principles as it utilises a combination of JSON to retrieve the `FormSchema` and submitting data using `application/x-www-form-urlencoded` POST requests. This method has proven effective, and there are currently no plans to alter its functionality.

Because of this you should generally avoid updating large parts of a DataObject with either a POST, PUT or PATCH requests, since `FormSchema` can handle that for you. We recommend only use POST requests to submit small amounts of data such as creating a new record without any data or updating sort order fields.

## Creating a controller

Create a subclass of [`AdminController`](api:SilverStripe\Admin\AdminController). This ensures that users must be logged in to the admin interface to access the endpoint. Additionally, it provides access to the methods [`jsonSuccess()`](api:SilverStripe\Admin\AdminController::jsonSuccess()) and [`jsonError()`](api:SilverStripe\Admin\AdminController::jsonError()).

> [!WARNING]
> To enhance security, do not create a direct subclass of [`Controller`](api:SilverStripe\Control\Controller) routed using YAML on the `/admin/*` route. This practice is strongly discouraged as it circumvents the requirement to log in to the CMS to access the endpoints. At best you'd be re-implementing logic that already exists.

When naming this class, it's best practice to add a "Controller" suffix to this class, for instance name it `MySomethingController`.

Define the URL segment of your controller using the [`url_segment`](api:SilverStripe\Admin\AdminController->url_segment) configuration property. For example `private static string $url_segment = 'my-segment';`. For small optional modules, this may typically be the composer name of the module, for instance "linkfield".

Use the [`required_permission_codes`](api:SilverStripe\Admin\AdminController->required_permission_codes) configuration property to declare what permissions are required to access endpoints on the controller. For example `private static string $required_permission_codes = 'CMS_ACCESS_CMSMain';`.

See [user permissions](/developer_guides/security/permissions/) for more information about declaring permissions.

If you need form schema functionality, you will need to create a subclass of [`FormSchemaController`](api:SilverStripe\Admin\FormSchemaController) instead.

## Handling requests with `$url_handlers`

Utilise the [`url_handlers`](api:SilverStripe\Control\Controller->url_handlers) configuration property to get the following benefits:

- Ensure the HTTP request method aligns with the intended use for each method, for instance, restricting it to GET or POST.
- If you're using form schema logic in a subclass of `LeftAndMain`, avoid potential conflicts with existing methods on the superclass, such as [`LeftAndMain::sort()`](api:SilverStripe\Admin\LeftAndMain::sort()), by structuring the endpoint URL segment as `sort` and associating it with a method like `MySomethingController::apiSort()`.

Use the request param `$ItemID` if you need a record ID into a URL so that you have an endpoint for a specific record. Use `$ItemID` because it's consistent with the request param used in Form Schema requests. For example, to use `$ItemID` in a GET request to view a single record:

```php
// app/src/Controllers/MySomethingController.php
namespace App\Controllers;

use SilverStripe\Admin\AdminController;
use SilverStripe\Control\HTTPResponse;

class MySomethingController extends AdminController
{
    // ...
    private static array $url_handlers = [
        'GET view/$ItemID' => 'apiView',
    ];

    private static array $allowed_actions = [
        'apiView',
    ];

    public function apiView(): HTTPResponse
    {
        $itemID = $request->param('ItemID');
        // Note: would normally validate that $itemID is a valid integer and that $obj exists
        $obj = MyDataObject::get()->byID($itemID);
        $data = ['ID' => $obj->ID, 'Title' => $obj->Title];
        return $this->jsonSuccess(200, $data);
    }
}
```

Remember to add all public methods that are used as endpoints to [`allowed_actions`](api:SilverStripe\Control\Controller->allowed_actions).

See [URL handlers](/developer_guides/controllers/routing/#url-handlers) for more details about how `url_handlers` and `allowed_actions` work.

## Permission checks

As mentioned in [creating a controller](#creating-a-controller) above, any permissions you add to the `required_permission_codes` configuration property for your controller will be checked before initialising the controller.

You should also incorporate additional permission checks, such as calling `canEdit()` on a `DataObject` record, into all relevant endpoints to ensure secure access control.

When returning `DataObject` records as JSON, remember to invoke `canView()` on each record. In a CMS context where the number of records is typically limited (e.g. by pagination), the performance impact of these checks should not be a significant concern. If the permission check fails then call `$this->jsonError(403);` to return a 403 status code.

## Return values and error handling

All public endpoint methods (aka "actions") must declare a return type of [`HTTPResponse`](api:SilverStripe\Control\HTTPResponse).

All return values should be returned via the `jsonSuccess()` method to create the `HTTPResponse` as this method is used to standardise JSON responses in the CMS.

Do not throw exceptions in the controller, as this leads to a poor content-editor experience. Instead all non-success conditions should call `jsonError()`.

### Using `jsonSuccess()`

When using the optional `$data` parameter in `jsonSuccess()` to return JSON in the response body, do not add any "success metadata" around it, for example `['success' => true, 'data' => $data]`. Instead, solely rely on standard HTTP status codes to clearly indicate the success of the operation.

For scenarios where no JSON data is returned in the response body upon success, use the status code 201 without the `$data` parameter i.e. `return $this->jsonSuccess(201);`. Alternatively, when the response includes JSON data, usually return a 200 status code i.e. `return $this->jsonSuccess(200, $data);`.

### Using `jsonError()`

When calling `jsonError()`, you don't need to use the `return` keyword because that method throws an exception. The exception gets caught in [`handleRequest()`](api:SilverStripe\Control\RequestHandler::handleRequest()) and converted into an `HTTPResponse` object.

Generally you should not include a message outlining the nature of the error when calling `jsonError()`. Instead just include the appropriate HTTP status code for the error type, for example call `$this->jsonError(403)` if a permission check fails.

If you do include a message, remember that error messages are only intended for developers so do not use the `_t()` function to make them translatable. Do not use any returned messages on the frontend for things like toast notifications, instead those messages should be added directly in JavaScript.

> [!NOTE]
> Despite the slightly convoluted JSON format returned by `jsonError()` with multiple nodes, its usage remains consistent with `FormSchema`. It's better to use this method for uniformity rather than introducing separate methods for `FormSchema` and non-FormSchema failures.

## CSRF token

When performing non-view operations, include an `X-SecurityID` header in your JavaScript request, with its value set to the security token.

Access the token value in JavaScript like so:

```js
import Config from 'lib/Config';

const securityID = Config.get('SecurityID');
```

> [!WARNING]
> The `lib/Config` import is provided by the `silverstripe/admin` module via [`@silverstripe/webpack-config`](https://www.npmjs.com/package/@silverstripe/webpack-config).

Ensure the security of your endpoints by validating the security token on relevant endpoints.

```php
use SilverStripe\Security\SecurityToken;
// ...
if (!SecurityToken::inst()->checkRequest($this->getRequest())) {
    $this->jsonError(400);
}
```

> [!NOTE]
> The `400` HTTP status code used here is consistent with the code used when the CSRF check fails when submitting data using `FormSchema`.

## Passing values from PHP to global JavaScript

To transmit values from PHP to global JavaScript, which is used for component configuration as opposed to data, override [`getClientConfig()`](api:SilverStripe\Admin\AdminController::getClientConfig()) within your controller. Begin your method with `$clientConfig = parent::getClientConfig();` to ensure proper inheritance, or better yet, use [`beforeExtending()`](api:SilverStripe\Core\Extensible::beforeExtending()) so that extensions implementing the `updateClientConfig()` extension hook can update your config.

Include any relevant links to endpoints in the client configuration. For example, add `'myEndpointUrl' => $this->Link('my-endpoint')`, where `my-endpoint` is specified in `private static array $url_handlers`.

```php
// app/src/Controllers/MySomethingController.php
namespace App\Controllers;

use SilverStripe\Admin\AdminController;

class MySomethingController extends AdminController
{
    // ...
    private static array $url_handlers = [
        'my-endpoint' => 'apiEndpoint',
    ];

    public function getClientConfig(): array
    {
        $this->beforeExtending('updateClientConfig', function (array &$clientConfig): void {
            $clientConfig['myForm'] = [
                'myEndpointUrl' => $this->Link('my-endpoint'),
            ];
        });
        return parent::getClientConfig();
    }
}
```

In JavaScript, access these values as following:

```js
import Config from 'lib/Config';

const endpointUrl = Config.getSection('App\\Controller\\MySomethingController').myForm.myEndpointUrl;
```

## JavaScript AJAX requests

Use the `backend` helper which is a wrapper around `fetch()` when making JavaScript requests. Import it using `import backend from 'lib/Backend';`.

The `backend` helper is able to use a `catch()` block to handle 400-500 response codes, offering a more streamlined approach compared to using vanilla `fetch()`. It also provides handy shorthand methods such as `.get()` and `.post()`, for writing concise code.

The following code will make a POST request to an endpoint passing JSON data in the request body.

```js
import backend from 'lib/Backend';
import Config from 'lib/Config';

const section = 'App\\Controller\\MySomethingController';
const endpoint = `${Config.getSection(section).myForm.myEndpointUrl}`;
const data = { somekey: 123 };
const headers = { 'X-SecurityID': Config.get('SecurityID') };
backend.post(endpoint, data, headers)
  .then(() => {
    // handle 200-299 status code response here
  })
  .catch(() => {
    // handle 400-500 status code response here
  });
```

> [!WARNING]
> The `lib/Config` and `lib/Backend` imports are provided by the `silverstripe/admin` module via [`@silverstripe/webpack-config`](https://www.npmjs.com/package/@silverstripe/webpack-config).

On the controller's endpoint method, retrieve the POST data using `$json = json_decode($this->getRequest()->getBody());`.

## Unit testing

Write unit tests with a subclass of [`FunctionalTest`](api:SilverStripe\Dev\FunctionalTest) instead of the regular [`SapphireTest`](api:SilverStripe\Dev\SapphireTest). This allows you to make HTTP requests to your endpoints and ensures comprehensive functional testing.

See [functional testing](/developer_guides/testing/functional_testing/) for more information about setting up and running functional tests.
