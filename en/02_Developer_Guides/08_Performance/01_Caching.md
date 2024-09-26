---
title: Caching
summary: Optimise performance by caching expensive processes
icon: tachometer-alt
---
# Caching

## Overview

The framework uses caches to store infrequently changing values.
By default, the storage mechanism chooses the most performant adapter available
(PHP opcache, APC, or filesystem). Other cache backends can be configured.

The most common caches are manifests of various resources:

- PHP class locations ([ClassManifest](api:SilverStripe\Core\Manifest\ClassManifest))
- Configuration settings from YAML files ([CachedConfigCollection](api:SilverStripe\Config\Collections\CachedConfigCollection))
- Language files ([i18n](api:SilverStripe\i18n\i18n))

Flushing the various manifests is performed through a GET
parameter (`flush=1`). Since this action requires more server resources than normal requests,
executing the action is limited to the following cases when performed via a web request:

- The [environment](/getting_started/environment_management) is in "dev mode"
- A user is logged in with ADMIN permissions
- An error occurs during startup

Caution: Not all caches are cleared through `flush=1`.
While cache objects can expire, when using filesystem caching the files are not actively pruned.
For long-lived server instances, this can become a capacity issue over time - see
[workaround](https://github.com/silverstripe/silverstripe-framework/issues/6678).

## Configuration

We are using the [PSR-16](https://www.php-fig.org/psr/psr-16/) standard ("SimpleCache")
for caching, through the [symfony/cache](https://symfony.com/doc/current/components/cache.html) library.

Note that this library describes usage of [PSR-6](https://www.php-fig.org/psr/psr-6/) by default,
though Silverstripe wraps these in a PSR-16 interface using the [Psr16Cache](https://github.com/symfony/cache/blob/6.1/Psr16Cache.php) class.

Cache objects are configured via YAML
and Silverstripe CMS's [dependency injection](/developer_guides/extending/injector) system.

```yml
SilverStripe\Core\Injector\Injector:
  Psr\SimpleCache\CacheInterface.myCache:
    factory: SilverStripe\Core\Cache\CacheFactory
    constructor:
      namespace: "myCache"
```

> [!CAUTION]
> Please note that if you have the `silverstripe/versioned` module installed (automatically installed by the
> `silverstripe/cms` module), caches will automatically be segmented by current “stage”. This ensures that
> any content written to the cache in the *draft* reading mode isn’t accidentally exposed in the *live* reading mode.
> Please read the [versioned cache segmentation](#versioned-cache-segmentation) section for more information.

Cache objects are instantiated through a [CacheFactory](api:SilverStripe\Core\Cache\CacheFactory),
which determines which cache adapter is used (see [adapters](#adapters) below for details).
This factory allows us you to globally define an adapter for all cache instances.

```php
use Psr\SimpleCache\CacheInterface;
use SilverStripe\Core\Injector\Injector;

$cache = Injector::inst()->get(CacheInterface::class . '.myCache');
```

Caches are namespaced, which might allow granular clearing of a particular cache without affecting others.
In our example, the namespace is "myCache", expressed in the service name as
`Psr\SimpleCache\CacheInterface.myCache`. We recommend the `::class` short-hand to compose the full service name.

Clearing caches by namespace is dependent on the used adapter: While the `FilesystemAdapter` clears only the namespaced cache,
a `MemcachedAdapter` adapter will clear all caches regardless of namespace, since the underlying memcached
service doesn't support this. See "Invalidation" for alternative strategies.

## Usage

Cache objects follow the [PSR-16](https://www.php-fig.org/psr/psr-16/) class interface.

```php
use Psr\SimpleCache\CacheInterface;
use SilverStripe\Core\Injector\Injector;

$cache = Injector::inst()->get(CacheInterface::class . '.myCache');


// create a new item by trying to get it from the cache
$myValue = $cache->get('myCacheKey');

// set a value and save it via the adapter
$cache->set('myCacheKey', 1234);

// retrieve the cache item
if (!$cache->has('myCacheKey')) {
    // ... item does not exists in the cache
}
```

## Invalidation

Caches can be invalidated in different ways. The easiest is to actively clear the
entire cache. If the adapter supports namespaced cache clearing,
this will only affect a subset of cache keys ("myCache" in this example):

```php
use Psr\SimpleCache\CacheInterface;
use SilverStripe\Core\Injector\Injector;

$cache = Injector::inst()->get(CacheInterface::class . '.myCache');

// remove all items in this (namespaced) cache
$cache->clear();
```

You can also delete a single item based on it's cache key:

```php
use Psr\SimpleCache\CacheInterface;
use SilverStripe\Core\Injector\Injector;

$cache = Injector::inst()->get(CacheInterface::class . '.myCache');

// remove the cache item
$cache->delete('myCacheKey');
```

Individual cache items can define a lifetime, after which the cached value is marked as expired:

```php
use Psr\SimpleCache\CacheInterface;
use SilverStripe\Core\Injector\Injector;

$cache = Injector::inst()->get(CacheInterface::class . '.myCache');

// set a cache item with an expiry
// cache for 300 seconds
$cache->set('myCacheKey', 'myValue', 300);
```

If a lifetime isn't defined on the `set()` call, it'll use the adapter default.
In order to increase the chance of your cache actually being hit,
it often pays to increase the lifetime of caches.
You can also set your lifetime to `0`, which means they won't expire.
Since many adapters don't have a way to actively remove expired caches,
you need to be careful with resources here (e.g. filesystem space).

```yml
---
Name: my-project-cache
After: '#corecache'
---
SilverStripe\Core\Injector\Injector:
  Psr\SimpleCache\CacheInterface.cacheblock:
    constructor:
      defaultLifetime: 3600
```

In most cases, invalidation and expiry should be handled by your cache key.
For example, including the `LastEdited` value when caching `DataObject` results
will automatically create a new cache key when the object has been changed.
The following example caches a member's group names, and automatically
creates a new cache key when any group is edited. Depending on the used adapter,
old cache keys will be garbage collected as the cache fills up.

```php
use Psr\SimpleCache\CacheInterface;
use SilverStripe\Core\Injector\Injector;

$cache = Injector::inst()->get(CacheInterface::class . '.myCache');

// Automatically changes when any group is edited
$cacheKey = implode(['groupNames', $member->ID, Group::get()->max('LastEdited')]);
$cache->set($cacheKey, $member->Groups()->column('Title'));
```

If `?flush=1` is requested in the URL, this will trigger a call to `flush()` on
any classes that implement the [Flushable](/developer_guides/execution_pipeline/flushable/)
interface. Use this interface to trigger `clear()` on your caches.

## Adapters

We use the `smyfony/cache` library which supports various [cache adapters](https://github.com/symfony/cache/tree/6.1/Adapter).

Silverstripe CMS tries to provide a sensible default cache implementation for your system
through the [`DefaultCacheFactory`](api:SilverStripe\Core\Cache\DefaultCacheFactory) implementation.

- `PhpFilesAdapter` (PHP with [opcache](https://php.net/manual/en/book.opcache.php) enabled).
    This cache has relatively low [memory defaults](https://php.net/manual/en/opcache.configuration.php#ini.opcache.memory-consumption).
    We recommend increasing it for large applications, or enabling the
    [`file_cache` fallback](https://php.net/manual/en/opcache.configuration.php#ini.opcache.file-cache).
    You must have [`opcache.enable_cli`](https://www.php.net/manual/en/opcache.configuration.php#ini.opcache.enable-cli) set to `true`
    to use this cache adapter. This is so that your cache is shared between CLI and the webserver.
- `FilesystemAdapter` if the above isn't available

### Adding an in-memory cache adapter

The cache adapter needs to be available before configuration has been loaded, so we use an environment variable to determine which class will be used to instantiate the in-memory cache adapter. The class must be referenced using its Fully Qualified Class Name (including namespace), and must be an instance of [`InMemoryCacheFactory`](api:SilverStripe\Core\Cache\InMemoryCacheFactory).

```bash
SS_IN_MEMORY_CACHE_FACTORY="SilverStripe\Core\Cache\MemcachedCacheFactory"
```

Silverstripe CMS comes with three in-memory cache factories you can choose from, each with their own requirements. You can of course add your own, as well.

#### Memcached cache

[Memcached](https://www.danga.com/memcached/) is a "high-performance, distributed memory object caching system". To use this cache, you must install the [memcached PHP extension](https://php.net/memcached).

You can tell the cache adapter which Memcached server(s) to connect to by defining a DSN in the `SS_MEMCACHED_DSN` environment variable.

> [!TIP]
> The format for the DSN is exactly as defined in [the Symfony documentation](https://symfony.com/doc/current/components/cache/adapters/memcached_adapter.html#configure-the-connection).

```bash
SS_IN_MEMORY_CACHE_FACTORY="SilverStripe\Core\Cache\MemcachedCacheFactory"
SS_MEMCACHED_DSN="memcached://localhost:11211"
```

#### Redis cache

[Redis](https://redis.io/) is an "in-memory database for caching and streaming" with built-in replication and a lot of deployment options. To use this cache, you must install one of:

- [predis/predis](https://github.com/predis/predis/)
- [cachewerk/relay](https://github.com/cachewerk/relay)
- [the Redis PHP extension](https://github.com/phpredis/phpredis)

You can tell the cache adapter which Redis server(s) to connect to by defining a DSN in the `SS_REDIS_DSN` environment variable.

> [!TIP]
> The format for the DSN is exactly as defined in [the Symfony documentation](https://symfony.com/doc/current/components/cache/adapters/redis_adapter.html#configure-the-connection).

```bash
SS_IN_MEMORY_CACHE_FACTORY="SilverStripe\Core\Cache\RedisCacheFactory"
SS_REDIS_DSN="redis://verysecurepassword@localhost:6379"
```

#### APCu cache

APCu is "an in-memory key-value store for PHP". This runs locally on the same computer as your webserver, and is only available for the webserver itself. To use this cache, you must install the [APCu PHP extension](https://www.php.net/apcu).

> [!WARNING]
> APCu cache cannot be shared with your CLI, which means flushing cache from the terminal will not flush the cache your webserver uses.
>
> The filesystem cache will still be used as a backup, which *is* shared with CLI, so flushing the cache with a web request will always flush the cache CLI uses.

We include this option because it requires very little effort to set up, so it may be appropriate for smaller projects. Just remember to always flush the cache with an HTTP request using `?flush=1`.

```bash
SS_IN_MEMORY_CACHE_FACTORY="SilverStripe\Core\Cache\ApcuCacheFactory"
```

## Versioned cache segmentation

`SilverStripe\Core\Cache\CacheFactory` now maintains separate cache pools for each versioned stage (if you have the
`silverstripe/versioned` module installed). This prevents developers from caching draft data and then
accidentally exposing it on the live stage without potentially required authorisation checks. Unless you
rely on caching across stages, you don't need to change your own code for this change to take effect. Note
that cache keys will be internally rewritten, causing any existing cache items to become invalid when this
change is deployed.

```php
// Before:
$cache = Injector::inst()->get(CacheInterface::class . '.myapp');
Versioned::set_stage(Versioned::DRAFT);
$cache->set('my_key', 'Some draft content. Not for public viewing yet.');
Versioned::set_stage(Versioned::LIVE);
// 'Some draft content. Not for public viewing yet'
$cache->get('my_key');

// After:
$cache = Injector::inst()->get(CacheInterface::class . '.myapp');
Versioned::set_stage(Versioned::DRAFT);
$cache->set('my_key', 'Some draft content. Not for public viewing yet.');
Versioned::set_stage(Versioned::LIVE);
// null
$cache->get('my_key');
```

Data that is not content sensitive can be cached across stages by simply opting out of the segmented cache
with the `disable-container` argument.

```yml
SilverStripe\Core\Injector\Injector:
  Psr\SimpleCache\CacheInterface.myapp:
    factory: SilverStripe\Core\Cache\CacheFactory
    constructor:
      namespace: "MyInsensitiveData"
      disable-container: true
```

## Additional caches

Unfortunately not all caches are configurable via cache adapters.

- [`SSTemplateEngine`](api:SilverStripe\View\SSTemplateEngine) writes compiled templates as PHP files to the filesystem
   (in order to achieve opcode caching on `include()` calls)

- [i18n](api:SilverStripe\i18n\i18n) uses `Symfony\Component\Config\ConfigCacheFactoryInterface` (filesystem-based)
