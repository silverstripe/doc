---
title: Composer
summary: What is Composer and how to use it with Silverstripe CMS
---

# Using Silverstripe CMS with Composer

## Requirements

[Composer](https://getcomposer.org/) is a package management tool for PHP that lets you install and upgrade Silverstripe CMS
and its modules. We also have separate instructions
for [installing modules with Composer](/developer_guides/extending/modules).

Before installing Composer you should ensure your system has the version control
system [Git installed](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git). Composer uses Git to check out
the code dependencies you need to run your Silverstripe CMS website from the code repositories maintained on GitHub.

Next, [install Composer](https://getcomposer.org/download/). For our documentation we assume the `composer` command is
installed globally. You should now be able to run the command:

```bash
composer help
```

> [!NOTE]
> If you already have Composer installed, make sure it is composer 2 by running `composer --version`. If you're running Composer 1, run [`composer self-update`](https://getcomposer.org/doc/03-cli.md#self-update-selfupdate). You may also want to check out the [upgrade guide for Composer 1.x to 2.0](https://getcomposer.org/upgrade/UPGRADE-2.0.md).

## Create a new site

Composer can create a new site for you, using the installer as a template. By default it will download the latest stable
version:

```bash
composer create-project silverstripe/installer my-project
```

> [!TIP]
> With the above command, `my-project` is the path (relative to your current working directory) where Composer will create the project.
>
> For example, on OSX, you might want to create a project as a sub-directory of `~/Sites`. You could do that by running `cd ~/Sites` before
> the above command.
>
> If that directory doesn't exist, Composer will create it for you.

If you want to get additional fixtures for testing, such as behat and phpunit configuration, an
example `.env.example` file, and all documentation, then it's recommended to use the `--prefer-source` option
to include these files.

If you want a minimal installation with the bare essentials to get working without any additional overhead, and don't
plan on contributing back changes to framework, use the `--prefer-dist` option (which is implied by default) for a more
lightweight install.

This will get all the code that you need. As long as your web server and database are up and running, you can now
visit the site in your web browser and the installation process will be completed.

You can also specify a constraint to download explicit versions or set boundary conditions for what versions Composer is allowed to install.
For example, this will download the latest patch of the `6.0` release:

```bash
composer create-project silverstripe/installer ./my-project ^6.0
```

Learn more about Composer constraints in [the official composer documentation](https://getcomposer.org/doc/articles/versions.md#writing-version-constraints)

When `create-project` is used with a release version like above, it will try to get the code from archives instead of
creating git repositories. If you're planning to contribute to Silverstripe CMS,
see [Using development versions](#using-development-versions).

## Adding modules to your project

Composer isn't only used to download Silverstripe CMS, it is also used to manage all Silverstripe CMS modules
and any other PHP dependencies you may have.
You can find thousands of Silverstripe CMS modules on [Packagist](https://packagist.org/search/?type=silverstripe-vendormodule).
Installing a module can be done with [the require command](https://getcomposer.org/doc/03-cli.md#require-r):

```bash
composer require dnadesign/silverstripe-elemental
```

This will install the `dnadesign/silverstripe-elemental` module in the latest compatible version. If you know the specific version you
want to install already (such as `^6`), you can add it after the package name as
a [version constraint](https://getcomposer.org/doc/articles/versions.md#writing-version-constraints):

```bash
composer require dnadesign/silverstripe-elemental ^6
```

> [!WARNING]
> **Version constraints:** `master` or `main` is not a legal version string - it's a branch name. These are different things. The
> version string that would get you the branch is `dev-main`. The version string that would get you a numeric branch is
> a little different. The version string for the `6` branch is `6.x-dev`.

## Updating dependencies

Except for the control code of the Voyager space probe, every piece of code in the universe gets updated from time to
time. Silverstripe CMS modules are no exception.

To get the latest updates of the modules in your project, run [the update command](https://getcomposer.org/doc/03-cli.md#update-u-upgrade):

```bash
composer update
```

Updates to the required modules will be installed, and the `composer.lock` file will get updated with the specific
commits and version constraints for each of them.

> [!TIP]
> The update command can also be used to *downgrade* dependencies - if you edit your `composer.json` file and set a version
> constraint that will require a lower version to be installed, running `composer update` will "update" your installed
> dependencies to match your constraints, which in this case would install lower versions than what you had previously.
>
> You may occasionally need to use the `--with-all-dependencies` option when downgrading dependencies to avoid conflicting
> version constraints.

## Deploying projects with Composer

When deploying projects with Composer, you could just push the code and run `composer update`. This, however, is risky.
In particular, if you were referencing development dependencies and a change was made between your testing and your
deployment to production, you would end up deploying untested code. Not cool!

The `composer.lock` file helps with this. It references the specific commits that have been checked out, rather than the
version string. You can run `composer install` to install dependencies from this rather than `composer.json`.

So your deployment process, as it relates to Composer, should be as follows:

- Run `composer update` on your development version before you start whatever testing you have planned. Perform all the
  necessary testing.
- Check `composer.lock` into your repository.
- Deploy your project code base, using the deployment tool of your choice.
- Run `composer install --no-dev -o` on your production version. In this command, the `--no-dev` command tells Composer
  not to install your development-only dependencies, and `-o` is an alias for `--optimise-autoloader`, which will
  convert your PSR-0 and PSR-4 autoloader definitions into a classmap to improve the speed of the autoloader.

## Composer managed modules, Git and `.gitignore`

Modules and themes managed by Composer should not be committed with your project's source code. Silverstripe CMS recipes ship with
a [.gitignore](https://git-scm.com/docs/gitignore) file by default which prevents this. For more details
read [Should I commit the dependencies in my vendor directory?](https://getcomposer.org/doc/faqs/should-i-commit-the-dependencies-in-my-vendor-directory.md)
.

## Dev environments for contributing code {#contributing}

So you want to contribute to Silverstripe CMS? Fantastic! You can do this with Composer too. You have to tell Composer three
things in order to be able to do this:

- Keep the full git repository information
- Include dependencies marked as "developer" requirements
- Use the development version, not the latest stable version

The first two steps are done as part of the initial create project using additional arguments.

```bash
composer create-project --keep-vcs silverstripe/installer ./my-project 6.x-dev --prefer-source
```

The process will take a bit longer, since all modules are checked out as full git repositories which you can work on.
The command checks out from the 6.x release line. If you are providing a patch to fix a bug, replace `6.x-dev` with the
latest minor branch instead, e.g. `6.1.x-dev` if the latest minor release was for CMS 6.1.

The `--keep-vcs` flag will make sure you have access to the git history of the installer and the requirements.

It's also a good idea to require [`silverstripe/recipe-testing`](https://github.com/silverstripe/recipe-testing) as a
dev dependency - it adds a few modules which are useful for Silverstripe CMS development:

- The `behat-extension` module allows running [Behat](https://behat.org) integration tests
- The `phpunit` library is used to run unit and functional tests
- The `php_codesniffer` library is used to lint PHP to ensure it adheres to our
[coding conventions](/contributing/php_coding_conventions/#php-coding-conventions).

Please read the [Contributing Code](/contributing/code) documentation to find out how to create forks and send pull
requests.

## Advanced usage

### Manually editing `composer.json`

To remove dependencies, or if you prefer seeing your dependencies in a text file, you can edit the `composer.json`
file. It will appear in your project root, and by default, it will look something like this:

```json
{
    "name": "silverstripe/installer",
    "type": "silverstripe-recipe",
    "description": "The SilverStripe Framework Installer",
    "require": {
        "php": "^8.3",
        "silverstripe/recipe-plugin": "~2.0.1@stable",
        "silverstripe/vendor-plugin": "~3.0.0@stable",
        "silverstripe/recipe-cms": "~6.0.0@stable",
        "silverstripe/startup-theme": "~1.0.0@stable",
        "silverstripe/login-forms": "~6.0.0@stable"
    },
    "require-dev": {
        "phpunit/phpunit": "^11.3"
    },
    "extra": {
        "project-files-installed": [
            ".htaccess",
            "app/.htaccess",
            "app/_config.php",
            "app/_config/mimevalidator.yml",
            "app/_config/mysite.yml",
            "app/src/Page.php",
            "app/src/PageController.php"
        ],
        "public-files-installed": [
            ".htaccess",
            "index.php",
            "web.config"
        ]
    },
    "config": {
        "process-timeout": 600,
        "allow-plugins": {
            "composer/installers": true,
            "silverstripe/recipe-plugin": true,
            "silverstripe/vendor-plugin": true
        }
    },
    "prefer-stable": true,
    "minimum-stability": "dev"
}
```

To add modules, you should add more entries into the `"require"` section. For example, we might add the blog and forum
modules.

> [!WARNING]
> Be careful with the commas at the end of the lines! You can run `composer validate` to be sure your `composer.json`
> file is formatted correctly.

Save your file, and then run the following command to update the installed packages:

```bash
composer update
```

### Using development versions

Composer will by default download the latest stable version of silverstripe/installer. The `composer.json` file that
comes with silverstripe/installer may also explicitly state it requires the stable version of CMS and framework - this
is to ensure that when developers are getting started, running `composer update` won't upgrade their project to an
unstable version

However it is relatively easy to tell Composer to use development versions. Not only is this required if you want to
contribute back to the Silverstripe CMS project, it also allows you to get fixes and API changes early.

This is a two step process. First you get Composer to start a project based on the latest unstable
silverstripe/installer

```bash
composer create-project silverstripe/installer ./my-project 6.x-dev
```

Or for the latest development version in the 6.0.x minor release (i.e. if you're developing a bug fix)

```bash
composer create-project silverstripe/installer ./my-project 6.0.x-dev
```

### Working with project forks and unreleased modules

By default, Composer will install modules listed on the Packagist site. There are a few reasons that you might not want
to do this. For example:

- You may have your own fork of a module, either specific to a project, or because you are working on a pull request
- You may have a module that hasn't been released to the public.

There are many ways that you can address this, but this is one that we recommend, because it minimises the changes you
would need to make to switch to an official version in the future.

This is how you do it:

- **Ensure that all of your fork repositories have correct `composer.json` files.** Set up the project forks as you would
  a distributed package. If you have cloned a repository that already has a `composer.json` file, then there's nothing you
  need to do, but if not, you will need to create one yourself.

- **List all your fork repositories in your project's `composer.json` files.** You do this in a `repositories` section.
  Set the `type` to `vcs`, and `url` to the URL of the repository. The result will look something like this:

```json
{
    "name": "silverstripe/installer",
    "description": "The Silverstripe Framework Installer",
    "repositories": [
        {
            "type": "vcs",
            "url": "git@github.com:sminnee/silverstripe-cms.git"
        }
    ]
}
```

- **Install the module as you would normally.** Use the regular Composer commands - there are no special flags to use a
  fork. Your fork will be used in place of the package version, so long as it meets the dependency version constraint.

```bash
composer require silverstripe/cms
```

> [!WARNING]
> In most cases, you will probably have a specific branch of the fork you want to install. You should use the appropriate
> version constraint to install that branch. For example, to install a branch called `fix/issue-1990` your version constraint
> should be `dev-fix/issue-1990`.
>
> Depending on what other installed modules have that package as a dependency, you may also need to declare an
> [inline alias](https://getcomposer.org/doc/articles/aliases.md#require-inline-alias).
>
> See more about this in [Forks and branch names](#forks-and-branch-names) below.

Composer will scan all of the repositories you list, collect meta-data about the packages within them, and use them in
favour of the packages listed on packagist. To switch back to using the mainline version of the package, just remove
the `repositories` section from `composer.json` and run `composer update`.

Now add an "upstream" remote to the original repository location so you can rebase or merge your fork as required.

```bash
cd cms
git remote add -f upstream git://github.com/silverstripe/silverstripe-cms.git
```

For more information, read
the ["Repositories" chapter of the Composer documentation](https://getcomposer.org/doc/05-repositories.md).

#### Forks and branch names

For simplicity, you should keep using the same pattern of branch names as the main repositories does. If your version is a
fork of 6.0, then call the branch `6.0`, not `6.0-myproj` or `myproj`. Otherwise, the dependency resolution gets
confused.

Sometimes, however, this isn't feasible. For example, you might have a number of project forks stored in a single
repository, such as your personal GitHub fork of a project. Or you might be testing/developing a feature branch. Or it
might just be confusing to other team members to call the branch of your modified version `6.0`.

In this case, you need to use Composer's aliasing feature to specify how you want the project branch to be treated, when
it comes to dependency resolution.

Open `composer.json`, and find the module's `require`. Then put `as (core version name)` on the end.

```json
{
    "require": {
        "php": "^8.3",
        "silverstripe/recipe-cms": "~6.0.0@stable",
        "silverstripe/framework": "dev-myproj as 6.0.0",
        "silverstripe/startup-theme": "~1.0.0"
    }
}
```

What this means is that when the `myproj` branch is checked out into a project, this will satisfy any dependencies
that `6.0.0` would meet. So, if another module has `"silverstripe/framework": "^6.0.0"` in its dependency list, it
won't get a conflict.

Both the version and the alias are specified as Composer versions, not branch names. For the relationship between
branch/tag names and Composer versions,
read [the relevant Composer documentation](https://getcomposer.org/doc/02-libraries.md).

This is not the only way to set things up in Composer. For more information on this topic, read
the ["Aliases" chapter of the Composer documentation](https://getcomposer.org/doc/articles/aliases.md).

## FAQ

### How should I name my module?

Follow the packagist.org advice on choosing a [unique name and vendor prefix](https://packagist.org/about#naming-your-package). Please don't
use the `silverstripe/<modulename>` vendor prefix, since that's reserved for modules produced by Silverstripe Ltd. In
order to declare that your module is in fact a Silverstripe CMS module, use the `silverstripe` tag in the `composer.json`
file, and set the "type" to `silverstripe-vendormodule`.

### What about themes?

Themes are technically just "modules" which are placed in the `themes/` subdirectory. We denote a special type for them
in the `composer.json` (`"type": "silverstripe-theme"`), which triggers their installation into the correct path.

Themes should not have any PHP code in them - if your theme needs some PHP code to function correctly, add that PHP code
to a separate module and include it as a dependency in the theme's `composer.json` file.

### Do I need Composer on my live server?

It depends on your deployment process. If you copy or rsync files to your live server, you won't need Composer on the live server. If
the live server hosts a git repository checkout, which is updated to push a newer version, you'll need to
run `composer install` checking out the code. We recommend looking
into [Composer "lock" files](https://getcomposer.org/doc/01-basic-usage.md#composer-lock-the-lock-file) for this purpose.

### Can I keep using downloads, subversion externals or Git submodules?

Composer is more than just a file downloader. It comes with additional features such as
[autoloading](https://getcomposer.org/doc/01-basic-usage.md#autoloading)
and [scripts](https://getcomposer.org/doc/articles/scripts.md)
which some modules rely on. You really should be using Composer to manage your PHP dependencies.

### I don't want to get development versions of everything

You don't have to, Composer is designed to work on the constraints you set. You can declare
the ["minimum-stability"](https://getcomposer.org/doc/04-schema.md#minimum-stability)
on your project as suitable, or even whitelist specific modules as tracking a development branch while keeping others to
their stable release. Read up
on [Composer "lock" files](https://getcomposer.org/doc/01-basic-usage.md#commit-your-composer-lock-file-to-version-control) on how this all
fits together.
