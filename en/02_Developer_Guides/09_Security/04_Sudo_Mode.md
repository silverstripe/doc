---
title: Sudo Mode
summary: Require users to verify their identity when performing sensitive actions
icon: key
---

# Sudo mode

Sudo mode represents a heightened level of permission in that you are more certain that the current CMS user is actually the person whose account is logged in. This is performed by re-validating that the account's password is correct, and will then last for a certain amount of time (configurable) until it will be checked again.

Sudo mode is not active when a user logs in to the CMS. If the PHP session lifetime expires before the sudo mode lifetime, that sudo mode will also be cleared. If the user leaves their CMS open, or continues to use it, for an extended period of time with automatic refreshing in the background, sudo mode will eventually deactivate once the max lifetime is reached.

Data protected by sudo mode is still viewable without entering the password, but any actions that modify data will require the user to enter their password. This is done to balance usability with security. The following [`DataObject`](api:SilverStripe\ORM\DataObject) subclasses are protected by sudo mode:

- [`Member`](api:SilverStripe\Security\Member)
- [`Group`](api:SilverStripe\Security\Group)
- [`PermissionRole`](api:SilverStripe\Security\PermissionRole)
- [`PermissionRoleCode`](api:SilverStripe\Security\PermissionRoleCode)
- [`SiteConfig`](api:SilverStripe\SiteConfig\SiteConfig)

## Adding or removing sudo mode for a `DataObject`

To add sudo mode for a particular `DataObject`, simply set the [`DataObject.require_sudo_mode`](api:SilverStripe\ORM\DataObject->require_sudo_mode) configuration property to `true`, either via yml or directly on the class.

While it's not recommended, to remove sudo mode for a particular `DataObject`, set the [`DataObject.require_sudo_mode`](api:SilverStripe\ORM\DataObject->require_sudo_mode) configuration property to `false` via yml.

```yml
SilverStripe\Security\Member:
  require_sudo_mode: false
```

## Configuring the lifetime

The default [`SudoModeServiceInterface`](api:SilverStripe\Security\SudoMode\SudoModeServiceInterface) implementation is [`SudoModeService`](api:SilverStripe\Security\SudoMode\SudoModeService), and its lifetime can be configured with YAML. You should read the lifetime value using `SudoModeServiceInterface::getLifetime()`.

```yml
SilverStripe\Security\SudoMode\SudoModeService:
  lifetime_minutes: 25
```

## Enabling sudo mode for controller endpoints

Performing a sudo mode verification check in a controller endpoint by using the sudo mode service to validate the request:

```php
namespace App\Control;

use SilverStripe\Core\Injector\Injector;
use SilverStripe\Security\SudoMode\SudoModeServiceInterface;

class MyController extends Controller
{
    // ...

    public function myAction(HTTPRequest $request): HTTPResponse
    {
        $service = Injector::inst()->get(SudoModeServiceInterface::class);
        if (!$service->check($request->getSession())) {
            return $this->httpError(403, 'Sudo mode is required for this action');
        }
        // ... continue with sensitive operations
    }
}
```

## Using sudo mode in a react component

The `silverstripe/admin` module defines a [React Higher-Order-Component](https://reactjs.org/docs/higher-order-components.html) (aka HOC) which can
be applied to React components in your module or code to intercept component rendering and show a "sudo mode required"
information and log in screen, which will validate, activate sudo mode, and re-render the wrapped component afterwards
on success.

> [!WARNING]
> The `WithSudoMode` HOC is exposed via [Webpack's expose-loader plugin](https://webpack.js.org/loaders/expose-loader/). You will need to add it as a [webpack external](https://webpack.js.org/configuration/externals/) to use it. The recommended way to do this is via the [@silverstripe/webpack-config npm package](https://www.npmjs.com/package/@silverstripe/webpack-config) which handles all the external configuration for you.

You can get the injector to apply the HOC to your component automatically using [injector transformations](/developer_guides/customising_the_admin_interface/reactjs_redux_and_graphql/#transforming-services-using-middleware):

```js
import WithSudoMode from 'containers/SudoMode/SudoMode';

Injector.transform('MyComponentWithSudoMode', (updater) => {
  updater.component('MyComponent', WithSudoMode);
});
```

If the user has already activated sudo mode and it has not expired, they can interact with your component automatically. Otherwise, they will need to verify their identity by re-entering their password.

![Sudo mode HOC example](../../_images/sudomode.png)

### Requirements for adding to a component

While the `sudoModeActive` prop is gathered automatically from the Redux configuration store, backend validation is
also implemented to ensure that the frontend UI cannot simply be tampered with to avoid re-validation on sensitive
operations.

Ensure you protect your endpoints from [cross site request forgery (CSRF)](/developer_guides/forms/form_security/#cross-site-request-forgery-csrf)
at the same time.
