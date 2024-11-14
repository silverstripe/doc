---
title: Introduction to the Data Model and ORM
summary: Introduction to creating and querying a database records through the ORM (object-relational mapping)
icon: database
---

# Introduction to the data model and ORM

Silverstripe uses an [object-relational mapping](https://en.wikipedia.org/wiki/Object-relational_mapping) (aka "ORM") to represent its
information.

- Each database table maps to a PHP class.
- Each database row maps to a PHP object.
- Each database column maps to a property on a PHP object.

All data tables in Silverstripe CMS are defined as subclasses of [DataObject](api:SilverStripe\ORM\DataObject). The [DataObject](api:SilverStripe\ORM\DataObject) class represents a
single row in a database table, following the ["Active Record"](https://en.wikipedia.org/wiki/Active_record_pattern)
design pattern. Database Columns are defined as [Data Types](/developer_guides/model/data_types_and_casting) in the static `$db` variable
along with any [relationships](relations) defined as `$has_one`, `$has_many`, `$many_many` properties on the class.

Let's look at a simple example:

```php
// app/src/Model/Player.php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class Player extends DataObject
{
    private static $class_description = 'Sports player';

    private static $table_name = 'Player';

    private static $db = [
        'PlayerNumber' => 'Int',
        'FirstName' => 'Varchar(255)',
        'LastName' => 'Text',
        'Birthday' => 'Date',
        'Status' => 'Varchar(255)',
    ];
}
```

This `Player` class definition will create a database table `Player` with columns for `PlayerNumber`, `FirstName` and
so on. After writing this class, we need to regenerate the database schema.

> [!TIP]
> You can technically omit the `table_name` property, and a default table name will be created based on the fully qualified class name - but this can result in table names that are too long for the database engine to handle. We recommend that you *always* explicitly declare a table name for your models.
>
> See more in [Mapping classes to tables with `DataObjectSchema`](#mapping-classes-to-tables) below.

## Generating the database schema

After adding, modifying or removing `DataObject` subclasses, make sure to rebuild your Silverstripe CMS database. The
database schema is generated automatically by visiting `/dev/build` (e.g. `https://www.example.com/dev/build`) in your browser
while authenticated as an administrator, or by running `sake db:build` on the command line (see [Sake](/developer_guides/cli/sake/) to learn more about using Sake).

> [!NOTE]
> In "dev" mode, you do not need to be authenticated to visit `/dev/build`. See [Environment Types](/developer_guides/debugging/environment_types) for more information.

This script will analyze the existing schema, compare it to what's required by your data classes, and alter the schema
as required.

It will perform the following changes:

- Create any missing tables
- Create any missing field columns
- Create any missing indexes
- Alter the field type of any existing fields
- Rename any obsolete tables that it previously created to `_obsolete_tablename` (e.g. `_obsolete_player`)
  - Obsolete tables are only renamed if the `DataObject` model which owns the table has no need for the table (e.g. no fields are declared for the model, and it is a subclass of some other `DataObject`).

It **won't** do any of the following

- Delete tables
- Delete field columns
- Rename any tables that it doesn't recognize. This allows other applications to coexist in the same database, as long as
  their table names don't match a Silverstripe CMS data class.

When rebuilding the database schema through the [ClassLoader](api:SilverStripe\Core\Manifest\ClassLoader) the following additional fields are
automatically set on the `DataObject`.

- `ID`: Primary Key. This will use the database's built-in auto-numbering system on the base table, and apply the same ID to all subclass tables.
- `ClassName`: An enumeration listing this data-class and all of its subclasses. The value is the actual `DataObject` subclass used to write the record.
- `Created`: A date/time field set to the creation date (i.e. when it was first written to the database) of this record
- `LastEdited`: A date/time field set to the date this record was last edited through `write()`

The table creation SQL statement for our `Player` model above looks like this:

```sql
CREATE TABLE `Player` (
    `ID` int(11) NOT NULL AUTO_INCREMENT,
    `ClassName` enum('Player') DEFAULT 'Player',
    `LastEdited` datetime DEFAULT NULL,
    `Created` datetime DEFAULT NULL,
    `PlayerNumber` int(11) NOT NULL DEFAULT '0',
    `FirstName` varchar(255) DEFAULT NULL,
    `LastName` mediumtext,
    `Birthday` datetime DEFAULT NULL,
    `Status` varchar(255) DEFAULT NULL,

    PRIMARY KEY (`ID`),
    KEY `ClassName` (`ClassName`)
);
```

## Creating data records

A new instance of a [DataObject](api:SilverStripe\ORM\DataObject) can be created using the `new` keyword.

```php
$player = new Player();
```

However, a better way is to use the `create()` method.

```php
$player = Player::create();
```

> [!TIP]
> Using the `create()` method provides chainability (known as a "fluent API" or "[fluent interface](https://en.wikipedia.org/wiki/Fluent_interface)"), which can add elegance and brevity to your code, e.g. `Player::create(['FirstName' => 'Sam'])->write()`.
>
> More importantly, however, it will look up the class in the [`Injector`](api:SilverStripe\Core\Injector\Injector) so that the class can be overridden by [dependency injection](../extending/injector). For this reason, instantiating records using the `new` keyword is considered bad practice.

Database columns (aka fields) can be set as class properties on the object. The Silverstripe CMS ORM handles the saving
of the values through a custom `__set()` method.

```php
$player->FirstName = 'Sam';
$player->PlayerNumber = 07;
```

To save the `DataObject` to the database, use the `write()` method. The first time `write()` is called, an `ID` will be
set.

```php
$player->write();
```

For convenience, the `write()` method returns the record's ID. This is particularly useful when creating new records.

```php
$player = Player::create();
$id = $player->write();
```

## Querying data

With the `Player` class defined we can query our data using the ORM. The ORM provides
shortcuts and methods for fetching, sorting and filtering data from our database.

> [!TIP]
> All of the below methods to get a single record will return `null` if there is no record to return.

```php
// returns a `DataList` containing all the `Player` objects.
$players = Player::get();

// returns the first and last `Player` record in the list, respectively.
$firstPlayer = $players->first();
$lastPlayer = $players->last();

// returns a single `Player` record that has the ID of 2.
$player = Player::get()->byID(2);
$player = Player::get_by_id(2);
```

> [!NOTE]
> `DataObject::get()->byID()` and `DataObject::get_by_id()` achieve similar results, though the object returned by `DataObject::get_by_id()` is cached against a `static` property within `DataObject`.

The ORM uses a "fluent" syntax, where you specify a query by chaining together different methods.  Two common methods
are `filter()` and `sort()`:

```php
// returns a `DataList` containing all the `Player` records that have the `FirstName` of 'Sam'
$members = Player::get()->filter([
    'FirstName' => 'Sam',
])->sort('Surname');
```

There's a lot more to filtering and sorting, so make sure to keep reading.

> [!NOTE]
> Values passed in to the `filter()` and `sort()` methods are automatically escaped and do not require any additional escaping. This makes it easy to safely filter/sort records by user input.

### Querying data when you have a record

When you have a `DataObject` record, it already has all of its database field data attached to it. You can get those values without triggering further database queries.

> [!WARNING]
> This does not apply to relations, which are always lazy loaded. See the [Relations between Records](relations) documentation for more information.

```php
$player = Player::get()->byID(2);

// returns the players' `ID` field value
$id = $player->ID;

// gets the `LastEdited` field value in its default format
$lastEdited = $player->LastEdited;
// calls the `Ago` method on the `LastEdited` field.
$timeSinceLastEdit = $player->dbObject('LastEdited')->Ago();
```

All database fields have a default value format which you can retrieve by treating the field as a class property - but there are also other formats available depending on the field type. You'll learn more about those in the [Data Types and Casting](data_types_and_casting) section - but the important thing to know is that you can get the `DBField` instance for a field by calling `dbObject('FieldName')` on the record - and that `DBField` instance will have different methods on it depending on the data type that let you access different formats of the field's value.

## Lazy loading

The ORM doesn't actually execute the [SQLSelect](api:SilverStripe\ORM\Queries\SQLSelect) query until you iterate on the result (e.g. with a `foreach()` or `<% loop %>`).

> [!TIP]
> Some convenience methods (e.g. [`column()`](api:SilverStripe\ORM\DataList::column()) or aggregator methods like [`min()`](api:SilverStripe\ORM\DataList::min())) will also execute the query.

It's smart enough to generate a single efficient query at the last moment in time without needing to post-process the
result set in PHP. In `MySQL` the query generated by the ORM may look something like this

```php
$players = Player::get()->filter([
    'FirstName' => 'Sam',
]);

$players = $players->sort('Surname');

// executes the following single query
// SELECT * FROM Player WHERE FirstName = 'Sam' ORDER BY Surname
```

This also means that getting the count of a list of objects will be done with a single, efficient query.

```php
$players = Player::get()->filter([
    'FirstName' => 'Sam',
])->sort('Surname');

// This will create an single SELECT COUNT query
// SELECT COUNT(*) FROM Player WHERE FirstName = 'Sam'
echo $players->Count();
```

## Looping over a list of objects

[`get()`](api:SilverStripe\ORM\DataObject::get()) returns a [`DataList`](api:SilverStripe\ORM\DataList) instance. You can loop over `DataList` instances in both PHP and templates.

```php
$players = Player::get();

foreach ($players as $player) {
    echo $player->FirstName;
}
```

Notice that we can step into the loop safely without having to check if `$players` exists. The `get()` call is robust, and will at worst return an empty `DataList` object. But if you do want to check if the query returned any records, you can use the `exists()` method, e.g.

```php
$players = Player::get();

if ($players->exists()) {
    // do something here
}
```

> [!TIP]
> While you could use `if ($players->Count() > 0)` for this condition, the `exists()` method uses an `EXISTS` SQL query, which is more performant.

See the [Lists](lists) documentation for more information on dealing with [SS_List](api:SilverStripe\Model\List\SS_List) instances.

## Sorting

If you would like to sort the list by `FirstName` in an ascending way (from A to Z).

```php
 // Sort can either be Ascending (ASC) or Descending (DESC)
$players = Player::get()->sort('FirstName', 'ASC');

 // Ascending is implied
$players = Player::get()->sort('FirstName');
```

To reverse the sort

```php
$players = Player::get()->sort('FirstName', 'DESC');

// or..
$players = Player::get()->sort('FirstName', 'ASC')->reverse();
```

However you might have several entries with the same `FirstName` and would like to sort them by `FirstName` and
`LastName`

```php
$players = Players::get()->sort([
    'FirstName' => 'ASC',
    'LastName' => 'ASC',
]);
```

You can also sort randomly.

```php
$players = Player::get()->shuffle();
```

## Filtering results

The `filter()` method filters the list of objects that gets returned.

```php
$players = Player::get()->filter([
    'FirstName' => 'Sam',
]);
```

Each element of the array specifies a filter. You can specify as many filters as you like, and they **all** must be
true for the record to be included in the result.

The key in the filter corresponds to the field that you want to filter and the value in the filter corresponds to the
value that you want to filter to.

So, this would return only those players called "Sam Minnée":

```php
// SELECT * FROM Player WHERE FirstName = 'Sam' AND LastName = 'Minnée'
$players = Player::get()->filter([
    'FirstName' => 'Sam',
    'LastName' => 'Minnée',
]);
```

There is also a shorthand way of getting Players with a specific value for any given field:

```php
$players = Player::get()->filter('FirstName', 'Sam');
```

Or if you want to find both Sam and Sig:

```php
// SELECT * FROM Player WHERE FirstName IN ('Sam', 'Sig')
$players = Player::get()->filter('FirstName', ['Sam', 'Sig']);
```

You can use an array of values when passing an array of filters in as the first argument as well, e.g:

```php
// SELECT * FROM Player WHERE FirstName = 'Sam' AND LastName in ('Minnée', 'Carter')
$players = Player::get()->filter([
    'FirstName' => 'Sam',
    'LastName' => 'Minnée',
]);
```

You can use a `SearchFilter` to add additional behavior to your `filter` command rather than an
exact match. See [SearchFilter Modifiers](searchfilters) for more information.

```php
$players = Player::get()->filter([
    'FirstName:StartsWith' => 'S',
    'PlayerNumber:GreaterThan' => '10',
]);
```

### `filterAny`

Use the `filterAny()` method to match multiple criteria non-exclusively (with an "OR" disjunctive),

```php
// SELECT * FROM Player WHERE ("FirstName" = 'Sam' OR "Age" = '17')
$players = Player::get()->filterAny([
    'FirstName' => 'Sam',
    'Age' => 17,
]);
```

You can combine both conjunctive ("AND") and disjunctive ("OR") statements.

```php
// SELECT * FROM Player WHERE ("LastName" = 'Minnée' AND ("FirstName" = 'Sam' OR "Age" in ('17', '18')))
$players = Player::get()
    ->filter([
        'LastName' => 'Minnée',
    ])
    ->filterAny([
        'FirstName' => 'Sam',
        'Age' => [17, 18],
    ]);
```

You can use [SearchFilters](searchfilters) to add additional behavior to your `filterAny` command.

```php
$players = Player::get()->filterAny([
    'FirstName:StartsWith' => 'S',
    'PlayerNumber:GreaterThan' => '10',
]);
```

### Filtering by null values

Since null values in SQL are special, they are non-comparable with other values. Certain filters will add
`IS NULL` or `IS NOT NULL` predicates automatically to your query. As per [ANSI SQL-92](https://en.wikipedia.org/wiki/SQL-92), any comparison
condition against a field will filter out nulls by default. Therefore, it's necessary to include certain null
checks to ensure that exclusion filters behave predictably.

For instance, the below code will select only values that do not match the given value, including nulls.

```php
// ... WHERE "FirstName" != 'Sam' OR "FirstName" IS NULL
// Returns rows with any value (even null) other than Sam
$players = Player::get()->filter('FirstName:not', 'Sam');
```

If null values should be excluded, include the null in your check.

```php
// ... WHERE "FirstName" != 'Sam' AND "FirstName" IS NOT NULL
// Only returns non-null values for "FirstName" that aren't Sam.
// Strictly the IS NOT NULL isn't necessary in the resulting query, but is included for explicitness
$players = Player::get()->filter('FirstName:not', ['Sam', null]);
```

It is also often useful to filter by all rows with either empty or null for a given field.

```php
// ... WHERE "FirstName" == '' OR "FirstName" IS NULL
// Returns rows with FirstName which is either empty or null
$players = Player::get()->filter('FirstName', [null, '']);
```

### Filtering by aggregates

You can use aggregate expressions in your filters, as well.

```php
// get the teams that have more than 10 players
$teams = Team::get()->filter('Players.Count():GreaterThan', 10);

// get the teams with at least one player who has scored 5 or more points
$teams = Team::get()->filter('Players.Min(PointsScored):GreaterThanOrEqual', 5);

// get the teams with players who are averaging more than 15 points
$teams = Team::get()->filter('Players.Avg(PointsScored):GreaterThan', 15);

// get the teams whose players have scored less than 300 points combined
$teams = Team::get()->filter('Players.Sum(PointsScored):LessThan', 300);
```

> [!TIP]
> The above examples are using "dot notation" to get the aggregations of the `Players` relation on the `Teams` model. See [Relations between Records](relations) to learn more.

### `filterByCallback`

It is possible to filter by a PHP callback using the [`filterByCallback()`](api:SilverStripe\ORM\DataList::filterByCallback()) method. This will force the data model to fetch all records and loop them in
PHP which will be much worse for performance, thus `filter()` or `filterAny()` are to be preferred over `filterByCallback()`.

> [!WARNING]
> Because `filterByCallback()` has to run in PHP, it has a significant performance tradeoff, and should not be used on large recordsets.
>
> `filterByCallback()` will always return an `ArrayList`.

The first parameter to the callback is the record, the second parameter is the list itself. The callback will run once
for each record. The callback must return a boolean value. If the callback returns true, the current record will be included in the list of returned items.

The below example will get all `Player` records aged over 10.

```php
$players = Player::get()->filterByCallback(function ($record, $list) {
    return ($record->Age() > 10);
});
```

### `exclude`

The [`exclude()`](api:SilverStripe\ORM\DataList::exclude()) method is the opposite to `filter()` in that it determines which entries to *exclude* from a list, where `filter()` determines which to *include*.

```php
// SELECT * FROM Player WHERE FirstName != 'Sam'
$players = Player::get()->exclude('FirstName', 'Sam');
```

Exclude both Sam and Sig.

```php
$players = Player::get()->exclude([
    'FirstName' => ['Sam', 'Sig'],
]);
```

`exclude()` follows the same pattern as filter, so for excluding anyone with both the first name "Sam" and the last name "Minnée" from the list:

```php
// SELECT * FROM Player WHERE (FirstName != 'Sam' OR LastName != 'Minnée')
$players = Player::get()->exclude([
    'FirstName' => 'Sam',
    'Surname' => 'Minnée',
]);
```

Removing players with *either* the first name of "Sam" or the last name of "Minnée" requires multiple `exclude()` calls - or simply using the `excludeAny()` method:

```php
// SELECT * FROM Player WHERE FirstName != 'Sam' AND LastName != 'Minnée'
$players = Player::get()->exclude('FirstName', 'Sam')->exclude('Surname', 'Minnée');
$players = Player::get()->excludeAny([
    'FirstName' => 'Sam',
    'Surname' => 'Minnée',
]);
```

And removing any named "Sig" or "Sam" with that are either age 17 or 43.

```php
// SELECT * FROM Player WHERE ("FirstName" NOT IN ('Sam','Sig) OR "Age" NOT IN ('17', '43'));
$players = Player::get()->exclude([
    'FirstName' => ['Sam', 'Sig'],
    'Age' => [17, 43],
]);
```

You can use [SearchFilters](searchfilters) to add additional behavior to your `exclude` command.

```php
$players = Player::get()->exclude([
    'FirstName:EndsWith' => 'S',
    'PlayerNumber:LessThanOrEqual' => '10',
]);
```

### `subtract`

You can subtract entries from a [DataList](api:SilverStripe\ORM\DataList) by passing in another DataList to `subtract()`

```php
$sam = Player::get()->filter('FirstName', 'Sam');
$noSams = Player::get()->subtract($sam);
```

Though for the above example it would probably be easier to use `filter()` and `exclude()` directly on the final list. A better use case could be
when you want to find all the members that do not exist in a Group.

```php
// ... Finding all members that do not belong to $group.
use SilverStripe\Security\Member;
// Assuming we have some `Group` $group:
$otherMembers = Member::get()->subtract($group->Members());
```

### `limit`

You can limit the amount of records returned in a DataList by using the `limit()` method.

```php
use SilverStripe\Security\Member;
$members = Member::get()->limit(5);
```

`limit()` accepts two arguments, the first being the amount of results you want returned, with an optional second
parameter to specify the offset, which allows you to tell the system where to start getting the results from. The
offset, if not provided as an argument, will default to 0 (i.e. start with the first result).

```php
// Return 10 members with an offset of 4 (starting from the 5th result).
$members = Member::get()->sort('Surname')->limit(10, 4);
```

### Mapping classes to tables with `DataObjectSchema` {#mapping-classes-to-tables}

Note that by default, the underlying database table for any `DataObject` instance will be the same as the class name.
However, relying on this default behaviour can result in table names that are too long for the database engine to support.
We recommend explicitly declaring a table name for each `DataObject` subclass you create.

For instance, the below model will be stored in the table name `BannerImage`

```php
namespace SilverStripe\BannerManager;

use SilverStripe\ORM\DataObject;

class BannerImage extends DataObject
{
    private static $table_name = 'BannerImage';
}
```

Note that any model class which does not explicitly declare a `table_name` config option will have a name
automatically generated for them. In the above case, the table name would have been
`SilverStripe_BannerManager_BannerImage`

When creating raw SQL queries that contain table names, it is necessary to ensure your queries have the correct
table. This functionality can be provided by the [DataObjectSchema](api:SilverStripe\ORM\DataObjectSchema) service, which can be accessed via
`DataObject::getSchema()`. This service provides the following methods, most of which have a table and class
equivalent version.

Methods which return class names:

- [`tableClass($table)`](api:SilverStripe\ORM\DataObjectSchema::tableClass()) - Finds the class name for a given table. This also handles suffixed tables such as `Table_Live` (see [Versioning](versioning)).
- [`baseDataClass($class)`](api:SilverStripe\ORM\DataObjectSchema::baseDataClass()) - Returns the base data class for the given class.
- [`classForField($class, $field)`](api:SilverStripe\ORM\DataObjectSchema::classForField()) - Finds the specific class that directly holds the given field

Methods which return table names:

- [`tableName($class)`](api:SilverStripe\ORM\DataObjectSchema::tableName()) - Returns the table name for a given class or object.
- [`baseDataTable($class)`](api:SilverStripe\ORM\DataObjectSchema::baseDataTable()) - Returns the base data class for the given class.
- [`tableForField($class, $field)`](api:SilverStripe\ORM\DataObjectSchema::tableForField()) - Finds the specific class that directly holds the given field and returns the table.

Note that in cases where the class name is required, an instance of the object may be substituted.

For example, if running a query against a particular model, you will need to ensure you use the correct
table and column.

```php
use SilverStripe\ORM\DataObject;
use SilverStripe\ORM\Queries\SQLSelect;

public function countDuplicates($model, $fieldToCheck)
{
    $table = DataObject::getSchema()->tableForField($model, $field);
    $query = SQLSelect::create();
    $query->setFrom("\"{$table}\"");
    $query->setWhere(["\"{$table}\".\"{$field}\"" => $model->$fieldToCheck]);
    return $query->count();
}
```

### Common table expressions (CTE aka the `WITH` clause) {#cte}

Common Table Expressions are a powerful tool both for optimising complex queries, and for creating recursive queries. You can use these by calling the [`DataQuery::with()`](api:SilverStripe\ORM\DataQuery::with()) method.

Note that there is no direct abstraction for this on `DataList`, so you'll need to [modify the underlying `DataQuery`](#modifying-the-underlying-query) to apply a CTE to a `DataList`.

Older database servers don't support this functionality, and the core implementation is only valid for MySQL (though community modules may add support for other database connectors). You can esure this code will only be used when it's supported by wrapping it in a conditional call to [`DB::get_conn()->supportsCteQueries()`](api:SilverStripe\ORM\Connect\Database::supportsCteQueries()). See the [SQL Queries](/developer_guides/model/sql_select/#cte) documentation for more details.

The following example is the equivalent to the example in the [SQL Queries](/developer_guides/model/sql_select/#cte) documentation, except it is modifying the underlying query of a `DataList`. This means we are effectively filtering the `DataList` to include only records which are ancestors of the `$someRecord` record.

```php
use App\Model\ObjectWithParent;
use SilverStripe\Core\Convert;
use SilverStripe\ORM\DB;
use SilverStripe\ORM\DataObject;
use SilverStripe\ORM\DataQuery;

// Only use the CTE functionality if it is supported by the current database
if (DB::get_conn()->supportsCteQueries(true)) {
    $ancestors = ObjectWithParent::get()->alterDataQuery(function (DataQuery $baseQuery) use ($someRecord) {
        $schema = DataObject::getSchema();
        $parentIdField = $schema->sqlColumnForField(ObjectWithParent::class, 'ParentID');
        $idField = $schema->sqlColumnForField(ObjectWithParent::class, 'ID');
        $cteIdField = Convert::symbol2sql('hierarchy_cte.ParentID');

        $cteQuery = new DataQuery(ObjectWithParent::class);
        $cteQuery->where([
            "$parentIdField > 0",
            $idField => $someRecord->ID,
        ]);
        $recursiveQuery = new DataQuery(ObjectWithParent::class);
        $recursiveQuery->innerJoin('hierarchy_cte', "$idField = $cteIdField")
            ->where("$parentIdField > 0")
            // MySQL doesn't support ORDER BY or DISTINCT in the recursive portion of a CTE
            ->sort(null)
            ->distinct(false);
        $cteQuery->union($recursiveQuery);
        $baseQuery->with('hierarchy_cte', $cteQuery, [], true);
        $baseQuery->innerJoin('hierarchy_cte', "$idField = $cteIdField");
        // This query result will include only the ancestors of whatever record is stored in the $someRecord variable.
        return $baseQuery;
    });
} else {
    // provide an alternative implementation, e.g. a recursive PHP method which runs a query at each iteration
}
```

The PHPDoc for the [`DataQuery::with()`](api:SilverStripe\ORM\DataQuery::with()) method has more details about what each of the arguments are and how they're used, though note that you should ensure you understand the underlying SQL concept of CTE queries before using this API.

#### Gotchas {#cte-gotchas}

There are a few things that might catch you off guard with this abstraction if you aren't looking for them. Many of these are specifically enforced by MySQL and may not apply to other databases.

- `DataQuery` wants to use `DISTINCT` and to apply a sort order by default. MySQL 8 doesn't support `ORDER BY`, `DISTINCT`, or `LIMIT` in the recursive query block of Common Table Expressions so we need to make sure to explicitly set the sort order to null and distinct to false when using a `DataQuery` for that part of the query.
- If you use `DataQuery` for `$cteQuery` (i.e. the `$query` argument of the `with()` method), you can reduce the fields being selected by including them in the `$cteFields` argument. Be aware though that the number of fields passed in must match the number used in the recursive query if your CTE is recursive.
  - `$cteFields` will be used to set the select fields for the `$cteQuery` if it's a `DataQuery` - but if it's a `SQLSelect` then this argument works the same as it does with `SQLSelect::addWith()`.

### Raw SQL

Occasionally, the system described above won't let you do exactly what you need to do. In these situations, we have
methods that manipulate the SQL query at a lower level. When using these, please ensure that all table and field names
are escaped with double quotes, otherwise some database backends (e.g. [PostgreSQL](https://github.com/silverstripe/silverstripe-postgresql)) won't work.

#### Modifying the underlying query

Under the hood, query generation is handled by the [DataQuery](api:SilverStripe\ORM\DataQuery) class. This class provides more direct access
to certain SQL features that `DataList` abstracts away from you. You can modify the underlying `DataQuery` by calling
the [`alterDataQuery()`](api:SilverStripe\ORM\DataList::alterDataQuery()) method.

This can be useful for accessing abstractions that exist on the `DataQuery` layer but aren't available at the `DataList` layer.

```php
$members = Member::get()->alterDataQuery(function (DataQuery $query) {
    return $query->union($anotherQuery);
});
```

#### Using raw SQL directly

In general, we advise against using these methods unless it's absolutely necessary. If the ORM doesn't do quite what
you need it to, you may also consider extending the ORM with new data types or filter modifiers

##### Where clauses

You can specify a WHERE clause fragment (that will be combined with other filters using AND) with the `where()` method:

```php
$members = Member::get()->where("\"FirstName\" = 'Sam'");
```

##### Order by clauses

You can specify an ORDER BY clause fragment with the `orderBy` method:

```php
$members = Member::get()->orderBy(/* some raw SQL here */);
```

##### Joining tables

You can specify a join with the `innerJoin`, `leftJoin`, and `rightJoin` methods. All of these methods have the same arguments:

- The name of the table to join to.
- The filter clause for the join.
- An optional alias.

```php
// Without an alias
$members = Member::get()
    ->leftJoin("Group_Members", '"Group_Members"."MemberID" = "Member"."ID"');
$members = Member::get()
    ->rightJoin("Group_Members", '"Group_Members"."MemberID" = "Member"."ID"');
$members = Member::get()
    ->innerJoin("Group_Members", '"Group_Members"."MemberID" = "Member"."ID"');

// With an alias "Rel"
$members = Member::get()
    ->leftJoin("Group_Members", '"Rel"."MemberID" = "Member"."ID"', "Rel");
$members = Member::get()
    ->rightJoin("Group_Members", '"Rel"."MemberID" = "Member"."ID"', "Rel");
$members = Member::get()
    ->innerJoin("Group_Members", '"Rel"."MemberID" = "Member"."ID"', "Rel");
```

> [!CAUTION]
> Using a join will *filter* results further by the JOINs performed against the foreign table. It will
> **not return** the additionally joined data. For the examples above, we're still only selecting values for the fields
> on the `Member` class table.

### Default values

Define the default values for all the `$db` fields. This example sets the `Status` column on Player to "Active"
whenever a new record is created.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class Player extends DataObject
{
    // ...
    private static $defaults = [
        'Status' => 'Active',
    ];
}
```

See [Default Values and Records](/developer_guides/model/how_tos/dynamic_default_fields/) for more about setting default values and records.

## Subclasses

Inheritance is supported in the data model. Separate tables will be linked together, the data spread across these
tables depending on which class declares them. The mapping and saving logic is handled by Silverstripe CMS - you don't need to worry about writing SQL most of the
time.

For example, suppose we have the following set of classes:

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class Product extends DataObject
{
    private static $table_name = 'Product';

    private static $db = [
        'SKU' => 'Text',
    ];
}
```

```php
namespace App\Model;

class DigitalProduct extends Product
{
    private static $table_name = 'Product_Digital';
}
```

```php
namespace App\Model;

class Computer extends DigitalProduct
{
    private static $table_name = 'Product_Digital_Computer';

    private static $db = [
        'IsPreBuilt' => 'Boolean',
    ];
}
```

The data for the following classes would be stored across the following tables:

```yml
Product:
  ID: Int
  ClassName: Enum('Sport', 'BallSport', 'Tennis')
  Created: Datetime
  LastEdited: Datetime
  SKU: Text
Product_Digital_Computer:
  ID: Int
  IsPreBuilt: 'Boolean'
```

> [!TIP]
> Note that because `DigitalProduct` doesn't define any new fields it doesn't need its own table. We should still declare a `$table_name` though - who knows if this model might have its own table created in the future (e.g. if we add fields to it later on).

Accessing the data is transparent to the developer.

```php
$products = Computer::get();

foreach ($products as $product) {
    echo $product->SKU;
}
```

The way the ORM stores the data is this:

- "Base classes" are direct sub-classes of [DataObject](api:SilverStripe\ORM\DataObject). They are always given a table, whether or not they declare
   their own fields. This is called the "base table". In our case, `Product` is the base table.
- The base table's `ClassName` field is set to class of the given record. The column is an enumeration of all
   subclasses of the base class (including the base class itself).
- Each subclass of the base object will also be given its own table *as long as it has custom fields*. In the
   example above, `DigitalProduct` didn't define any new fields, so an extra table would be redundant.
- In all the tables, `ID` is the primary key. A matching `ID` number is used for all parts of a particular record:
   record #2 in the `Product` table refers to the same object as record #2 in the `Product_Digital_Computer` table.

To retrieve a `Computer` record, Silverstripe CMS joins the `Product` and `Product_Digital_Computer` tables by their `ID` columns.

## Related documentation

- [Data Types and Casting](/developer_guides/model/data_types_and_casting)

## API documentation

- [DataObject](api:SilverStripe\ORM\DataObject)
- [DataList](api:SilverStripe\ORM\DataList)
- [DataQuery](api:SilverStripe\ORM\DataQuery)
- [DataObjectSchema](api:SilverStripe\ORM\DataObjectSchema)
