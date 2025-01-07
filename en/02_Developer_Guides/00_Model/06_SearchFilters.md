---
title: SearchFilter Modifiers
summary: Use suffixes on your ORM queries.
icon: search
---

# `SearchFilter` modifiers

The `filter()`, `exclude()`, and other related methods on [`DataList`](api:SilverStripe\ORM\DataList), [`ArrayList`](api:SilverStripe\Model\List\ArrayList), and [`EagerLoadedList`](api:SilverStripe\ORM\EagerLoadedList) specify exact matches by default. However, when calling these methods, there are a number of suffixes that
you can put on field names to change this behavior. These are represented as `SearchFilter` subclasses and include:

- [`ExactMatchFilter`](api:SilverStripe\ORM\Filters\ExactMatchFilter)
- [`StartsWithFilter`](api:SilverStripe\ORM\Filters\StartsWithFilter)
- [`EndsWithFilter`](api:SilverStripe\ORM\Filters\EndsWithFilter)
- [`PartialMatchFilter`](api:SilverStripe\ORM\Filters\PartialMatchFilter)
- [`GreaterThanFilter`](api:SilverStripe\ORM\Filters\GreaterThanFilter)
- [`GreaterThanOrEqualFilter`](api:SilverStripe\ORM\Filters\GreaterThanOrEqualFilter)
- [`LessThanFilter`](api:SilverStripe\ORM\Filters\LessThanFilter)
- [`LessThanOrEqualFilter`](api:SilverStripe\ORM\Filters\LessThanOrEqualFilter)

See [`SilverStripe\ORM\Filters` in the API docs](api:SilverStripe\ORM\Filters) for a full list of `SearchFilter` classes available in silverstripe/framework.

An example of a `SearchFilter` in use:

```php
// fetch any player whose first name starts with the letter 'S' and has a PlayerNumber greater than 10
$players = Player::get()->filter([
    'FirstName:StartsWith' => 'S',
    'PlayerNumber:GreaterThan' => '10',
]);

// fetch any player whose name contains the letter 'z'
$players = Player::get()->filterAny([
    'FirstName:PartialMatch' => 'z',
    'LastName:PartialMatch' => 'z',
]);
```

> [!TIP]
> Notice the syntax - to invoke a `SearchFilter` in the `filter()`/`filterAny()`/`find()` or `exclude()`/`excludeAny()` methods, you add a colon after the field name, followed by the name of the filter (excluding the actual word "filter"). e.g. for a `StartsWithFilter`: `'FieldName:StartsWith'`

Developers can define their own [SearchFilter](api:SilverStripe\ORM\Filters\SearchFilter) if needing to extend the ORM filter and exclude behaviors.

## Modifiers and case sensitivity {#modifiers}

`SearchFilter`s can also take modifiers. The modifiers currently supported are `":not"`, `":nocase"`, and
`":case"` (though you can implement custom modifiers on your own `SearchFilter` implementations). These negate the filter, make it case-insensitive and make it case-sensitive, respectively.

> [!NOTE]
> The default comparison uses the database's default case sensitivity (even for `ArrayList`). For MySQL and MSSQL, this is case-insensitive. For PostgreSQL, this is case-sensitive. But you can declare the default
> case sensitivity for your project by setting the `default_case_sensitive` configuration property on `SearchFilter` like so:
>
> ```yml
> SilverStripe\ORM\Filters\SearchFilter:
>   default_case_sensitive: false
> ```

```php
// Fetch players that their FirstName is exactly 'Sam'
// Caution: This might be case in-sensitive if MySQL or MSSQL is used
$players = Player::get()->filter([
    'FirstName:ExactMatch' => 'Sam',
]);

// Fetch players that their FirstName is exactly 'Sam' (force case-sensitive)
$players = Player::get()->filter([
    'FirstName:ExactMatch:case' => 'Sam',
]);

// Fetch players that their FirstName is exactly 'Sam' (force NOT case-sensitive)
$players = Player::get()->filter([
    'FirstName:ExactMatch:nocase' => 'Sam',
]);
```

By default the `:ExactMatch` filter is applied, so we can shorthand the above to:

```php
// Default DB engine behaviour
$players = Player::get()->filter('FirstName', 'Sam');
// case-sensitive
$players = Player::get()->filter('FirstName:case', 'Sam');
// NOT case-sensitive
$players = Player::get()->filter('FirstName:nocase', 'Sam');
```

Note that all search filters (e.g. `:PartialMatch`) refer to services registered with [`Injector`](api:SilverStripe\Core\Injector\Injector)
within the `DataListFilter.` prefixed namespace. New filters can be registered using the below yml
config:

```yml
SilverStripe\Core\Injector\Injector:
  DataListFilter.CustomMatch:
    class: MyVendor\Search\CustomMatchFilter
```

The following is a query which will return everyone whose first name starts with "S", either lowercase or uppercase:

```php
$players = Player::get()->filter([
    'FirstName:StartsWith:nocase' => 'S',
]);

// use :not to get everyone whose first name does NOT start with "S"
$players = Player::get()->filter([
    'FirstName:StartsWith:not' => 'S',
]);
```

> [!TIP]
> You can combine `:not` and either `:nocase` or `:case`. Note that the order doesn't matter - these two calls are equivalent:
>
> ```php
> $players = Player::get()->filter([
>     'FirstName:StartsWith:nocase:not' => 'S',
> ]);
> $players = Player::get()->filter([
>     'FirstName:StartsWith:not:nocase' => 'S',
> ]);
> ```

## API documentation

- [SearchFilter](api:SilverStripe\ORM\Filters\SearchFilter)
