---
title: Modules
summary: Extend core functionality with modules.
icon: code
---

# Modules

Silverstripe CMS is designed to be a modular application system - even the CMS is simply a module that plugs into the core
framework.

A module is a collection of classes, templates, and other resources that is loaded into a directory.
Modules are [Composer packages](https://getcomposer.org/), and are placed in the `vendor/` folder.
These packages need to contain one of the following in its root directory to be recognised as a module by Silverstripe CMS:

- a `_config` directory
- a `_config.php` file
- a `composer.json` file with a `type` of either `silverstripe-vendormodule` or `silverstripe-theme`([example](https://github.com/silverstripe/silverstripe-module/blob/5/composer.json)).

Like with any Composer package, we recommend declaring your PHP classes through
[PSR-4 autoloading](https://getcomposer.org/doc/01-basic-usage.md#autoloading).
Silverstripe CMS will automatically discover templates and configuration settings
within your module when you next flush your cache.

## Finding modules

- [Packagist.org "Silverstripe" tag](https://packagist.org/search/?tags=silverstripe)
- [GitHub.com "Silverstripe" search](https://github.com/search?q=silverstripe)

## Installation

Modules are installed through the [Composer](https://getcomposer.org) package manager. It
enables you to install modules from specific versions, checking for compatibilities between modules and even allowing
to track development branches of them. To install modules using this method, you will first need to setup Silverstripe CMS
with [Composer](../../getting_started/composer).

Each module has a unique identifier, consisting of a vendor prefix and name. For example, the "linkfield" module has the
identifier `silverstripe/linkfield` as it is published by *Silverstripe*. To install, use the following command executed in
the project root folder:

```bash
composer require silverstripe/linkfield
```

This will fetch the latest compatible stable version of the module.

Composer uses [version constraints](https://getcomposer.org/doc/articles/versions.md). You can declare a specific constraint to install
if you want to, but typically if you leave the constraint blank Composer will correctly detect an appropriate constraint for you based
on your project's stability configuration.

To lock down to a specific version, branch or commit, read up on
["lock" files](https://getcomposer.org/doc/01-basic-usage.md#commit-your-composer-lock-file-to-version-control).

> [!WARNING]
> After you add or remove modules, make sure you rebuild the database and flush the cache by running `sake db:build --flush`

## Creating a module {#create}

Creating a module is a good way to re-use code and templates across multiple projects,
or share your code with the community. Silverstripe CMS already
has certain modules included, for example the `cms` module and various functionality such as commenting and spam protection
are also abstracted into modules allowing developers the freedom to choose what they want.

### Create a new repository

The easiest way to get started is our [Module Skeleton](https://github.com/silverstripe/silverstripe-module).

You can create a new repository based on the skeleton using the ["Use this template"](https://github.com/silverstripe/silverstripe-module/generate) function on GitHub.

### Allow your module to be importable by Composer

You could import your project using Composer right away - but the name in your `composer.json` file is still "silverstripe-module/skeleton"
instead of being the appropriate name for your module.

You can either edit the file directly in GitHub, or you can clone the repository to a temporary directory - but bare in mind you'll only be
using this directory to update the `composer.json` file. Once that's updated in the repository, you'll use Composer to include your module in
a Silverstripe CMS project and do your development from there.

Here is an example for a module that builds on the functionality provided by the `silverstripe/linkfield` module, so it has that module as a dependency:

```json
{
    "name": "my_vendor/module_name",
    "description": "Short module description",
    "type": "silverstripe-vendormodule",
    "require": {
        "silverstripe/framework": "^6.0",
        "silverstripe/cms": "^6.0",
        "silverstripe/linkfield": "^5.0"
    }
}
```

Commit your change, and if you chose to make this change locally on your computer, push the changes back up to GitHub. You can delete the temporary
directory now, if you created one.

### Add the module to a project for development

To develop the module, you'll want to include it in a Silverstripe CMS project. It's up to you whether you
do this in a project you already have set up, or start a new one from scratch specifically for this purpose.
In most cases we recommended using a new project, at least at first.

If this is a module you intend to be available publicly, it might make sense to submit the repository to
[Packagist](https://packagist.org/) at this stage.

> [!NOTE]
> If you want your module to be private or for some reason don't want to publish it in packagist just yet,
> see [Including a private module in your project](#including-a-private-module-in-your-project) below.

Once you've done that, you can simply install it like
you would any other dependency - just make sure you use the `--prefer-source` option, which will ensure
Composer installs the module directly from GitHub and keeps the initialised local git repository.

```bash
composer require my_vendor/module_name:dev-main --prefer-source
```

> [!TIP]
> The `dev-main` portion of the above command above is a version constraint which tells Composer to install your module from the `main` branch.
> If you are using a different branch name, you should use the correct branch here instead (e.g. if the branch name is `development`, the constraint
> will be `dev-development`).

Once Composer has installed the module, you can develop your module in the `vendor/my_vendor/module_name` directory,
and commit/push changes from there to the remote repository in GitHub.

#### Including a private module in your project

Including public or private repositories that are not indexed on **Packagist** is different from simply using the `composer require` command. We will need to point Composer to specific URLs. Background information can be found at
[Working with project forks and unreleased modules](../../getting_started/composer/#working-with-project-forks-and-unreleased-modules).

For our example module you can add the following lines to your `composer.json` file in the root directory of your main project.

> [!WARNING]
> This goes into the `composer.json` for the Silverstripe CMS project where you're installing your module, *not* into the `composer.json` of your module itself.

```json
{
    "repositories": [
        {
            "type": "vcs",
            "url": "git@github.com:my_vendor/module_name.git",
        }
    ]
}
```

This will add the repository to the list of URLs Composer checks when updating the project dependencies. You can
now include the dependency as normal, e.g:

```bash
composer require my_vendor/module_name:dev-main --prefer-source
```

### Open-sourcing your creation for the community to use

In case you want to share your creation with the community, read more about [publishing a module](how_tos/publish_a_module).

## Module standard

The Silverstripe CMS module standard defines a set of conventions that high-quality Silverstripe CMS modules should follow. It’s a bit like PSR for Silverstripe CMS. Suggested improvements can be raised as pull requests.
This standard is also part of the more highlevel
[Supported Modules Definition](https://www.silverstripe.org/software/addons/supported-modules-definition/)
which the Silverstripe CMS project applies to the modules it creates and maintains directly.

### Coding guidelines

- Complies to a well defined module directory structure and coding standards:
  - `templates/` (for templates e.g. `*.ss` files)
  - `src/` (for `.php` files)
  - `tests/` (for `*Test.php` test files), and;
  - `_config/` (for `.yml` config files)
- The module is a Composer package.
- All Composer dependencies are bound to a single major release (e.g. `^6.0` not `>=6`) unless there are obvious reasons not to for some specific dependency.
- There is a level of test coverage.
- Uses strong typing where appropriate.
- A clear [public API](/project_governance/public_api/) documented in the docblock tags.
  - If parameters and return values don't need additional description and are strongly typed, these should be ommitted from the docblock.
- Code follows [PSR-1](https://www.php-fig.org/psr/psr-1/) and [PSR-2](https://www.php-fig.org/psr/psr-2/) style guidelines.
- `.gitattributes` will be used to exclude non-essential files from the distribution. At a minimum tests, docs, and IDE/dev-tool config should be excluded.
- Add a [PSR-4 compatible autoload reference](https://getcomposer.org/doc/04-schema.md#psr-4) for your module.

### Documentation guidelines

Documentation will use the following format:

- README.md provides:
  - Links or badges to CI and code quality tools.
  - A short summary of the module, end-user.
  - Installation instructions.
- CONTRIBUTING.md explaining terms of contribution.
- Has a licence (`LICENSE` file) - for Silverstripe CMS supported this needs to be BSD.
- Detailed documentation in `/docs/en` as a nested set of GitHub-compatible Markdown files.
  - It is suggested to use a documentation page named `userguide.md` in `docs/en/` that includes documentation of module features that have CMS user functionality (if applicable). For modules with large userguides, this should be in a directory named `userguide` with an `index.md` linking to any other userguide pages.
- Links and image references are relative, and are able to be followed in viewers such as GitHub.
- Markdown may include non-visible comments or meta-data.

Documentation will cover:

- Installation
- Configuration
- Usage guides for key features; screenshots are recommended.

## Related

- [Module Skeleton](https://github.com/silverstripe/silverstripe-module)
- [Publishing a module](how_tos/publish_a_module)
