---
title: Data types, Overriding and Casting
summary: Learn how data is stored going in and coming out of the ORM and how to modify it.
icon: code
---

# Data types and casting

Each model in a Silverstripe CMS [`DataObject`](api:SilverStripe\ORM\DataObject) will handle data at some point. This includes database columns such as
the ones defined in a `$db` array and methods that return data for for use in templates.

A data type is represented in Silverstripe CMS by a [`DBField`](api:SilverStripe\ORM\FieldType\DBField) subclass. The class is responsible for telling the ORM
how to store its data in the database and how to format the information coming out of the database, i.e. on a template.

In the `Player` example, we have four database columns each with a different data type (`Int`, `Varchar`, etc).

```php
// app/src/Player.php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class Player extends DataObject
{
    private static $db = [
        'PlayerNumber' => 'Int',
        'FirstName' => 'Varchar(255)',
        'LastName' => 'Text',
        'Birthday' => 'Date',
    ];
}
```

Most `DBField` subclasses will be validated using a [`FieldValidator`](api:SilverStripe\Core\Validation\FieldValidation\FieldValidator) subclass which is call as part of the `DataObject::validate()` method. This means that when a value is set on a `DBField` subclass, it will be validated against the constraints of that field. If the value is invalid then a [`ValidationException`](api:SilverStripe\Core\Validation\ValidationException) will be thrown.

## Available types

| Field | API link | Description | Validation |
| --- | --- | --- | --- |
| `BigInt` | [`DBBigInt`](api:SilverStripe\ORM\FieldType\DBBigInt) | An 8-byte signed integer field | Must be an int between -9223372036854775808 and 9223372036854775807 |
| `Boolean` | [`DBBoolean`](api:SilverStripe\ORM\FieldType\DBBoolean) | A boolean field stored as a tinyint | Must be a boolean |
| `Currency` | [`DBCurrency`](api:SilverStripe\ORM\FieldType\DBCurrency) | A number with 2 decimal points of precision, designed to store currency values | Must be a decimal |
| `Date` | [`DBDate`](api:SilverStripe\ORM\FieldType\DBDate) | A date field | Must be a valid date in `Y-m-d` format |
| `Datetime` | [`DBDatetime`](api:SilverStripe\ORM\FieldType\DBDatetime) | A date/time field | Must be a valid datetime in `Y-m-d H:i:s` format |
| `ClassName` | [`DBClassName`](api:SilverStripe\ORM\FieldType\DBClassName) | A special enumeration for storing class names | Must be a valid FQCN of a `DataObject` subclass |
| `ClassNameVarchar` | [`DBClassNameVarchar`](api:SilverStripe\ORM\FieldType\DBClassNameVarchar) | A special enumeration for storing class names in a `Varchar` field | Must be a valid FQCN of a `DataObject` subclass |
| `Decimal` | [`DBDecimal`](api:SilverStripe\ORM\FieldType\DBDecimal) | A decimal number | Must be a decimal. |
| `Double` | [`DBDouble`](api:SilverStripe\ORM\FieldType\DBDouble) | A floating point number with double precision | Must be numeric |
| `Email` | [`DBEmail`](api:SilverStripe\ORM\FieldType\DBEmail) | An email field | Must be a valid email address |
| `Enum` | [`DBEnum`](api:SilverStripe\ORM\FieldType\DBEnum) | An enumeration of a set of strings that can store a single value | Must be one of the defined values |
| `Float` | [`DBFloat`](api:SilverStripe\ORM\FieldType\DBFloat) | A floating point number | Must be numeric |
| `ForeignKey` | [`DBForeignKey`](api:SilverStripe\ORM\FieldType\DBForeignKey) | A special `Int` field used for foreign keys in `has_one` relationships | Must be an int |
| `HTMLFragment` | [`DBHTMLText`](api:SilverStripe\ORM\FieldType\DBHTMLText) | A variable-length string of up to 2MB, designed to store HTML. Doesn't process [shortcodes](/developer_guides/extending/shortcodes/) | Must be a string |
| `HTMLText` | [`DBHTMLText`](api:SilverStripe\ORM\FieldType\DBHTMLText) | A variable-length string of up to 2MB, designed to store HTML. Processes [shortcodes](/developer_guides/extending/shortcodes/) | Must be a string |
| `HTMLVarchar` | [`DBHTMLVarchar`](api:SilverStripe\ORM\FieldType\DBHTMLVarchar) | A variable-length string of up to 255 characters, designed to store HTML. Can process [shortcodes](/developer_guides/extending/shortcodes/) with additional configuration | String must not be longer than specified length |
| `Int` | [`DBInt`](api:SilverStripe\ORM\FieldType\DBInt) | A 32-bit signed integer field | Must be an int between -2147483648 and 2147483647 |
| `IP` | [`DBIp`](api:SilverStripe\ORM\FieldType\DBIp) | An IP field | Must be a valid IP address, either IPv4 or IPv6 |
| `Locale` | [`DBLocale`](api:SilverStripe\ORM\FieldType\DBLocale) | A field for storing locales | Must be a valid locale |
| `Money` | [`DBMoney`](api:SilverStripe\ORM\FieldType\DBMoney) | A localised money field with a 3 character currency component, and an amount component. | Currency string must not be greater than 3 characters, amount must be a decimal |
| `MutliEnum` | [`DBMultiEnum`](api:SilverStripe\ORM\FieldType\DBMultiEnum) | An enumeration set of strings that can store multiple values | Must be one of the allowable values |
| `Percentage` | [`DBPercentage`](api:SilverStripe\ORM\FieldType\DBPercentage) | A decimal number between 0 and 1 that represents a percentage | Must be a decimal between 0 and 1 |
| `PolymorphicForeignKey` | [`DBPolymorphicForeignKey`](api:SilverStripe\ORM\FieldType\DBPolymorphicForeignKey) | A special ForeignKey class that handles relations with arbitrary class types | Must be an int |
| `PrimaryKey` | [`DBPrimaryKey`](api:SilverStripe\ORM\FieldType\DBPrimaryKey) | A special type Int field used for primary keys | Must be an int |
| `Text` | [`DBText`](api:SilverStripe\ORM\FieldType\DBText) | A variable-length string of up to 2MB, designed to store raw text | Must be a string |
| `Time` | [`DBTime`](api:SilverStripe\ORM\FieldType\DBTime) | A time field | Must be a valid time in `H:i:s` format |
| `URL` | [`DBUrl`](api:SilverStripe\ORM\FieldType\DBUrl) | A URL field | Must be a valid URL |
| `Varchar` | [`DBVarchar`](api:SilverStripe\ORM\FieldType\DBVarchar) | A variable-length string of up to 255 characters, designed to store raw text | String must not be longer than specified length |
| `Year` | [`DBYear`](api:SilverStripe\ORM\FieldType\DBYear) | Represents a single year field | Must be a valid year between 1901 and 2155 |

See the [API documentation](api:SilverStripe\ORM\FieldType) for a full list of available data types. You can define your own [`DBField`](api:SilverStripe\ORM\FieldType\DBField) instances if required as well.

## Default values for new database columns {#default-values}

One way to define default values for new records is to use the `$defaults` configuration property, which is described in default in [Dynamic Default Values](how_tos/dynamic_default_fields).
That will only affect *new* records, however. If you are adding a new field to an existing `DataObject` model, you may want to apply a default value for that field to *existing* records as well.

When adding a new `$db` field to a `DataObject`, you can specify a default value
to be applied to all existing and new records when the column is added in the database
for the first time. You do this by passing an argument for the default value in your
`$db` items.

For integer values, the default is the first parameter in the field specification.
For string values, you will need to declare this default using the options array.
For enum values, it's the second parameter.

For example:

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class Car extends DataObject
{
    private static $db = [
        'Wheels' => 'Int(4)',
        'Condition' => 'Enum("New,Fair,Junk", "Fair")',
        'Make' => 'Varchar(["default" => "Honda"])',
    ];
}
```

> [!NOTE]
> `Enum` fields will use the first defined value as the default if you don't explicitly declare one. In the example above, the default value would be "New" if it hadn't been declared.

## Formatting output

The data type does more than set up the correct database schema. They can also define methods and formatting helpers for
output. You can manually create instances of a data type and pass it through to the template.

In this case, we'll create a new method for our `Player` that returns the full name. By wrapping this in a [`DBVarchar`](api:SilverStripe\ORM\FieldType\DBVarchar)
object we can control the formatting and it allows us to call methods defined from `Varchar` as `LimitCharacters`.

```php
// app/src/Model/Player.php
namespace App\Model;

use SilverStripe\ORM\DataObject;
use SilverStripe\ORM\FieldType\DBField;

class Player extends DataObject
{
    public function getName()
    {
        return DBField::create_field('Varchar', $this->FirstName . ' ' . $this->LastName);
    }
}
```

Then we can refer to a new `Name` column on our `Player` instances. In templates we don't need to use the `get` prefix.

```php
use App\Model\Player;

$player = Player::get()->byId(1);

// returns the `DBVarChar` instance, which has the value "Sam Minnée"
$name = $player->Name;

// returns the `DBVarChar` instance, which has the value "Sam Minnée"
$name = $player->getName();

// returns "Sa…"
$name = $player->getName()->LimitCharacters(2);
```

> [!TIP]
> For `DBField` types that represent strings, you can just treat the instance like a string.
>
> ```php
> $player = Player::get()->byId(1);
> // returns the string "Name: Sam Minnée"
> $string = 'Name: ' . $player->Name;
> ```
>
> For other types, we need to make sure we get the value from the `DBField` instance first:
>
> ```php
> $player = Player::get()->byId(1);
> // where `getAge()` returns a `DBInt` field:
> // this will throw a "TypeError: Unsupported operand types: SilverStripe\ORM\FieldType\DBInt + int"
> $player->Age + 5;
> // returns the correct int value as a result
> $player->Age->value + 5;
> ```
>
> That doesn't apply to templates, where we can just treat all `DBField` instances as though they are primitives *or* call methods on them:
>
> ```ss
> <% with $player %>
>     <%-- prints out the name, e.g. Sam Minnée --%>
>     $Name
>     <%-- prints out the name in all caps, e.g. SAM MINNÉE --%>
>     $Name.UpperCase
> <% end_with %>
> ```

On the most basic level, the `DBField` classes can be used for simple conversions from one value to another, e.g. to round a number.

```php
use SilverStripe\ORM\FieldType\DBField;
// returns 1.23
DBField::create_field('Double', 1.23456)->Round(2);
```

Of course that's much more verbose than using the equivalent built-in PHP [`round()`](https://www.php.net/manual/en/function.round.php) function. The power of [DBField](api:SilverStripe\ORM\FieldType\DBField) comes with its more sophisticated helpers, like showing the time difference to the current date:

```php
use SilverStripe\ORM\FieldType\DBField;
// returns "30 years ago"
DBField::create_field('Date', '1982-01-01')->TimeDiff();
```

## Casting

Most objects in Silverstripe CMS extend from [ModelData](api:SilverStripe\Model\ModelData), which means they know how to present themselves in a view
context. Rather than manually returning objects from your custom functions. You can use the `$casting` configuration property. This casting only happens when you get the values in a template, so calling the method in your PHP code will always return the raw value.

> [!TIP]
> While these examples are using `DataObject` subclasses, you can use the `$casting` configuration property on *any* `ModelData` subclass.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class Player extends DataObject
{
    private static $casting = [
        'Name' => 'Varchar',
    ];

    public function getName()
    {
        return $this->FirstName . ' ' . $this->LastName;
    }
}
```

Using this configuration, properties, fields, and methods on any Silverstripe CMS model can be type casted automatically in templates, by transforming its scalar value into an
instance of the [`DBField`](api:SilverStripe\ORM\FieldType\DBField) class, providing additional helpers. For example, a string can be cast as a [`DBText`](api:SilverStripe\ORM\FieldType\DBText)
type, which has a `FirstSentence()` method to retrieve the first sentence in a longer piece of text.

As mentioned above, this leaves you free to use the raw values in your PHP code while giving you all of the helper methods of the `DBField` instances in your templates.

```php
$player = Player::get()->byId(1);

// returns the string "Sam Minnée"
$name = $player->Name;

// returns the string "Sam Minnée"
$name = $player->getName();

// throws an exception, since `getName()` returns a string, not a `DBVarchar` instance
$name = $player->getName()->LimitCharacters(2);
```

```ss
<% with $player %>
    <%-- prints out the name, e.g. Sam Minnée --%>
    $Name
    <%-- prints out the name in all caps, e.g. SAM MINNÉE --%>
    $Name.UpperCase
<% end_with %>
```

You can get the casted `DBField` instance of these properties by calling the [`obj()`](api:SilverStripe\Model\ModelData::obj()) method:

```php
$player = Player::get()->byId(1);
// returns string
$player->getName();
// returns string
$player->Name;
// returns DBVarchar instance
$player->obj('Name');
// returns string
$player->obj('Name')->LimitCharacters(2);
```

The database field types [`DBHTMLVarchar`](api:SilverStripe\ORM\FieldType\DBHTMLVarchar)/[`DBHTMLText`](api:SilverStripe\ORM\FieldType\DBHTMLText) and [`DBVarchar`](api:SilverStripe\ORM\FieldType\DBVarchar)/[`DBText`](api:SilverStripe\ORM\FieldType\DBText) are exactly the same in
the database. However, the template engine knows to escape the non-HTML variants automatically in templates,
to prevent them from rendering HTML interpreted by browsers. This escaping prevents attacks like CSRF or XSS (see
[security](../security)), which is important if these fields store user-provided data.

See the [Template casting](/developer_guides/templates/casting) section for controlling casting in your templates.

## Overriding

"Getter" and "setter" methods help save fields to our [`DataObject`](api:SilverStripe\ORM\DataObject) instances. By default, the
methods `getField()` and `setField()` are used to set column data.  They save to the protected array, `$obj->record`.
We can override the default behavior by making a method called "`get<fieldname>()`" or "`set<fieldname>()`".

The following example will use the result of `getCost()` instead of the `Cost` database column. We can refer to the
database column using `getField()`.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

/**
 * @property float $Cost
 */
class Product extends DataObject
{
    private static $db = [
        'Title' => 'Varchar(255)',
        //cost in pennies/cents
        'Cost' => 'Int',
    ];

    public function getCost()
    {
        return $this->getField('Cost') / 100;
    }

    public function setCost($value)
    {
        return $this->setField('Cost', $value * 100);
    }
}
```

> [!TIP]
> Note that in the example above we've used a PHPDoc comment to indicate that the `$Cost` property is a `float`, even though the database field type is `Int`. This is because the `getCost()` getter method will automatically be used when trying to access `Cost` as a property (i.e. `$product->Cost` will return the result of `$product->getCost()`).

## API documentation

- [DataObject](api:SilverStripe\ORM\DataObject)
- [DBField](api:SilverStripe\ORM\FieldType\DBField)
