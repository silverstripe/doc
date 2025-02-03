---
title: Requirements
summary: How to include and require other resources in your templates such as javascript and CSS files.
iconBrand: js
---

# Requirements

The [`Requirements`](api:SilverStripe\View\Requirements) class takes care of including CSS and JavaScript into your applications. This is preferred to hard
coding any references in the `<head>` tag of your template, as it is more robust, allows for include templates to add stylesheets or JavaScript to the `<head>` of the resulting markup, and avoids duplicate resources.

The examples below use the naming conventions from the [Directory Structure](/getting_started/directory_structure/) section, but the `Requirements` class can work with arbitrary file paths.

## Exposing static resources

Before requiring static resource files in PHP code or in a template, those resources need to be "exposed". This process allows Silverstripe CMS projects and Silverstripe CMS modules to make static resource files available via the web server from locations that would otherwise be blocked from web server access, such as the `vendor` folder.

### Configuring your project "exposed" folders

Exposed resources are made available in your web root in a dedicated `_resources` directory. The name of the resources directory can be configured by defining the `extra.resources-dir` key in your `composer.json` file.

Each folder that needs to be exposed must be entered under the `extra.expose` key in your `composer.json` file. Module developers should use a path relative to the root of their module (don't include the `vendor/package-developer/package-name` path).

This is a sample Silverstripe CMS project `composer.json` file configured to expose some resources.

```json
{
    "name": "app/myproject",
    ...
    "extra": {
        "resources-dir": "_my-custom-resources-dir",
        "expose": [
            "app/client/dist",
            "app/client/lang"
        ]
    }
}
```

Files contained inside the `app/client/dist` and `app/client/lang` directories will be made publicly available under the `public/_resources` directory.

Silverstripe CMS projects should not track the `public/_resources` directory in their source control system.

### Exposing resources in the web root {#exposing-resources-webroot}

Silverstripe CMS projects ship with [silverstripe/vendor-plugin](https://github.com/silverstripe/vendor-plugin).
This Composer plugin automatically tries to expose resources from your project and installed modules after installation, or after an update.

Developers can explicitly expose static resources by calling `composer vendor-expose`. This is necessary after updating your `resources-dir` or `expose` configuration in your `composer.json` file.

`composer vendor-expose` accepts an optional `method` argument (e.g: `composer vendor-expose auto`). This controls how the files are exposed in the `_resources` directory:

- `none` disables all symlink / copy
- `copy` copies the exposed files
- `symlink` create symbolic links to the exposed folder
- `junction` uses a junction (Windows only)
- `auto` creates symbolic links (or junctions on Windows), but fails over to copy.

### Referencing exposed resources

When referencing exposed static resources, use either the project file path (relative to the project root folder) or a module name and relative file path to that module's root folder. e.g:

```php
use SilverStripe\View\Requirements;

// When referencing project files, use the same path defined in your `composer.json` file.
Requirements::javascript('app/client/dist/bundle.js');

// When referencing theme files, use a path relative to the root of your project
Requirements::javascript('themes/my-theme/javascript/script.js');

// When referencing files from a module, you need to prefix the path with the module name.
Requirements::javascript('silverstripe/admin:client/dist/js/bundle.js');
```

When rendered in HTML code, these URLs will be rewritten to their matching path inside the `public/_resources` directory.

## Template requirements API

You can require resources using the `require` template statement.

```ss
<%-- <my-module-dir>/templates/SomeTemplate.ss --%>
<% require css("<my-module-dir>/css/some_file.css") %>
<% require themedCSS("some_themed_file") %>
<% require javascript("<my-module-dir>/javascript/some_file.js") %>
```

Also see [Direct resource urls](#direct-resource-urls) below if you need to include the resource URL directly in your template.

> [!CAUTION]
> Requiring resources from the template is restricted compared to the PHP API.

## PHP requirements API

It is common practice to include most Requirements either in the `init()` method of your [controller](../controllers/), or
as close to rendering as possible (e.g. in [`FormField::Field()`](api:SilverStripe\Forms\FormField::Field())).

```php
namespace App\Control;

use SilverStripe\View\Requirements;

class MyCustomController extends Controller
{
    protected function init()
    {
        parent::init();

        Requirements::javascript('<my-module-dir>/javascript/some_file.js');
        Requirements::css('<my-module-dir>/css/some_file.css');
    }
}
```

### CSS files

```php
use SilverStripe\View\Requirements;

Requirements::css($path, $media);
```

If you're using the CSS method a second argument can be used. This argument defines the 'media' attribute of the
`<link>` element, so you can define 'screen' or 'print' for example.

```php
Requirements::css('<my-module-dir>/css/some_file.css', 'screen,projection');
```

### JavaScript files

```php
use SilverStripe\View\Requirements;

Requirements::javascript($path, $options);
```

A variant on the inclusion of custom JavaScript is the inclusion of *templated* JavaScript.  Here, you keep your
JavaScript in a separate file and instead load, via search and replace, several PHP generated variables into that code.

```php
use SilverStripe\Security\Security;
use SilverStripe\View\Requirements;

$vars = [
    'MemberID' => Security::getCurrentUser()->ID,
];

Requirements::javascriptTemplate('<my-module-dir>/javascript/some_file.js', $vars);
```

In this example, `some_file.js` is expected to contain a replaceable variable expressed as `$MemberID`.

If you are using front-end script combination mechanisms, you can optionally declare
that your included files provide these scripts. This will ensure that subsequent
Requirement calls that rely on those included scripts will not double include those
files.

```php
use SilverStripe\View\Requirements;

Requirements::javascript('<my-module-dir>/javascript/dist/bundle.js', ['provides' => [
    '<my-module-dir>/javascript/jquery.js'
    '<my-module-dir>/javascript/src/main.js',
    '<my-module-dir>/javascript/src/functions.js',
]]);
// Will skip this file
Requirements::javascript('<my-module-dir>/javascript/jquery.js');
```

You can also use the second argument to add the 'async' and/or 'defer attributes to the script tag generated:

```php
use SilverStripe\View\Requirements;

Requirements::javascript(
    '<my-module-dir>/javascript/some_file.js',
    [
        'async' => true,
        'defer' => true,
    ]
);
```

### Custom inline CSS or JavaScript

You can also quote custom scripts directly. This may seem a bit ugly, but is useful when you need to transfer some kind
of 'configuration' from the database in a raw format. Using the `heredoc` syntax to quote JS and CSS,
is cleaner than concatenating lots of strings together, and marks that section of code as belonging to a different
language.

```php
use SilverStripe\View\Requirements;

Requirements::customScript(<<<JS
  alert("hi there");
  JS
);

Requirements::customCSS(<<<CSS
  .tree li.$className {
    background-image: url($icon);
  }
  CSS
);
```

## Combining files

You can concatenate several CSS or JavaScript files into a single dynamically generated file. This increases performance
by reducing HTTP requests.

```php
use SilverStripe\View\Requirements;

Requirements::combine_files(
    'foobar.js',
    [
        '<my-module-dir>/javascript/foo.js',
        '<my-module-dir>/javascript/bar.js',
    ]
);
```

> [!CAUTION]
> To make debugging easier in your local environment, combined files is disabled when running your application in `dev`
> mode. You can re-enable dev combination by setting `Requirements_Backend.combine_in_dev` to true.

### Configuring combined file storage

Silverstripe CMS provides an API for combining multiple resource files together into a single file to reduce the number of network calls required.

> [!WARNING]
> It is generally accepted that if your webserver supports HTTP/2, multiple smaller resource files are better than a larger combined file. If you are using Apache, you will need to use php-fpm to support HTTP/2.

In some situations or server configurations, it may be necessary to customise the behaviour of generated JavaScript
files in order to ensure that current files are served in requests.

By default, files will be generated on demand in the format `assets/_combinedfiles/name-<hash>.js`,
where `<hash>` represents the hash of the source files used to generate that content. The default flysystem backend,
as used by the [`AssetStore`](api:SilverStripe\Assets\Storage\AssetStore) backend, is used for this storage, but it can be substituted for any
other backend.

> [!NOTE]
> Note that these combined files are stored as assets (by default in the `public/assets` directory), rather than being stored with other resources in your `public/_resources` directory.

You can also use any of the below options in order to tweak this behaviour:

- `Requirements.disable_flush_combined` - By default all combined files are deleted on flush.
   If combined files are stored in source control, and thus updated manually, you might want to
   turn this on to disable this behaviour.
- `Requirements_Backend.combine_hash_querystring` - By default the `<hash>` of the source files is appended to
   the end of the combined file (prior to the file extension). If combined files are versioned in source control,
   or running in a distributed environment (such as one where the newest version of a file may not always be
   immediately available) then it may sometimes be necessary to disable this. When this is set to true, the hash
   will instead be appended via a querystring parameter to enable cache busting, but not in the
   filename itself. I.e. `assets/_combinedfiles/name.js?m=<hash>`
- `Requirements_Backend.default_combined_files_folder` - This defaults to `_combinedfiles`, and is the folder
   within the configured requirements backend that combined files will be stored in. If using a backend shared with
   other systems, it is usually necessary to distinguish combined files from other assets.
- `Requirements_Backend.combine_in_dev` - By default combined files will not be combined except in test
   or live environments. Turning this on will allow for pre-combining of files in development mode.
- `Requirements_Backend.resolve_relative_css_refs` - Enables rewriting of relative paths to image/font resources
   to accommodate the fact that the combined CSS is placed in a totally different folder than the source CSS
   files. Disabled by default.

In some cases it may be necessary to create a new storage backend for combined files, if the default location
is not appropriate. Normally a single backend is used for all site assets, so a number of objects must be
replaced. For instance, the below will set a new set of dependencies to write to `app/javascript/combined`

```yml
---
Name: myrequirements
---
SilverStripe\View\Requirements:
  disable_flush_combined: true
SilverStripe\View\Requirements_Backend:
  combine_in_dev: true
  combine_hash_querystring: true
  default_combined_files_folder: 'combined'
  resolve_relative_css_refs: true
SilverStripe\Core\Injector\Injector:
  # Create adapter that points to the custom directory root
  SilverStripe\Assets\Flysystem\PublicAdapter.custom-adapter:
    class: SilverStripe\Assets\Flysystem\PublicAssetAdapter
    constructor:
      Root: ./app/javascript
  # Set flysystem filesystem that uses this adapter
  League\Flysystem\Filesystem.custom-filesystem:
    class: 'League\Flysystem\Filesystem'
    constructor:
      Adapter: '%$SilverStripe\Assets\Flysystem\PublicAdapter.custom-adapter'
  # Create handler to generate assets using this filesystem
  SilverStripe\Assets\Storage\GeneratedAssetHandler.custom-generated-assets:
    class: SilverStripe\Assets\Flysystem\GeneratedAssets
    properties:
      Filesystem: '%$League\Flysystem\Filesystem.custom-filesystem'
  # Assign this generator to the requirements builder
  SilverStripe\View\Requirements_Backend:
    properties:
      AssetHandler: '%$SilverStripe\Assets\Storage\GeneratedAssetHandler.custom-generated-assets'
```

In the above configuration, automatic expiry of generated files has been disabled, and it is necessary for
the developer to maintain these files manually. This may be useful in environments where assets must
be pre-cached, where scripts must be served alongside static files, or where no framework PHP request is
guaranteed. Alternatively, files may be served from instances other than the one which generated the
page response, and file synchronisation might not occur fast enough to propagate combined files to
mirrored filesystems.

In any case, care should be taken to determine the mechanism appropriate for your development
and production environments.

### Combined CSS files

You can also combine CSS files into a media-specific stylesheets as you would with the `Requirements::css()` call - use
the third parameter of the `combine_files` function:

```php
use SilverStripe\View\Requirements;
use SilverStripe\View\SSViewer;
use SilverStripe\View\ThemeResourceLoader;

$loader = ThemeResourceLoader::inst();
$themes = SSViewer::get_themes();

$printStylesheets = [
    $loader->findThemedCSS('print_HomePage.css', $themes),
    $loader->findThemedCSS('print_Page.css', $themes),
];

Requirements::combine_files('print.css', $printStylesheets, 'print');
```

By default, all requirements files are flushed (deleted) when manifests are flushed (see [Flushing](/developer_guides/execution_pipeline/manifests/#flushing)).
This can be disabled by setting the `Requirements.disable_flush_combined` config to `true`.

> [!CAUTION]
> When combining CSS files, take care of relative urls, as these will not be re-written to match
> the destination location of the resulting combined CSS unless you have set the
> `Requirements_Backend.resolve_relative_css_refs` configuration property to `true`.

### Combined JS files

You can also add the 'async' and/or 'defer' attributes to combined JavaScript files as you would with the
`Requirements::javascript()` call - use the third parameter of the `combine_files` function:

```php
use SilverStripe\View\Requirements;
use SilverStripe\View\SSViewer;
use SilverStripe\View\ThemeResourceLoader;

$loader = ThemeResourceLoader::inst();
$themes = SSViewer::get_themes();

$scripts = [
    $loader->findThemedJavascript('some_script.js', $themes),
    $loader->findThemedJavascript('some_other_script.js', $themes),
];

Requirements::combine_files('scripts.js', $scripts, ['async' => true, 'defer' => true]);
```

## Clearing resources

```php
use SilverStripe\View\Requirements;

Requirements::clear();
```

Clears all defined requirements. You can also clear specific requirements.

```php
use SilverStripe\View\Requirements;

Requirements::clear('modulename/javascript/some-lib.js');
```

> [!CAUTION]
> Depending on where you call this command, a Requirement might be *re-included* afterwards.

## Blocking

Requirements can also be explicitly blocked from inclusion, which is useful to avoid conflicting JavaScript logic or
CSS rules. These blocking rules are independent of where the `block()` call is made. It applies both for already
included requirements, and ones included after the `block()` call.

One common example is to block `jquery.js` which would otherwise be added to the front-end by various modules - for example if you already have jQuery in your frontend and don't want multiple copies.

```php
Requirements::block('some/module:client/dist/jquery.js');
```

> [!CAUTION]
> The CMS also uses the `Requirements` system, and its operation can be affected by `block()` calls. Avoid this by
> limiting the scope of your blocking operations, e.g. in `init()` of your controller.

## Inclusion order

Requirements acts like a stack, where everything is rendered sequentially in the order it was included. There is no way
to change inclusion-order, other than using *Requirements::clear* and rebuilding the whole set of requirements.

> [!CAUTION]
> Inclusion order is both relevant for CSS and JavaScript files in terms of dependencies, inheritance and overlays - be
> careful when messing with the order of requirements.

## JavaScript placement

By default, Silverstripe CMS includes all JavaScript files at the bottom of the page body, unless there's another script
already loaded, then, it's inserted before the first `<script>` tag. If this causes problems, it can be configured.

```php
use SilverStripe\View\Requirements;

Requirements::set_force_js_to_bottom(true);
```

`Requirements.force_js_to_bottom`, will force Silverstripe CMS to write the JavaScript to the bottom of the page body, even
if there is an earlier script tag.

If the JavaScript files are preferred to be placed in the `<head>` tag rather than in the `<body>` tag,
`Requirements.write_js_to_body` should be set to false.

```php
use SilverStripe\View\Requirements;

Requirements::set_write_js_to_body(false);
```

## Direct resource urls

In templates, you can use the [`$themedResourceURL()`](api:SilverStripe\View\ThemeResourceLoader::themedResourceURL()) or [`$resourceURL()`](api:SilverStripe\Core\Manifest\ModuleResourceLoader::resourceURL()) helper methods to inject links to
resources directly.

If you want to get a resource using cascading themes, use `$themedResourceURL()`:

```ss
<img src="$themedResourceURL('images/my-image.jpg')">
<img src="$themedResourceURL('images')/$Image.jpg">
```

If you want to get a resource for a *specific* theme or from somewhere that is not a theme (your app directory or a module), use `$resourceURL()`:

```ss
<img src="$resourceURL('app/images/my-image.jpg')">
<img src="$resourceURL('my/module:images/my-image.jpg')">
<img src="$resourceURL('themes/my-theme/images/my-image.jpg')">
<img src="$resourceURL('themes/my-theme/images')/$Image.jpg">
```

> [!TIP]
> Notice the `vendor/module:some/path/to/file.jpg` syntax (used to get a resource from a specific module) is only valid for the `$resourceURL()` helper method. It won't work for `$themedResourceURL()`.

### Resource URLs or filepaths from a PHP context

In PHP you can directly resolve urls and file paths for resources using the [`ModuleResourceLoader`](api:SilverStripe\Core\Manifest\ModuleResourceLoader) and [`ThemeResourceLoader`](api:SilverStripe\View\ThemeResourceLoader) helpers.

```php
use SilverStripe\Core\Manifest\ModuleResourceLoader;
use SilverStripe\View\ThemeResourceLoader;

// Get the URL or relative file path for an image in the silverstripe/admin module
$fileUrl = ModuleResourceLoader::singleton()->resolveURL('silverstripe/admin:client/dist/images/spinner.gif');
$filePath = ModuleResourceLoader::singleton()->resolvePath('silverstripe/admin:client/dist/images/spinner.gif');

// Get the URL or relative file path for an image in a theme, using cascading themes
$themeFileUrl = ThemeResourceLoader::themedResourceURL('images/spinner.gif');
$themeFilePath = ThemeResourceLoader::inst()->findThemedResource('images/spinner.gif');
```

You can also get file paths specifically for JavaScript and CSS files using the [`findThemedJavascript()`](api:SilverStripe\Core\Manifest\ModuleResourceLoader::findThemedJavascript()) and [`findThemedCss()`](api:SilverStripe\Core\Manifest\ModuleResourceLoader::findThemedCss()) methods.

## API documentation

- [Requirements](api:SilverStripe\View\Requirements)
- [CMS Architecture and Build Tooling](/developer_guides/customising_the_admin_interface/cms_architecture)
