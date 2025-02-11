---
title: Introduction to Forms
summary: An introduction to creating a Form instance and handling submissions.
iconBrand: wpforms
---

# Forms

The HTML `Form` is the most used way to interact with a user. Silverstripe CMS provides classes to generate forms through
the [Form](api:SilverStripe\Forms\Form) class, [FormField](api:SilverStripe\Forms\FormField) instances to capture data and submissions through [FormAction](api:SilverStripe\Forms\FormAction).

## Creating a form

Creating a [Form](api:SilverStripe\Forms\Form) has the following signature.

```php
use SilverStripe\Forms\Form;

$form = Form::create(
    // the Controller to render this form on
    $controller,
    // name of the method that returns this form on the controller
    $name,
    // FieldList of FormField instances
    $fields,
    // FieldList of FormAction instances
    $actions,
    // optional use of Validator object
    $validator
);
```

In practice, this looks like:

```php
// app/src/PageType/MyFormPageController.php
namespace App\PageType;

use PageController;
use SilverStripe\Forms\FieldList;
use SilverStripe\Forms\Form;
use SilverStripe\Forms\FormAction;
use SilverStripe\Forms\TextField;
use SilverStripe\Forms\Validation\RequiredFieldsValidator;

class MyFormPageController extends PageController
{
    private static $allowed_actions = [
        'getHelloForm',
    ];

    private static $url_handlers = [
        'HelloForm' => 'getHelloForm',
    ];

    public function getHelloForm()
    {
        $fields = FieldList::create(
            TextField::create('Name', 'Your Name')
        );

        $actions = FieldList::create(
            FormAction::create('doSayHello')->setTitle('Say hello')
        );

        $required = RequiredFieldsValidator::create('Name');

        $form = Form::create($this, 'HelloForm', $fields, $actions, $required);

        return $form;
    }

    public function doSayHello($data, Form $form)
    {
        $form->sessionMessage('Hello ' . $data['Name'], 'success');

        return $this->redirectBack();
    }
}
```

```ss
<%-- app/templates/App/PageType/MyFormPage.ss --%>
$HelloForm
```

> [!NOTE]
> The examples above use `FormField::create()` instead of the  `new` operator (`new FormField()`). This is best practice,
> as it allows you to use [dependency injection](/developer_guides/extending/injector/) to replace the actual class being used at runtime.
>
> As an extra incentive, it also allows you to chain operations like `setTitle()` without assigning the field instance to a temporary
> variable.

When constructing the `Form` instance (`Form::create($controller, $name)`) both controller and name are required. The
`$controller` and `$name` are used to allow Silverstripe CMS to calculate the origin of the `Form object`. When a user
submits the `HelloForm` from your `contact-us` page the form submission will go to `contact-us/HelloForm` before any of
the [FormAction](api:SilverStripe\Forms\FormAction). The URL is known as the `$controller` instance will know the 'contact-us' link and we provide
`HelloForm` as the `$name` of the form. `$name` **needs** to match the method name.

If you want to comply with PSR12, you may want to call your method `getHelloForm` (instead of just `HelloForm`) - in that case, make sure you
add the appropriate `$url_handlers` configuration per below.

Because the `getHelloForm()` method will be the location the user is taken to when submitting the form, it needs to be handled like any other
controller action. To grant it access through URLs, we add it to the `$allowed_actions` array.

```php
namespace App\PageType;

use PageController;

class MyFormPageController extends PageController
{
    private static $allowed_actions = [
        'getHelloForm',
    ];

    private static $url_handlers = [
        'HelloForm' => 'getHelloForm',
    ];

    // ...
}
```

See [URL Handlers](/developer_guides/controllers/routing/#url-handlers) for more information about handling controller actions.

> [!WARNING]
> Form actions (`doSayHello()`), on the other hand, should *not* be included in `$allowed_actions`; these are handled
> separately through [Form::httpSubmission()](api:SilverStripe\Forms\Form::httpSubmission()).

## Adding formFields

Fields in a [Form](api:SilverStripe\Forms\Form) are represented as a single [FieldList](api:SilverStripe\Forms\FieldList) instance containing subclasses of [FormField](api:SilverStripe\Forms\FormField).
Some common examples are [TextField](api:SilverStripe\Forms\TextField) or [DropdownField](api:SilverStripe\Forms\DropdownField).

```php
use SilverStripe\Forms\TextField;
TextField::create($name, $title, $value);
```

> [!NOTE]
> A list of the common FormField subclasses is available on the [Common Subclasses](field_types/common_subclasses/) page.

The fields are added to the [FieldList](api:SilverStripe\Forms\FieldList) `fields` property on the `Form` and can be modified at up to the point the
`Form` is rendered.

```php
use SilverStripe\Forms\EmailField;
use SilverStripe\Forms\FieldList;
use SilverStripe\Forms\Form;
use SilverStripe\Forms\TextField;

$fields = FieldList::create(
    TextField::create('Name'),
    EmailField::create('Email')
);

$form = Form::create($controller, 'MethodName', $fields, ...);

// or use `setFields`
$form->setFields($fields);

// to fetch the current fields..
$fields = $form->getFields();
```

A field can be appended to the [FieldList](api:SilverStripe\Forms\FieldList).

```php
use SilverStripe\Forms\Tab;
use SilverStripe\Forms\TextField;

$fields = $form->Fields();

// add a field
$fields->push(TextField::create(/* ... */));

// insert a field before another one
$fields->insertBefore('Email', TextField::create(/* ... */));

// insert a field after another one
$fields->insertAfter('Name', TextField::create(/* ... */));

// insert a tab before the main content tab (used to position tabs in the CMS)
$fields->insertBefore('Main', Tab::create(/* ... */));
// Note: you need to create and position the new tab prior to adding fields via addFieldToTab()
```

Fields can be fetched after they have been added in.

```php
$email = $form->Fields()->dataFieldByName('Email');
$email->setTitle('Your Email Address');
```

Fields can be removed from the form.

```php
$form->getFields()->removeByName('Email');
```

> [!CAUTION]
> Forms can be tabbed (such as the CMS interface). In these cases, there are additional functions such as `addFieldToTab`
> and `removeFieldByTab` to ensure the fields are on the correct interface. See [Tabbed Forms](tabbed_forms) for more
> information on the CMS interface.

## Modifying formFields

Each [FormField](api:SilverStripe\Forms\FormField) subclass has a number of methods you can call on it to customise its' behavior or HTML markup. The
default `FormField` object has several methods for doing common operations.

> [!WARNING]
> Most of the `set` operations will return the object back so methods can be chained.

```php
use SilverStripe\Forms\TextField;

$field = TextField::create(/* ... */)
    ->setMaxLength(100)
    ->setAttribute('placeholder', 'Enter a value..')
    ->setTitle('');
```

### Custom templates

The [Form](api:SilverStripe\Forms\Form) HTML markup and each of the [FormField](api:SilverStripe\Forms\FormField) instances are rendered into templates. You can provide custom
templates by using the `setTemplate` method on either the `Form` or `FormField`. For more details on providing custom
templates see [Form Templates](form_templates)

```php
namespace App\PageType;

use PageController;
use SilverStripe\Forms\Form;
use SilverStripe\Forms\TextField;

class MyFormPageController extends PageController
{
    private static $allowed_actions = [
        'getMyForm',
    ];

    private static $url_handlers = [
        'MyForm' => 'getMyForm',
    ];

    public function getMyForm()
    {
        $field = TextField::create(/* ... */);
        $field->setTemplate('CustomTextField');
        $field->setFieldHolderTemplate('CustomTextField_Holder');

        $form = Form::create(/* ... */);
        $form->setTemplate('CustomForm');

        return $form;
    }
}
```

## Adding formActions

[FormAction](api:SilverStripe\Forms\FormAction) objects are displayed at the bottom of the `Form` in the form of a `button` or `input` tag. When a
user presses the button, the form is submitted to the corresponding method.

```php
use SilverStripe\Forms\FormAction;

FormAction::create($action, $title);
```

As with [FormField](api:SilverStripe\Forms\FormField), the actions for a `Form` are stored within a [FieldList](api:SilverStripe\Forms\FieldList) instance in the `actions` property
on the form.

```php
namespace App\PageType;

use PageController;
use SilverStripe\Forms\FieldList;
use SilverStripe\Forms\Form;
use SilverStripe\Forms\FormAction;

class MyFormPageController extends PageController
{
    private static $allowed_actions = [
        'getMyForm',
    ];

    private static $url_handlers = [
        'MyForm' => 'getMyForm',
    ];

    public function getMyForm()
    {
        $fields = FieldList::create(/* .. */);

        $actions = FieldList::create(
            FormAction::create('doSubmitForm', 'Submit')
        );

        $form = Form::create($controller, 'MyForm', $fields, $actions);

        // Get the actions
        $actions = $form->Actions();

        // As actions is a FieldList, push, insertBefore, removeByName and other
        // methods described for `Fields` also work for actions.

        $actions->push(
            FormAction::create('doSecondaryFormAction', 'Another Button')
        );

        $actions->removeByName('doSubmitForm');
        $form->setActions($actions);

        return $form
    }

    public function doSubmitForm($data, $form)
    {
        // ...
    }

    public function doSecondaryFormAction($data, $form)
    {
        // ...
    }
}
```

The first `$action` argument for creating a `FormAction` is the name of the method to invoke when submitting the form
with the particular button. In the previous example, clicking the 'Another Button' would invoke the
`doSecondaryFormAction` method. This action can be defined (in order) on either:

- One of the `FormField` instances.
- The `Form` instance.
- The `Controller` instance.

> [!WARNING]
> If the `$action` method cannot be found on any of those or is marked as `private` or `protected`, an error will be
> thrown.

The `$action` method takes two arguments:

- `$data` an array containing the values of the form mapped from `$name => $value`
- `$form` the submitted [Form](api:SilverStripe\Forms\Form) instance.

```php
namespace App\PageType;

use PageController;
use SilverStripe\Forms\EmailField;
use SilverStripe\Forms\FieldList;
use SilverStripe\Forms\Form;
use SilverStripe\Forms\FormAction;
use SilverStripe\Forms\TextField;

class MyFormPageController extends PageController
{
    private static $allowed_actions = [
        'getMyForm',
    ];

    private static $url_handlers = [
        'MyForm' => 'getMyForm',
    ];

    public function getMyForm()
    {
        $fields = FieldList::create(
            TextField::create('Name'),
            EmailField::create('Email')
        );

        $actions = FieldList::create(
            FormAction::create('doSubmitForm', 'Submit')
        );

        $form = Form::create($controller, 'MyForm', $fields, $actions);

        return $form
    }

    public function doSubmitForm($data, $form)
    {
        // Submitted data is available as a map.
        echo $data['Name'];
        echo $data['Email'];

        // You can also fetch the value from the field.
        echo $form->Fields()->dataFieldByName('Email')->getValue();

        // Using the Form instance you can get / set status such as error messages.
        $form->sessionMessage('Successful!', 'good');

        // After dealing with the data you can redirect the user back.
        return $this->redirectBack();
    }
}
```

See [how_tos/handle_nested_data](How to: Handle nested form data) for more advanced use cases.

## Validation

Form validation is handled by the [Validator](api:SilverStripe\Forms\Validation\Validator) class and the `validator` property on the `Form` object. The validator
is provided with a name of each of the [FormField](api:SilverStripe\Forms\FormField)s to validate and each `FormField` instance is responsible for
validating its' own data value.

For more information, see the [Form Validation](validation) documentation.

```php
namespace App\PageType;

use PageController;
use SilverStripe\Forms\Form;
use SilverStripe\Forms\Validation\RequiredFieldsValidator;

class MyFormPageController extends PageController
{
    // ...

    public function getMyForm()
    {
        // ...

        $validator = RequiredFieldsValidator::create([
            'Name',
            'Email',
        ]);

        $form = Form::create($this, 'MyForm', $fields, $actions, $validator);

        return $form;
    }
}
```

## API documentation

- [Form](api:SilverStripe\Forms\Form)
- [FormField](api:SilverStripe\Forms\FormField)
- [FieldList](api:SilverStripe\Forms\FieldList)
- [FormAction](api:SilverStripe\Forms\FormAction)
