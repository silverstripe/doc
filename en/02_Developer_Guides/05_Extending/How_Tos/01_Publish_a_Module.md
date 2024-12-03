---
title: How to Publish a Silverstripe CMS module
summary: Have you created some work you think others can use? Turn it into a module and share it.
icon: rocket
---

# How to publish a Silverstripe CMS module

After you've [created](../modules#create) your own Silverstripe module,
you could decide to make it open source and share it with the world.

If you wish to submit your module to our public directory, you take responsibility for a certain level of code quality,
adherence to conventions, writing documentation, and releasing updates.

Silverstripe CMS uses [Composer](../../../getting_started/composer/) to manage module releases and dependencies between
modules. If you plan on releasing your module to the public, ensure that you provide a `composer.json` file in the root
of your module containing the meta-data about your module.

For more information about what your `composer.json` file should include, consult the
[Composer Documentation](https://getcomposer.org/doc/01-basic-usage.md).

```json
// mycustommodule/composer.json
{
  "name": "my-vendor/my-module",
  "description": "One-liner describing your module",
  "type": "silverstripe-vendormodule",
  "homepage": "https://github.com/my-vendor/my-module",
  "keywords": ["silverstripe", "some-tag", "some-other-tag"],
  "license": "BSD-3-Clause",
  "authors": [
    {
      "name": "Your Name","email": "your@email.com"
    }
  ],
  "support": {
    "issues": "https://github.com/my-vendor/my-module/issues"
  },
  "require": {
    "silverstripe/cms": "^6",
    "silverstripe/framework": "^6"
  },
  "autoload": {
    "psr-4": {
      "MyVendor\\MyModule\\": "src/"
    }
  },
  "extra": {
    "installer-name": "my-module",
    "expose": [
      "client"
    ],
    "screenshots": [
      "relative/path/screenshot1.png",
      "https://www.example.com/screenshot2.png"
    ]
  }
}
```

Once your module is published online with a service like GitHub.com or bitbucket.com, submit the repository to
[Packagist](https://packagist.org/) to have the module accessible to developers.

Note that Silverstripe CMS modules have the following distinct characteristics:

- Silverstripe CMS modules can be differentiated programatically from other packages by declaring `type: silverstripe-vendormodule`.
- Any folder which should be exposed to the public webroot must be declared in the `extra.expose` config.
   These paths will be automatically rewritten to public urls which don't directly serve files from the `vendor`
   folder. For instance, `vendor/my-vendor/my-module/client` will be rewritten to
   `_resources/my-vendor/my-module/client`. See [Exposing static resources](/developer_guides/templates/requirements/#exposing-static-resources)
   for more details about this.

## Releasing versions

Over time you may have to release new versions of your module to continue to work with newer versions of Silverstripe CMS.
By using Composer, this is made easy for developers by allowing them to specify what version they want to use. Each
version of your module should be a separate branch in your version control and each branch should have a `composer.json`
file explicitly defining what versions of Silverstripe CMS you support.

Say you have a module which supports Silverstripe CMS 6.0. A new release of this module takes advantage of new features
in Silverstripe CMS 6.1. In this case, you would create a new branch for the 6.0 compatible code base of your module. This
allows you to continue fixing bugs on this older release branch.

Other branches should be created on your module as needed if they're required to support specific Silverstripe CMS releases.

You can have an overlap in supported versions, e.g two branches in your module both support Silverstripe CMS 6.0. In this
case, you should explain the differences in your `README.md` file.

Here's some common values for your `require` section
(see [getcomposer.org](https://getcomposer.org/doc/01-basic-usage.md#package-versions) for details):

- `6.0.*`: Version `6.0`, including `6.0.1`, `6.0.2` etc, excluding `6.1`
- `~6.0`: Version `6.0` or higher, including `6.0.1` and `6.1` etc, excluding `7.0`
- `~6.0,<6.2`: Version `6.0` or higher, up until `6.2`, which is excluded
- `~6.0,>6.0.4`: Version `6.0` or higher, starting with `6.0.4`
