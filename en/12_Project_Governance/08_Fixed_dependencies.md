---
title: Fixed dependencies
description: Dependencies which will not be changed for the lifetime of a given major release
---

# Silverstripe CMS 6 fixed dependencies

Silverstripe CMS relies on third party packages. Silverstripe CMS projects may interact directly or indirectly with those dependencies.

We've defined a list of *fixed dependencies* for the lifetime of Silverstripe CMS 6. These dependencies will not be upgraded to new major version within the Silverstripe CMS 6 release line.

Dependencies not in this list may be changed or removed altogether within the lifetime of Silverstripe CMS 6.

Dependency | Fixed major release line
-- | --
[`composer-plugin-api`](https://getcomposer.org/doc/articles/composer-platform-dependencies.md#plugin-package-composer-plugin-api) | 2
[`composer/installers`](https://packagist.org/packages/composer/installers) | 2
[`guzzlehttp/guzzle`](https://packagist.org/packages/guzzlehttp/guzzle) | 7
[`guzzlehttp/psr7`](https://packagist.org/packages/guzzlehttp/psr7) | 2
[`intervention/image`](https://packagist.org/packages/intervention/image) | 3
[`league/flysystem`](https://packagist.org/packages/league/flysystem) | 3
[`monolog/monolog`](https://packagist.org/packages/monolog/monolog) | 3
[`phpunit/phpunit`](https://packagist.org/packages/phpunit/phpunit) | 11
[`symfony/*`](https://packagist.org/packages/symfony/) | 7

## Symfony dependencies

Silverstripe CMS relies on many Symfony packages. Silverstripe CMS 6 tracks the Symfony 7 release. If your Silverstripe CMS project uses Symfony packages, install packages version compatible with Symfony 6 or you may encounter dependency conflicts.

## Related information

- [Dependency management](/project_governance/major_release_policy/#dependency-management) as defined by our Major release policy
- [Symfony support commitments](/project_governance/major_release_policy/#symfony-support-commitments) as defined by our Major release policy
