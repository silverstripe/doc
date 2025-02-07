---
title: Model Validation and Constraints
summary: Validate your data at the model level
icon: check-square
---

# Validation and constraints

## Validation using `symfony/validator` constraints {#symfony-validator}

The [`ConstraintValidator`](api:SilverStripe\Core\Validation\ConstraintValidator) class provides an abstraction around [`symfony/validator`](https://symfony.com/doc/current/components/validator.html), so you can easily validate values against symfony's validation constraints and get a [`ValidationResult`](api:SilverStripe\Core\Validation\ValidationResult) object with the result.

```php
use SilverStripe\Core\Validation\ConstraintValidator;

/**
 * @var \Symfony\Component\Validator\Constraint $constraint
 * @var \SilverStripe\Core\Validation\ValidationResult $result
 */
$result = ConstraintValidator::validate($valueToValidate, $constraint);
```

For example, to test if a URL is valid:

```php
use SilverStripe\Core\Validation\ConstraintValidator;
use Symfony\Component\Validator\Constraints\Url;

$isValid = ConstraintValidator::validate($url, new Url())->isValid();
```

You can use most of the constraints listed in Symfony's [supported constraints](https://symfony.com/doc/current/reference/constraints.html) documentation, though note that some of them require additional symfony dependencies.

Validation using constraints that rely on `symfony/doctrine` is explicitly not supported in Silverstripe CMS.

## Model validation

Traditionally, validation in Silverstripe CMS has been mostly handled through [form validation](../forms/validation). While this is a useful approach, it can lead to data inconsistencies if the record is modified outside of the form context.

Most validation constraints are actually data constraints which belong on the model. Silverstripe CMS provides the
[`DataObject::validate()`](api:SilverStripe\ORM\DataObject::validate()) method for this purpose. The `validate()` method is
called any time the `write()` method is called. Implement this method in your `DataObject`
sub-classes to specify custom validation, or use the `updateValidate()` extension hook through an [Extension](api:SilverStripe\Core\Extension).

Invalid objects won't be able to be written - a [`ValidationException`](api:SilverStripe\Core\Validation\ValidationException) will be thrown and no write will occur.

If appropriate, you can call [`ValidationResult::setModelClass()`](api:SilverStripe\Core\Validation\ValidationResult::setModelClass())
and [`ValidationResult::setRecordID()`](api:SilverStripe\Core\Validation\ValidationResult::setRecordID()) on your `ValidationResult` instance to set the class and ID of the object being validated. This additional info will show in the validation error messages in a CLI context, as well as in a non-CLI context if the current controller is an instance or a subclass of a controller configured in the [`ValidationException.show_additional_info_non_cli_controllers`](api:SilverStripe\Core\Validation\ValidationException->show_additional_info_non_cli_controllers) configuration.

The return value of `validate()` is a [`ValidationResult`](api:SilverStripe\Core\Validation\ValidationResult) object.

```php
namespace App\Model;

use SilverStripe\Core\Validation\ValidationResult;
use SilverStripe\ORM\DataObject;

class MyObject extends DataObject
{
    private static $db = [
        'Country' => 'Varchar',
        'Postcode' => 'Varchar',
    ];

    public function validate(): ValidationResult
    {
        $result = parent::validate();

        // This will add a field specific error to the ValidationResult
        if (strlen($this->Postcode) > 10) {
            $result->addFieldError('Postcode', 'Postcode is too long');
        }

        // This will add a general error to the ValidationResult
        if ($this->Country == 'DE' && $this->Postcode && strlen($this->Postcode) !== 5) {
            $result->addError('Need five digits for German postcodes');
        }

        return $result;
    }
}
```

## DBField validation

[`DBField`](api:SilverStripe\ORM\FieldType\DBField) is the base class for all database fields in Silverstripe CMS. For instance when you defined `'MyField' => 'Varchar(255)'` in your [`DataObject`](api:SilverStripe\ORM\DataObject) subclass, the `MyField` property would be an instance of [`DBVarchar`](api:SilverStripe\ORM\FieldType\DBVarchar).

Most `DBField` subclasses will have their values validated as part of `DataObject::validate()`. This means that when a value is set on a `DBField` subclass, it will be validated against the constraints of that field. Field validation is called as part of `DataObject::validate()` which itself is called as part of [`DataObject::write()`](api:SilverStripe\ORM\DataObject::write()). If a value is invalid then a [`ValidationException`](api:SilverStripe\ORM\Validation\ValidationException) will be thrown.

For example, if you have a `Varchar(64)`, and you try to set a value longer than 64 characters, a validation exception will be thrown.

A full list of DBField subclasses and their validation rules can be found in [Data Types and Casting](data_types_and_casting).

## API documentation

- [DataObject](api:SilverStripe\ORM\DataObject)
- [ValidationResult](api:SilverStripe\Core\Validation\ValidationResult);
