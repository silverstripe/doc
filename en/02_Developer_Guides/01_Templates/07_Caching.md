---
title: Caching
summary: How template variables are cached.
icon: rocket
---

# Caching

## Object caching

Any values accessed from a template on a `ModelData` object are cached after first access. Because of this, methods that provide data to templates should ideally have no side effects. For
example, this `getCounter()` method will not behave as you might imagine when invoked from a template.

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyObject extends DataObject
{
    // ...

    private $counter = 0;

    public function getCounter()
    {
        $this->counter += 1;

        return $this->counter;
    }
}
```

```ss
$Counter, $Counter, $Counter
// renders as 1, 1, 1
```

When we render `$Counter` to the template we would expect the value to increase and output `1, 2, 3`. However, as
`$Counter` is cached at the first access, the value of `1` is used each time it is invoked in this template.

## Partial caching

Partial caching is a feature that allows caching of a portion of a page as a single string value. For more details read [its own documentation](partial_template_caching).

Example:

```ss
<% cached $CacheKey if $CacheCondition %>
    $CacheableContent
<% end_cached %>
```

## Template caching

Every time a raw template file is processed, some PHP code is generated from it which is then executed to produce the final rendered result.

That PHP code is cached in the filesystem so that the raw template file doesn't need to be processed again until either the cache is flushed or the raw template file is updated.
