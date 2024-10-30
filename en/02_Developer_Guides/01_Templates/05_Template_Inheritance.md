---
title: Template Inheritance
summary: Override and extend module and core markup templates from your application code.
icon: sitemap
---

# Template inheritance

## Theme types

Templates in Silverstripe CMS are bundled into one of two groups:

- default Templates, such as those provided in `vendor/silverstripe/framework/templates/` or `app/templates/` folders
- theme templates, such as those provided in `themes/mytheme/templates/` folders.

The default templates provide basic HTML formatting for elements such as Forms, Email, or RSS Feeds, and provide a
generic base for web content to be built on. They are also used to generate markup for many sections of the CMS.

## Template types and locations

Often templates will have the same name as the class they are used to render. So, your `Page` class will
be rendered with the `templates/Page.ss` template.

When the class has a namespace, the namespace will be interpreted as a subfolder within the `templates` path.
For example, the class `SilverStripe\Control\Controller` will be rendered with the
`templates/SilverStripe/Control/Controller.ss` template.

If you are using template "types" like `Layout` or `Includes`, these are just folders which you need
to append to your template file location (e.g. `templates/Layout/Page.ss` and `templates/SilverStripe/Control/Layout/Controller.ss` in the examples above).

Note however that when using the `<% include %>` template tag, you don't reference the `Includes/` folder. So for a template located at `templates/SilverStripe/Blog/Includes/BlogSideBar.ss` you would include it as `<% include SilverStripe\Blog\BlogSideBar %>`

When choosing templates to use, unless a file has been explicitly declared, Silverstripe CMS will look through all of the relevant templates until it finds a match.
In the CMS, the priority order is determined by the `LeftAndMain.admin_themes` configuration array, which includes all of the default templates but typically *excludes* theme templates. On the front-end, it is determined by the `SSViewer.themes` configuration array. See [Cascading themes](#cascading-themes) for more information.

### Nested layouts through `$Layout` type

Silverstripe CMS has basic support for nested layouts through a fixed template variable named `$Layout`. It's used for
keeping individual page layouts separate from top level template information (where you'll put your `<head>` and top-level nagivation, etc).

When `$Layout` is found within a root template file (ie. a template file directly in `templates/`), Silverstripe CMS will attempt to fetch a child
template from the `templates/<namespace>/Layout/<class>.ss` path, where `<namespace>` and `<class>` represent
the class being rendered. It will check for templates matching the current classname first, and if none are found it will search up the class hierarchy until
it finds a template.

This is better illustrated with an example. Take for instance our website that has two page types `Page` and `App\PageType\HomePage`.

Our site looks mostly the same across both templates with just the main content in the middle changing. The header,
footer and navigation will remain the same and we don't want to replicate this work across more than one spot. The
`$Layout` template variable allows us to define the child template area which wil be replaced with the relevant Layout template.

```ss
<%-- app/templates/Page.ss --%>
<html>
<head>
    ..
</head>

<body>
    <% include Header %>
    <% include Navigation %>

    $Layout

    <% include Footer %>
</body>
```

```ss
<%-- app/templates/Layout/Page.ss --%>

<p>You are on a $ClassName.ShortName page</p>

$Content
```

```ss
<%-- app/templates/App/PageType/Layout/HomePage.ss --%>
<h1>This is the homepage!</h1>

<strong>Hi!</strong>
```

## Cascading themes

There are potentially multiple themes, and definitely multiple modules with `template/` directories, in a typical
Silverstripe project. Across all of these template directories, there could be multiple file paths matching any
given template name.
For this reason, a cascading search is done to determine the resolved template for any specified template name.

In order to declare the priority for this search, themes can be declared in a cascading fashion in order
to determine resolution priority. This search is based on the following three configuration values:

- `SilverStripe\View\SSViewer.themes` - The list of all themes in order of priority (highest first).
   This includes the default set via `$default` as a theme set. This config is normally set by the web
   developer.
  - Note: In the admin panel (aka the CMS aka the backend), `SilverStripe\Admin\LeftAndMain.admin_themes` is used instead.
    It explicitly *does not* include front-end themes.
- `SilverStripe\Core\Manifest\ModuleManifest.project` - The name of the `$project` module, which
   defaults to `app`.
- `SilverStripe\Core\Manifest\ModuleManifest.module_priority` - The list of modules within which `$default`
   theme templates should be sorted, in order of priority (highest first). This config is normally set by
   the module author, and does not normally need to be customised. This includes the `$project` and
   `$other_modules` placeholder values.

### Declaring themes

All themes can be enabled and sorted via the `SilverStripe\View\SSViewer.themes` config value. For reference
on what syntax styles you can use for this value please see the [Configuring themes](./themes#configuring-themes) documentation.

Basic example:

```yml
# app/_config/themes.yml
---
Name: mytheme
---
SilverStripe\View\SSViewer:
  themes:
    - theme_name
    - '$default'
```

### Declaring module priority

The order in which templates are selected from modules can be explicitly declared
through configuration. To specify the order you want, make a list of the module
names under `SilverStripe\Core\Manifest\ModuleManifest.module_priority` in a
configuration YAML file.

Note: In order for modules to sort relative to other modules, it's normally necessary
to provide `before:` / `after:` declarations.

```yml
# mymodule/_config/config.yml
Name: modules-mymodule
After:
  - '#modules-framework'
  - '#modules-other'
---
SilverStripe\Core\Manifest\ModuleManifest:
  module_priority:
    - myvendor/mymodule
```

In this example, your module has applied its priority *lower* than the framework and "other" modules, meaning template lookup
will only defer to your modules templates folder if not found elsewhere.

You can use `Before` to declare that your module should take precedence over other modules' templates.

### Declaring project

The default project structure contains an `app/` folder, which (if you used `silverstripe/installer` to generate your project)
is automatically declared as the "project" via the `SilverStripe\Core\Manifest\ModuleManifest.project` configuration property.
It effectively acts as as a module in terms of template priorities, so declaring it explicitly as the project means it can be handled
specially.

For example, in the `ModuleManifest.module_priority` configuration mentioned above, the `$project` placeholder value is used to
represent the project in the priorty order.

See [Directory Structure](/getting_started/directory_structure) to find out how to declare a folder with some arbitrary name as the project.

### About module "names"

Module names are derived from their local `composer.json` files using the following precedence:

- The value of the `name` attribute in `composer.json`
- The value of `extras.installer-name` in `composer.json`
- The basename of the directory that contains the module
