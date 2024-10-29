---
title: Unit and Integration Testing
summary: Test models, database logic and your object methods.
---

# Unit and integration testing

A unit test is an automated piece of code that invokes a unit of work in the application and then checks the behavior
to ensure that it works as it should. A simple example would be to test the result of a PHP method.

```php
// app/src/Page.php
namespace {
    use SilverStripe\CMS\Model\SiteTree;

    class Page extends SiteTree
    {
        public static function myMethod()
        {
            return (1 + 1);
        }
    }
}
```

```php
// app/tests/PageTest.php
namespace App\Test;

use Page;
use SilverStripe\Dev\SapphireTest;

class PageTest extends SapphireTest
{
    public function testMyMethod()
    {
        $this->assertEquals(2, Page::myMethod());
    }
}
```

> [!NOTE]
> Tests for your application should be stored in the `app/tests` directory. Test cases for add-ons should be stored in
> the `(modulename)/tests` directory.
>
> Test case classes should end with `Test` (e.g. `PageTest`) and test methods must start with `test` (e.g. `testMyMethod`).
>
> Ensure you [import](https://php.net/manual/en/language.namespaces.importing.php#example-252) any classes you need for the test, including `SilverStripe\Dev\SapphireTest` or `SilverStripe\Dev\FunctionalTest`.

A Silverstripe CMS unit test is created by extending one of two classes, [SapphireTest](api:SilverStripe\Dev\SapphireTest) or [FunctionalTest](api:SilverStripe\Dev\FunctionalTest).

[SapphireTest](api:SilverStripe\Dev\SapphireTest) is used to test your model logic (such as a `DataObject`), and [FunctionalTest](api:SilverStripe\Dev\FunctionalTest) is used when
you want to test a `Controller`, `Form` or anything that requires a web page.

> [!NOTE]
> `FunctionalTest` is a subclass of `SapphireTest` so will inherit all of the behaviors. By subclassing `FunctionalTest`
> you gain the ability to load and test web pages on the site.
>
> `SapphireTest` in turn, extends `PHPUnit\Framework\TestCase`. For more information on `PHPUnit\Framework\TestCase` see
> the [PHPUnit](https://www.phpunit.de) documentation. It provides a lot of fundamental concepts that we build on in this
> documentation.

## Test databases and fixtures

> [!CAUTION]
> As the test runner will create new databases for the tests to run, the database user should have the appropriate
> permissions to create new databases on your server.

Silverstripe CMS tests create their own database when the test starts and fixture files are specified. New `ss_tmp` databases are created using the same
connection details you provide for the main website. The new `ss_tmp` database does not copy what is currently in your
application database. To provide seed data use a [Fixture](fixtures) file.

> [!WARNING]
> The test database is rebuilt every time one of the test methods is run and is removed afterwards. If the test is interrupted, the database will not be removed. Over time, you may have several hundred test
> databases on your machine. To get rid of them, run `sake tasks:CleanupTestDatabasesTask`.

## Custom PHPUnit configuration

The `phpunit` executable can be configured by command line arguments or through an XML file. Silverstripe CMS comes with a
default `phpunit.xml.dist` that you can use as a starting point. Copy the file into `phpunit.xml` and customize to your
needs.

```xml
<!-- phpunit.xml -->
<phpunit bootstrap="vendor/silverstripe/framework/tests/bootstrap.php" colors="true">
    <testsuites>
        <testsuite name="Default">
            <directory>app/tests</directory>
        </testsuite>
    </testsuites>
    <groups>
        <exclude>
            <group>sanitychecks</group>
        </exclude>
    </groups>
</phpunit>
```

### SetUp() and tearDown()

In addition to loading data through a [Fixture File](fixtures), a test case may require some additional setup work to be
run before each test method. For this, use the PHPUnit `setUp` and `tearDown` methods. These are run at the start and
end of each test.

```php
namespace App\Test;

use SilverStripe\Core\Config\Config;
use SilverStripe\Dev\SapphireTest;
use SilverStripe\Versioned\Mode\Versioned;

class PageTest extends SapphireTest
{
    protected $usesDatabase = true;

    protected function setUp(): void
    {
        parent::setUp();

        // create 100 pages
        for ($i = 0; $i < 100; $i++) {
            $page = new Page(['Title' => "Page $i"]);
            $page->write();
            $page->copyVersionToStage(Versioned::DRAFT, Versioned::LIVE);
        }

        // set custom configuration for the test.
        Config::modify()->update('Foo', 'bar', 'Hello!');
    }

    public function testMyMethod()
    {
        // ...
    }

    public function testMySecondMethod()
    {
        // ...
    }
}
```

`tearDownAfterClass` and `setUpBeforeClass` can be used to run code just once for the file rather than before and after
each individual test case. Remember to class the parent method in each method to ensure the core boot-strapping of tests
takes place.

```php
namespace App\Test;

use SilverStripe\Dev\SapphireTest;

class PageTest extends SapphireTest
{
    public static function setUpBeforeClass(): void
    {
        parent::setUpBeforeClass();

        // ...
    }

    public static function tearDownAfterClass(): void
    {
        parent::tearDownAfterClass();

        // ...
    }

    // ...
}
```

### Config and injector nesting

A powerful feature of both [`Config`](/developer_guides/configuration/configuration/) and [`Injector`](/developer_guides/extending/injector/) is the ability to "nest" them so that you can make changes that can easily be discarded without having to manage previous values.

The testing suite makes use of this to "sandbox" each of the unit tests as well as each suite to prevent leakage between tests.

If you need to make changes to `Config` (or `Injector`) for each test (or the whole suite) you can safely update `Config` (or `Injector`) settings in the `setUp` or `tearDown` functions.

It's important to remember that the `parent::setUp();` functions will need to be called first to ensure the nesting feature works as expected.

```php
namespace App\Test;

use SilverStripe\Core\Config\Config;
use SilverStripe\Dev\SapphireTest;

class MyTest extends SapphireTest
{
    public static function setUpBeforeClass(): void
    {
        parent::setUpBeforeClass();
        //this will remain for the whole suite and be removed for any other tests
        Config::inst()->update('ClassName', 'var_name', 'var_value');
    }

    public function testFeatureDoesAsExpected()
    {
        //this will be reset to 'var_value' at the end of this test function
        Config::inst()->update('ClassName', 'var_name', 'new_var_value');
    }

    public function testAnotherFeatureDoesAsExpected()
    {
        // this will be 'var_value'
        Config::inst()->get('ClassName', 'var_name');
    }
}
```

### Asserting errors, warnings and notices

The `phpunit` no longer supports the ability to assert expected errors, warning and notices. Calling [`SapphireTest::enableErrorHandler()`](api:SilverStripe\Dev\SapphireTest::enableErrorHandler()) at beginning on your unit test, or in the `setUp()` method, will use a custom error handler to catch any errors, warning and notices and re-throw them as exceptions which can then be asserted by calling `$this->expectException(<exception-class>);`. The following error types are converted to the following exceptions:

| Error types | Exception |
| ----------- | --------- |
| E_USER_ERROR, E_RECOVERABLE_ERROR | [`ExpectedErrorException`](api:SilverStripe\Dev\Exceptions\ExpectedErrorException) |
| E_NOTICE, E_USER_NOTICE | [`ExpectedNoticeException`](api:SilverStripe\Dev\Exceptions\ExpectedNoticeException) |
| E_WARNING, E_USER_WARNING | [`ExpectedWarningException`](api:SilverStripe\Dev\Exceptions\ExpectedWarningException) |

All other error types are not converted to exceptions and are handled by the default PHP error handler.

## Related documentation

- [How to Write a SapphireTest](how_tos/write_a_sapphiretest)
- [How to Write a FunctionalTest](how_tos/write_a_functionaltest)
- [Fixtures](fixtures)

## API documentation

- [SapphireTest](api:SilverStripe\Dev\SapphireTest)
- [FunctionalTest](api:SilverStripe\Dev\FunctionalTest)
