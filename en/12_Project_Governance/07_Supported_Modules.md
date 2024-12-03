---
title: Supported modules
description: Modules which are commercially supported by Silverstripe
---

# Commercially supported modules

Silverstripe CMS ships modules that receive commercial support. Commercially supported modules receive regular updates to work with the latest Silverstripe CMS release. Their [public API](/project_governance/public_api/) conforms to [semantic versioning](https://semver.org/). They are covered by:

- our [security release process](/contributing/managing_security_issues)
- our [minor release policy](minor_release_policy)
- our [major release policy](major_release_policy).

These modules will be supported for the lifetime of Silverstripe CMS 6 in the provided major versions.

## Core Silverstripe CMS modules

The "core" of Silverstripe refers to the module packages owned by the "Silverstripe" Packagist vendor that fall under one of the following recipes:

- [silverstripe/recipe-core](https://packagist.org/packages/silverstripe/recipe-core)
- [silverstripe/recipe-cms](https://packagist.org/packages/silverstripe/recipe-cms)
- [silverstripe/installer](https://packagist.org/packages/silverstripe/installer)

These modules provide the core Silverstripe CMS experience. Most Silverstripe CMS projects should install all of them.

Core Module | Supported major release line
-- | --
[silverstripe/admin](https://packagist.org/packages/silverstripe/admin) | 3
[silverstripe/asset-admin](https://packagist.org/packages/silverstripe/asset-admin) | 3
[silverstripe/assets](https://packagist.org/packages/silverstripe/assets) | 3
[silverstripe/cms](https://packagist.org/packages/silverstripe/cms) | 6
[silverstripe/config](https://packagist.org/packages/silverstripe/config) | 3
[silverstripe/errorpage](https://packagist.org/packages/silverstripe/errorpage) | 3
[silverstripe/framework](https://packagist.org/packages/silverstripe/framework) | 6
[silverstripe/graphql](https://packagist.org/packages/silverstripe/graphql) | 6
[silverstripe/installer](https://packagist.org/packages/silverstripe/installer) | 6
[silverstripe/login-forms](https://packagist.org/packages/silverstripe/login-forms) | 6
[silverstripe/mimevalidator](https://packagist.org/packages/silverstripe/mimevalidator) | 4
[silverstripe/recipe-cms](https://packagist.org/packages/silverstripe/recipe-cms) | 6
[silverstripe/recipe-core](https://packagist.org/packages/silverstripe/recipe-core) | 6
[silverstripe/recipe-plugin](https://packagist.org/packages/silverstripe/recipe-plugin) | 2
[silverstripe/reports](https://packagist.org/packages/silverstripe/reports) | 6
[silverstripe/session-manager](https://packagist.org/packages/silverstripe/session-manager) | 3
[silverstripe/siteconfig](https://packagist.org/packages/silverstripe/siteconfig) | 6
[silverstripe/template-engine](https://packagist.org/packages/silverstripe/template-engine) | 1
[silverstripe/vendor-plugin](https://packagist.org/packages/silverstripe/vendor-plugin) | 2
[silverstripe/versioned](https://packagist.org/packages/silverstripe/versioned) | 3
[silverstripe/versioned-admin](https://packagist.org/packages/silverstripe/versioned-admin) | 3

## Other supported modules

These modules extend the core Silverstripe CMS functionality. Silverstripe CMS projects can pick and choose which of these modules to install based on their needs.

Supported PHP Module | Supported versions
-- | --
[colymba/gridfield-bulk-editing-tools](https://packagist.org/packages/colymba/gridfield-bulk-editing-tools) | 5
[dnadesign/silverstripe-elemental](https://packagist.org/packages/dnadesign/silverstripe-elemental) | 6
[dnadesign/silverstripe-elemental-userforms](https://packagist.org/packages/dnadesign/silverstripe-elemental-userforms) | 5
[silverstripe-themes/simple](https://packagist.org/packages/silverstripe-themes/simple) | 3
[silverstripe/dynamodb](https://packagist.org/packages/silverstripe/dynamodb) | 6
[silverstripe/gridfieldqueuedexport](https://packagist.org/packages/silverstripe/gridfieldqueuedexport) | 4
[silverstripe/hybridsessions](https://packagist.org/packages/silverstripe/hybridsessions) | 4
[silverstripe/linkfield](https://packagist.org/packages/silverstripe/linkfield) | 5
[silverstripe/lumberjack](https://packagist.org/packages/silverstripe/lumberjack) | 4
[silverstripe/mfa](https://packagist.org/packages/silverstripe/mfa) | 6
[silverstripe/realme](https://packagist.org/packages/silverstripe/realme) | 6
[silverstripe/segment-field](https://packagist.org/packages/silverstripe/segment-field) | 4
[silverstripe/sharedraftcontent](https://packagist.org/packages/silverstripe/sharedraftcontent) | 4
[silverstripe/spamprotection](https://packagist.org/packages/silverstripe/spamprotection) | 5
[silverstripe/staticpublishqueue](https://packagist.org/packages/silverstripe/staticpublishqueue) | 7
[silverstripe/tagfield](https://packagist.org/packages/silverstripe/tagfield) | 4
[silverstripe/taxonomy](https://packagist.org/packages/silverstripe/taxonomy) | 4
[silverstripe/textextraction](https://packagist.org/packages/silverstripe/textextraction) | 5
[silverstripe/totp-authenticator](https://packagist.org/packages/silverstripe/totp-authenticator) | 6
[silverstripe/userforms](https://packagist.org/packages/silverstripe/userforms) | 7
[symbiote/silverstripe-advancedworkflow](https://packagist.org/packages/symbiote/silverstripe-advancedworkflow) | 7
[symbiote/silverstripe-gridfieldextensions](https://packagist.org/packages/symbiote/silverstripe-gridfieldextensions) | 5
[symbiote/silverstripe-queuedjobs](https://packagist.org/packages/symbiote/silverstripe-queuedjobs) | 6
[tractorcow/silverstripe-fluent](https://packagist.org/packages/tractorcow/silverstripe-fluent) | 8

## Supported NPM packages

The following two NPM packages are also supported because they are required to build the UI of Silverstripe CMS 6:

- [@silverstripe/webpack-config](https://www.npmjs.com/package/@silverstripe/webpack-config)
- [@silverstripe/eslint-config](https://www.npmjs.com/package/@silverstripe/eslint-config)

## Other modules in the `silverstripe` namespace

There are other modules hosted under the `silverstripe` Packagist namespace. These modules are maintained on a best effort basis. They are not guaranteed to go through regular regression testing. Their public API may be more fluid than supported modules. They maybe more experimental or may not receive the same level of care as supported modules.

These modules can still be used in Silverstripe CMS projects, but should be considered as community modules.
