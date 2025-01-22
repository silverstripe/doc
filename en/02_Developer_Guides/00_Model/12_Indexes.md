---
title: Indexes
summary: Add Indexes to your Data Model to optimize database queries.
icon: database
---

# Indexes

Indexes are a great way to improve performance in your application, especially as it grows. By adding indexes to your
data model you can reduce the time taken for the framework to find and filter data objects.

The addition of an indexes should be carefully evaluated as they can also increase the cost of other operations such as
`UPDATE`/`INSERT` and `DELETE`. An index on a column whose data is non unique will actually cost you performance.
For example, in most cases an index on `boolean` status flag, or `ENUM` state will not increase query performance.

It's important to find the right balance to achieve fast queries using the optimal set of indexes; For Silverstripe CMS
applications it's a good practice to:

- add indexes on columns which are frequently used in `filter`, `where` or `orderBy` statements
- for these, only include indexes for columns which are the most restrictive (return the least number of rows)

The Silverstripe CMS framework already places certain indexes for you by default:

- The primary key for each model has a `PRIMARY KEY` unique index
- The `ClassName` column if your model inherits from `DataObject`
- All relationships defined in the model have indexes for their `has_one` entity (for `many_many` relationships
this index is present on the associative entity).
- All fields used in `default_sort` configuration

## Defining an index

Indexes are represented on a `DataObject` through the [`DataObject.indexes`](api:SilverStripe\ORM\DataObject->indexes) configuration property which maps index names to a
descriptor. There are several supported notations:

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyObject extends DataObject
{
    private static $indexes = [
        '<column-name>' => true,
        '<index-name>' => [
            'type' => '<type>',
            'columns' => ['<column-name>', '<other-column-name>'],
        ],
        '<index-name>' => ['<column-name>', '<other-column-name>'],
    ];
}
```

The `<column-name>` is used to put a standard non-unique index on the column specified. For complex or large tables
we recommend building the index to suite the requirements of your data.

The `<index-name>` can be an arbitrary identifier in order to allow for more than one index on a specific database
column. The "advanced" notation supports more `<type>` notations. These vary between database drivers, but all of them
support the following:

- `index`: Standard non unique index.
- `unique`: Index plus uniqueness constraint on the value
- `fulltext`: Fulltext content index

> [!NOTE]
> Violating a unique index will throw a [`DuplicateEntryException`](api:SilverStripe\ORM\Connect\DuplicateEntryException) exception which you can catch and handle to produce appropriate validation messages.
>
> If the violation happens when calling [`DataObject::write()`](api:SilverStripe\ORM\DataObject::write()), the exception will be caught and a [`ValidationException`](api:SilverStripe\ORM\ValidationException) will be thrown instead. The CMS catches any `ValidationException` and displays them as user friendly validation errors in edit forms.

```php
// app/src/MyTestObject.php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyTestObject extends DataObject
{
    private static $db = [
        'MyField' => 'Varchar',
        'MyOtherField' => 'Varchar',
    ];

    private static $indexes = [
        'MyIndexName' => ['MyField', 'MyOtherField'],
    ];
}
```

## Complex/Composite indexes

For complex queries it may be necessary to define a complex or composite index on the supporting object. To create a
composite index, define the fields in the index order as a comma separated list.

- index (col1) - `WHERE col1 = ?`
- index (col1, col2) = `WHERE (col1 = ? AND col2 = ?)`
- index (col1, col2, col3) = `WHERE (col1 = ? AND col2 = ? AND col3 = ?)`

The index would not be used for a query `WHERE col2 = ?` or for `WHERE col1 = ? OR col2 = ?`

As an alternative to a composite index, you can also create a hashed column which is a combination of information from
other columns. If this is indexed, smaller and reasonably unique it might be faster that an index on the whole column.

## Index creation/destruction

Indexes are generated and removed automatically when building the database. Caution if you're working with large tables and
modify an index as the next time the database is built it will `DROP` the index, and then `ADD` it.

## API documentation

- [DataObject](api:SilverStripe\ORM\DataObject)
