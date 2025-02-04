---
title: Directory Structure
summary: An overview of what each directory contains in a Silverstripe CMS installation
icon: sitemap
---

# Directory structure

## Introduction

The directory-structure in Silverstripe is built on "convention over configuration", so the placement of some files and
directories is meaningful to its logic.

## Core structure

Directory            | Description
---------            | -----------
`public/`            | Webserver public webroot
`public/assets/`     | Images and other files uploaded via the Silverstripe CMS. You can also place your own content inside it, and link to it from within the content area of the CMS.
`public/assets/.protected/` | Default location for [protected assets](/developer_guides/files/file_security)
`public/_resources/` | Exposed public files added from modules. Folders within this parent will match that of the source root location ([this can be altered by configuration](/developer_guides/templates/requirements/#configuring-your-project-exposed-folders)).
`vendor/`            | Silverstripe modules and other supporting libraries (e.g. the framework is in `vendor/silverstripe/framework`)
`themes/`            | Standard theme installation location

## Custom code structure

We use `app/` as the default folder.

| Directory             | Description                                                         |
| ---------             | -----------                                                         |
| `app/`           | This directory contains all of your code that defines your website. |
| `app/_config`    | YAML configuration specific to your application                    |
| `app/src`        | PHP code specific to your application (subdirectories are optional)     |
| `app/tests`      | PHP unit/functional/end-to-end tests                                                      |
| `app/templates`  | [templates](/developer_guides/templates) for the `$default` theme   |
| `app/client/src` | Conventional directory for source resources (images/CSS/JavaScript) for your CMS customisations |
| `app/client/dist` | Conventional directory for transpiled resources (images/CSS/JavaScript) for your CMS customisations |
| `app/client/lang` | Conventional directory for [JavaScript translation tables](/developer_guides/i18n/#translation-tables-in-javascript) |
| `app/lang` | Contains [YAML translation tables](/developer_guides/i18n/#language-definitions) |
| `app/themes/<yourtheme>` | Custom nested themes (note: theme structure is described below)     |

Arbitrary directory-names are allowed, as long as they don't collide with existing modules or the directories lists in
"Core Structure". Here's how you would reconfigure your default folder to `myspecialapp`.

```yml
# myspecialapp/_config/config.yml
---
Name: myspecialapp
---
SilverStripe\Core\Manifest\ModuleManifest:
    project: 'myspecialapp'
```

Check our [JavaScript Coding Conventions](/contributing/javascript_coding_conventions/) for more details on folder and file naming in
Silverstripe core modules.

## Themes structure

| Directory                       | Description                                                     |
| ------------------              | ---------------------------                                     |
| `themes/startup-theme/`         | Default theme                                                   |
| `themes/<yourtheme>/`           | Custom theme base directory                                     |
| `themes/<yourtheme>/templates`  | Theme templates                                                 |
| `themes/<yourtheme>/css`        | Theme CSS files                                                 |

See [themes](/developer_guides/templates/themes).

## Module structure

Modules are commonly stored as composer packages in the `vendor/` folder. They need to have a `_config.php` file or
a `_config/` directory present, and should follow the same conventions as posed in "Custom Site Structure".

Example for the [silverstripe/blog](https://github.com/silverstripe/silverstripe-blog) module:

| Directory  | Description                                                         |
| ---------  | -----------                                                         |
| `vendor/silverstripe/blog/` | This directory contains all of your code that defines the module. |
| `vendor/silverstripe/_config` | YAML configuration specific to the module                    |
| `vendor/silverstripe/blog/src` | PHP code specific to the module (subdirectories are optional)     |
| ...        | ...                                                                 |

### Module documentation

Module developers can bundle developer documentation with their code by producing plain text files inside a `docs/`
folder located in the module folder. These files can be written with the Markdown syntax
(see [Contributing Documentation](/contributing/documentation))
and include media such as images or videos.

Inside the `docs/` folder, developers should organise the markdown files into each separate language they wish to write
documentation for (e.g. `en` for english documentation). Inside each languages' subfolder, developers then have freedom to create whatever
structure they wish for organising the documentation they wish.

Example Blog Documentation:

| Directory  | Description                                                         |
| ---------  | -----------                                                         |
| `vendor/silverstripe/blog/docs` | |
| `vendor/silverstripe/blog/docs/_manifest_exclude` | Empty file to signify that Silverstripe does not need to load classes from this folder |
| `vendor/silverstripe/blog/docs/en/`       | English documentation  |
| `vendor/silverstripe/blog/docs/en/index.md`    | Documentation homepage. Should provide an introduction and links to remaining docs |
| `vendor/silverstripe/blog/docs/en/Getting_Started.md` | Documentation page. Naming convention is Uppercase and underscores. |
| `vendor/silverstripe/blog/docs/en/_images/` | Folder to store any images or media |
| `vendor/silverstripe/blog/docs/en/Some_Topic/` | You can organise documentation into nested folders. Naming convention is Uppercase and underscores. |
| `vendor/silverstripe/blog/docs/en/04_Some_Topic/00_Getting_Started.md`|Structure is created by use of numbered prefixes. This applies to nested folders and documentations pages, `index.md` should not have a prefix.|

## Autoloading

Silverstripe recursively detects classes in PHP files by building up a manifest used for autoloading, as well as
respecting Composer's built-in autoloading for libraries. This means in most cases, you don't need to worry about
include paths or `require()` calls in your own code - after adding a new class, simply regenerate the manifest by using
a `flush=1` query parameter. See the ["Manifests" documentation](/developer_guides/execution_pipeline/manifests) for
details.

## Best practices

### Making /assets readonly

See [Secure coding](/developer_guides/security/secure_coding#filesystem)
