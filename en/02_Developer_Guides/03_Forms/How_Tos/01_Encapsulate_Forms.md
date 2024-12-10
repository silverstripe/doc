---
title: How to Encapsulate Forms
summary: Learn how to move a form from a controller into its own class definition.
iconBrand: wpforms
---

# How to encapsulate forms

Form definitions can often get long, complex and often end up cluttering up a `Controller` definition. We may also want
to reuse the `Form` across multiple `Controller` classes rather than just one. A nice way to encapsulate the logic and
code for a `Form` is to create it as a subclass to `Form`. Let's look at a example of a `Form` which is on our
`Controller` but would be better written as a subclass.

```php
// app/src/PageType/SearchPageController.php
namespace App\PageType;

use PageController;
use SilverStripe\Forms\CheckboxSetField;
use SilverStripe\Forms\CompositeField;
use SilverStripe\Forms\FieldList;
use SilverStripe\Forms\Form;
use SilverStripe\Forms\FormAction;
use SilverStripe\Forms\HeaderField;
use SilverStripe\Forms\NumericField;
use SilverStripe\Forms\OptionsetField;
use SilverStripe\Forms\Validation\RequiredFieldsValidator;

class SearchPageController extends PageController
{
    // ...

    public function searchForm()
    {
        $fields = FieldList::create(
            HeaderField::create('Header', 'Step 1. Basics'),
            OptionsetField::create('Type', '', [
                'foo' => 'Search Foo',
                'bar' => 'Search Bar',
                'baz' => 'Search Baz',
            ]),
            CompositeField::create(
                HeaderField::create('Header2', 'Step 2. Advanced '),
                CheckboxSetField::create('Foo', 'Select Option', [
                    'qux' => 'Search Qux',
                ]),
                CheckboxSetField::create('Category', 'Category', [
                    'Foo' => 'Foo',
                    'Bar' => 'Bar',
                ]),
                NumericField::create('Minimum', 'Minimum'),
                NumericField::create('Maximum', 'Maximum')
            )
        );

        $actions = FieldList::create(
            FormAction::create('doSearchForm', 'Search')
        );

        $required = RequiredFieldsValidator::create([
            'Type',
        ]);

        $form = Form::create($this, __FUNCTION__, $fields, $actions, $required);
        $form->setFormMethod('GET');

        $form->addExtraClass('no-action-styles');
        $form->disableSecurityToken();

        return $form;
    }

    public function doSearchForm(array $data, Form $form)
    {
        // Do something with the data, return results, or redirect
    }
}
```

That's quite a lot of code to include on our controller and generally makes the file look much more complex than it
should be. Good practice would be to move this to a `Form` subclass and create a new instance for your particular controller.

```php
// app/src/Form/SearchForm.php
namespace App\Form;

use SilverStripe\Forms\CheckboxSetField;
use SilverStripe\Forms\CompositeField;
use SilverStripe\Forms\FieldList;
use SilverStripe\Forms\Form;
use SilverStripe\Forms\FormAction;
use SilverStripe\Forms\HeaderField;
use SilverStripe\Forms\NumericField;
use SilverStripe\Forms\OptionsetField;
use SilverStripe\Forms\Validation\RequiredFieldsValidator;

class SearchForm extends Form
{
    /**
     * Our constructor only requires the controller and the name of the form
     * method. We'll create the fields and actions in here.
     */
    public function __construct($controller, $name)
    {
        $fields = FieldList::create(
            HeaderField::create('Header', 'Step 1. Basics'),
            OptionsetField::create('Type', '', [
                'foo' => 'Search Foo',
                'bar' => 'Search Bar',
                'baz' => 'Search Baz',
            ]),
            CompositeField::create(
                HeaderField::create('Header2', 'Step 2. Advanced '),
                CheckboxSetField::create('Foo', 'Select Option', [
                    'qux' => 'Search Qux',
                ]),
                CheckboxSetField::create('Category', 'Category', [
                    'Foo' => 'Foo',
                    'Bar' => 'Bar',
                ]),
                NumericField::create('Minimum', 'Minimum'),
                NumericField::create('Maximum', 'Maximum')
            )
        );

        $actions = FieldList::create(
            FormAction::create('doSearchForm', 'Search')
        );

        $required = RequiredFieldsValidator::create([
            'Type',
        ]);

        // now we create the actual form with our fields and actions defined
        // within this class
        parent::__construct($controller, $name, $fields, $actions, $required);

        // any modifications we need to make to the form.
        $this->setFormMethod('GET');

        $this->addExtraClass('no-action-styles');
        $this->disableSecurityToken();
    }

    /**
     * This method could be on the controller, but putting it here means we get the same
     * behaviour regardless of which controller uses this form.
     */
    public function doSearchForm(array $data, Form $form)
    {
        // Do something with the data, return results, or redirect
    }
}
```

Our controller will now just have to create a new instance of this form object. Keeping the file light and easy to read.

```php
// app/src/PageType/SearchPageController.php
namespace App\PageType;

use App\Form\SearchForm;
use PageController;

class SearchPageController extends PageController
{
    private static $allowed_actions = [
        'searchForm',
    ];

    public function searchForm()
    {
        return SearchForm::create($this, __FUNCTION__);
    }
}
```

## Related documentation

- [Introduction to Forms](../introduction)

## API documentation

- [`Form`](api:SilverStripe\Forms\Form)
