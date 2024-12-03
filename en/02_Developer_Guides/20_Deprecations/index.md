---
title: Deprecating code
summary: Details about how to deprecate code and emit or suppress deprecation warnings
icon: exclamation-circle
---

# Deprecating code

Needs of developers (both on core framework and custom projects) can outgrow the capabilities
of a certain API. Existing APIs might turn out to be hard to understand, maintain, test or stabilise.
In these cases, it is best practice to "refactor" these APIs into something more useful.

Sometimes refactoring API means we rename or entirely remove classes and methods in a major release.
To make upgrading to that release easier, deprecation warnings should be provided so developers know what is
going to be removed and how they need to update their code.

## Deprecating API best practice

- Add a `@deprecated` item to the docblock tag, with the version when the code was deprecated and a message indicating what to use instead.
- Update the deprecated code to throw a [Deprecation::notice()](api:SilverStripe\Dev\Deprecation::notice()) warning.
- Both the docblock and error message should contain the **version** where the functionality is deprecated from.
  So, if you're committing the change to a *1.2* minor release, the version will be *1.2.0*.
- Make sure that the old deprecated method works by calling the new function where possible - avoid duplicated code.
- Deprecated APIs can be removed only after developers have had sufficient time to react to the changes. Hence, deprecated APIs should be removed in major releases only. Between major releases, leave the code in place with a deprecation warning.

> [!NOTE]
> If there is no immediate replacement for deprecated code that is being called, either because the replacement is not available until the next major version, or because there is not a plan for there to be a replacement, the message should be "Will be removed without equivalent functionality to replace it."

When deprecating a method:

- Add the following docblock `@deprecated 1.2.0 Use anotherMethod() instead`
- `Deprecation::notice('1.2.0', 'Use anotherMethod() instead');` to the top of the method
- Wrap `Deprecation::notice()` with `Deprecation::withSuppressedNotice()` if there's no replacement for that deprecated method and it's not feasible to wrap all calls to the method

When deprecating a class:

- Add the following docblock `@deprecated 1.2.0 Use AnotherClass instead`
- Add `Deprecation::notice('1.2.0', 'Use AnotherClass instead', Deprecation::SCOPE_CLASS);` to the top of `__construct()`
- Wrap `Deprecation::notice()` with `Deprecation::withSuppressedNotice()` if there's no replacement for that deprecated class and it's not feasible to wrap all instantiations of the class

    ```php
    namespace App;

    /**
     * @deprecated 1.2.0 Will be removed without equivalent functionality
     */
    class MyDeprecatedClass extends AnotherClass
    {
        public function __construct()
        {
            Deprecation::withSuppressedNotice(function () {
                Deprecation::notice(
                    '1.2.0',
                    'Will be removed without equivalent functionality',
                    Deprecation::SCOPE_CLASS
                );
            });
            parent::__construct();
        }
    }
    ```

When deprecating config:

- Add the following docblock `@deprecated 1.2.0 Use different_config instead`

When deprecating some behaviour, combination of configuration values, parameters, etc:

- Add `Deprecation::notice('1.2.0', 'Using x with y is deprecated. Do [other thing] instead', Deprecation::SCOPE_GLOBAL);`
- It may not be immediately clear where this type of deprecation warning should go. In that case, add it to the `HTTPApplication::warnAboutDeprecatedSetups()` method.
- It may be appropriate to link to some documentation in the message for this type of deprecation warning.

In all cases if the replacement method or configuration property exists in another class, or you're replacing one class with another, refer to the replacement using its fully qualified class name.

## Avoiding deprecated API

Wherever possible, once some API has been deprecated, we should stop using it. This allows projects to emit deprecation warnings and have an accurate list of code that they need to change.

In some cases, we may not yet have a replacement for code that is required for the current major - but that we still have to call and support internally. In these cases, since we can't stop calling the code, wrap the call to the deprecated code in `Deprecation::withSuppressedNotice()` e.g:

```php
use SilverStripe\Dev\Deprecation;

// ...

// The $myVariable variable will get the result of the call to the deprecated $obj->myDeprecatedMethod()
$myVariable = Deprecation::withSuppressedNotice(function () {
    return $obj->myDeprecatedMethod();
});
```

For any unit tests using the deprecated method/class/config, add the following the the top of the unit test. This ensures that deprecated code is still supported as usual when `Deprecation` warnings are not enabled, though the tests are skipped whenever you are testing to see if deprecated code is still be called.

```php
namespace SilverStripe\Test;

use SilverStripe\Dev\Deprecation;
use SilverStripe\Dev\SapphireTest;

class MyTest extends SapphireTest
{
    public function testSomething()
    {
        if (Deprecation::isEnabled()) {
            $this->markTestSkipped('Test calls deprecated code');
        }
        // ...
    }
}
```

Here's an example for replacing `Director::isDev()` with a (theoretical) `SilverStripe\Core\Env::is_dev()`:

```php
namespace SilverStripe\Control;

use SilverStripe\Dev\Deprecation;
// ...

class Director
{
    // ...

    /**
     * Returns true if your are in development mode
     * @deprecated 1.2.0 Use SilverStripe\Core\Env::is_dev() instead.
     */
    public function isDev()
    {
        Deprecation::notice('1.2.0', 'Use SilverStripe\Core\Env::is_dev() instead');
        return Env::is_dev();
    }
}
```

This change could be committed to a minor release like *1.2.0*, and remains deprecated in all subsequent minor releases
(e.g. *1.3.0*), until a new major release (e.g. *2.0.0*), at which point it gets removed from the codebase.

## Enabling deprecation warnings

Deprecation warnings aren't enabled by default.  They can be turned on for dev environments with one of the following methods:

`.env`

```bash
SS_DEPRECATION_ENABLED=true
```

```php
// app/_config.php
use SilverStripe\Dev\Deprecation;

Deprecation::enable();
```

To test that deprecated code is no longer being called, run code via CI in an installer/kitchen-sink project that has deprecations enabled. Then view the CI output in the "Run tests" of the GitHub actions CI job and/or view the contents of `silverstripe.log` in the GitHub actions CI artifact to see if there are deprecation warnings. There should be zero deprecation warnings.

See [upgrading - deprecations](/upgrading/deprecations/) for more details about enabling deprecation warnings.
