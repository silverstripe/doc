---
title: Rich-text editing (WYSIWYG)
summary: Silverstripe CMS's use and configuration of TinyMCE html editor.
icon: file-code
---

# Rich-text editing (WYSIWYG)

Editing and formatting content is the bread and butter of every content management system, which is why Silverstripe CMS
has a tight integration with our preferred editor library, [TinyMCE](https://www.tiny.cloud/docs/tinymce/6/).

On top of the base functionality, we use our own insertion dialogs to ensure you can effectively select and upload
files. We also use [shortcodes](/developer_guides/extending/shortcodes) to store
information about inserted images or media elements.

The framework comes with a [HTMLEditorField](api:SilverStripe\Forms\HTMLEditor\HTMLEditorField) form field class which encapsulates most of the required
functionality. It is usually added through the [DataObject::getCMSFields()](api:SilverStripe\ORM\DataObject::getCMSFields()) method:

```php
// app/src/Model/MyObject.php
namespace App\Model;

use SilverStripe\Forms\FieldList;
use SilverStripe\Forms\HTMLEditor\HTMLEditorField;
use SilverStripe\ORM\DataObject;

class MyObject extends DataObject
{
    private static $db = [
        'Content' => 'HTMLText',
    ];

    public function getCMSFields()
    {
        return FieldList::create(
            HTMLEditorField::create('Content')
        );
    }
}
```

## Specify which configuration to use

By default, a config named 'CMS' is used in any new [HTMLEditorField](api:SilverStripe\Forms\HTMLEditor\HTMLEditorField).

If you have created your own [HTMLEditorConfig](api:SilverStripe\Forms\HTMLEditor\HTMLEditorConfig) and would like to use it,
you can call [`HTMLEditorConfig::set_active('myConfig')`](api:SilverStripe\Forms\HTMLEditor\HTMLEditorConfig::set_active()) and all subsequently created `HTMLEditorField` instances
will use the configuration with the name 'myConfig'.

You can also specify which `HTMLEditorConfig` to use on a per field basis via the construct argument.
This is particularly useful if you need different configurations for multiple `HTMLEditorField` on the same page or form.

```php
namespace App\Model;

use SilverStripe\Forms\FieldList;
use SilverStripe\Forms\HTMLEditor\HTMLEditorField;
use SilverStripe\ORM\DataObject;

class MyObject extends DataObject
{
    private static $db = [
        'Content' => 'HTMLText',
        'OtherContent' => 'HTMLText',
    ];

    public function getCMSFields()
    {
        return FieldList::create([
            HTMLEditorField::create('Content'),
            HTMLEditorField::create('OtherContent', 'Other content', $this->OtherContent, 'myConfig'),
        ]);
    }
}
```

In the above example, the 'Content' field will use the default 'CMS' config while 'OtherContent' will be using 'myConfig'.

## Configuration

To keep the JavaScript editor configuration manageable and extensible, we've wrapped it in a PHP class called
[HTMLEditorConfig](api:SilverStripe\Forms\HTMLEditor\HTMLEditorConfig). The class comes with its own defaults, which are extended through the [Configuration API](../../configuration)
in the framework (and the `cms` module in case you've got that installed).

There can be multiple configs, which should always be created / accessed using [HTMLEditorConfig::get()](api:SilverStripe\Forms\HTMLEditor\HTMLEditorConfig::get()). You can
then set the currently active config using `HTMLEditorConfig::set_active()`.

> [!WARNING]
> The order in which the `_config.php` files are executed depends on the module names. Execution
> order is alphabetical, so if you set a TinyMCE option in the `aardvark/_config.php` (i.e. the module name is simply `aardvark`),
> this will be overridden in `vendor/silverstripe/admin/_config.php` (because the module name is `silverstripe/admin`) and your modification will disappear.

## Adding and removing capabilities

In its simplest form, the configuration of the editor includes adding and removing buttons and plugins.

You can add plugins to the editor using the Framework's [TinyMCEConfig::enablePlugins()](api:SilverStripe\Forms\HTMLEditor\TinyMCEConfig::enablePlugins()) method. This will
transparently generate the relevant underlying TinyMCE code.

> [!TIP]
> The `enablePlugins()` method is implemented on `TinyCMEConfig`, which is a subclass of `HTMLEditorConfig`. This is true of most of the configuration methods used in this documentation. We've done an explicit `instanceof` check here for correctness, but in reality unless your project introduces an alternative WYSIWYG editor, you can safely omit that check. The remaining examples in this documentation will omit the check.

```php
// app/_config.php
use SilverStripe\Forms\HTMLEditor\HTMLEditorConfig;
use SilverStripe\Forms\HTMLEditor\TinyMCEConfig;

$editorConfig = HTMLEditorConfig::get('cms');
if ($editorConfig instanceof TinyMCEConfig) {
    $editorConfig->enablePlugins('emoticons');
}
```

> [!WARNING]
> This utilities the TinyMCE's [external_plugins](https://www.tiny.cloud/docs/tinymce/6/editor-important-options/#external_plugins)
> option under the hood.

Plugins and advanced themes can provide additional buttons that can be added (or removed) through the
configuration. Here is an example of adding a `ssmacron` button after the `charmap` button:

```php
// app/_config.php
use SilverStripe\Forms\HTMLEditor\TinyMCEConfig;
TinyMCEConfig::get('cms')->insertButtonsAfter('charmap', 'ssmacron');
```

Buttons can also be removed:

```php
// app/_config.php
use SilverStripe\Forms\HTMLEditor\TinyMCEConfig;
TinyMCEConfig::get('cms')->removeButtons('tablecontrols', 'blockquote', 'hr');
```

> [!WARNING]
> Internally `HTMLEditorConfig` uses the TinyMCE's `toolbar` option to configure these. See the
> [TinyMCE documentation of this option](https://www.tiny.cloud/docs/tinymce/6/toolbar-configuration-options/#toolbar)
> for more details.

### Setting options

TinyMCE behaviour can be affected through its [configuration options](https://www.tiny.cloud/docs/tinymce/6/basic-setup).
These options will be passed straight to the editor.

A default set of options has been defined in the [`TinyMCEConfig.default_options`](api:SilverStripe\Forms\HTMLEditor\TinyMCEConfig->default_options) configuration property.

One example of the usage of this capability is to redefine the TinyMCE's [whitelist of HTML
tags](https://www.tiny.cloud/docs/tinymce/6/content-filtering/#extended_valid_elements) - the tags that will not be stripped
from the HTML source by the editor.

```php
// app/_config.php
use SilverStripe\Forms\HTMLEditor\TinyMCEConfig;

// Add start and type attributes for <ol>, add <embed> with all attributes.
TinyMCEConfig::get('cms')->setOption(
    'extended_valid_elements',
    'img[class|src|alt|title|hspace|vspace|width|height|align|name|usemap|data*],' .
    'iframe[src|name|width|height|align|frameborder|marginwidth|marginheight|scrolling],' .
    'object[width|height|data|type],' .
    'embed[src|type|pluginspage|width|height|autoplay],' .
    'param[name|value],' .
    'map[class|name|id],' .
    'area[shape|coords|href|target|alt],' .
    'ol[start|type]'
);
```

Note that the `setOption()` *overrides* any existing value for that option. If you only want to change some small part
of the existing option value, you can call `getOption()`, modify the returned value, and then pass the result to `setOption()`.

```php
// app/_config.php
use SilverStripe\Forms\HTMLEditor\TinyMCEConfig;

// Add start and type attributes for <ol>, add <embed> with all attributes - without redeclaring everything else
$editor = TinyMCEConfig::get('cms');
$validElements = $editor->getOption('extended_valid_elements') . ',' .
    'embed[src|type|pluginspage|width|height|autoplay],' .
    'ol[start|type]';
$validElements = str_replace('iframe[', 'iframe[data-*|');
$editor->setOption('extended_valid_elements', $validElements);
```

> [!WARNING]
> The default setting for the CMS's `extended_valid_elements` we are overriding here can be found in
> `vendor/silverstripe/admin/_config.php`.

## Enabling custom plugins

It is also possible to add custom plugins to TinyMCE, for example toolbar buttons.
You can enable them through [TinyMCEConfig::enablePlugins()](api:SilverStripe\Forms\HTMLEditor\TinyMCEConfig::enablePlugins()):

```php
// app/_config.php
use SilverStripe\Forms\HTMLEditor\TinyMCEConfig;
TinyMCEConfig::get('cms')->enablePlugins(['myplugin' => 'app/javascript/myplugin/editor_plugin.js']);
```

> [!TIP]
> The path for the plugin file must be one of the following:
>
> - `null` (if the plugin being enabled is a built-in plugin)
> - a path, relative to your `_resources/` directory, to the plugin file
> - a `ModuleResource` instance representing the plugin JavaScript file (see `silverstripe/admin`'s `_config.php` file for examples)
> - an absolute URL (e.g. for a third-party plugin to be fetched from a CDN).

You can learn how to [create a plugin](https://www.tiny.cloud/docs/tinymce/6/creating-a-plugin/) from the TinyMCE documentation.

## Image and media insertion

The [HTMLEditorField](api:SilverStripe\Forms\HTMLEditor\HTMLEditorField) API also handles inserting images and media files into the managed HTML content. It can be
used both for referencing files on the webserver filesystem (through the [File](api:SilverStripe\Assets\File) and [Image](api:SilverStripe\Assets\Image) APIs), as well
as hotlinking files from the web.

We use [shortcodes](/developer_guides/extending/shortcodes) to store information about inserted images or media elements. The
[ShortcodeParser](api:SilverStripe\View\Parsers\ShortcodeParser) API post-processes the HTML content on rendering, and replaces the shortcodes accordingly. It also
takes care of care of placing the shortcode replacements relative to its surrounding markup (e.g. left/right alignment).

### Image size pre-sets

Silverstripe CMS will suggest pre-set image size in the HTMLEditor. Editors can quickly switch between the pre-set size when interacting with images in the HTMLEditorField.

The default values are "Best fit" (600 pixels width) and original size. Developers can customise the pre-set sizes by altering their TinyMCEConfig.

You can alter the defaults for all HTMLEditor in your YAML configuration.

```yml
SilverStripe\Forms\HTMLEditor\TinyMCEConfig:
  image_size_presets:
    - name: widesize
      i18n: SilverStripe\Forms\HTMLEditor\TinyMCEConfig.WIDE_SIZE
      text: Wide size
      width: 900
```

You can edit the image size pre-sets for an individual configuration with this code snippet.

```php
use SilverStripe\Forms\HTMLEditor\TinyMCEConfig;

TinyMCEConfig::get('cms')->setOption('image_size_presets', [
    [
        'width' => 300,
        'text' => 'Small fit',
        'name' => 'smallfit',
        'default' => true,
    ],
    [
        'width' => 600,
        'i18n' => TinyMCEConfig::class . '.BEST_FIT',
        'text' => 'Best fit',
        'name' => 'bestfit',
    ],
    [
        'i18n' => TinyMCEConfig::class . '.ORIGINAL_SIZE',
        'text' => 'Original size',
        'name' => 'originalsize',
    ],
]);
```

## OEmbed: embedding media through external services

The ["oEmbed" standard](https://www.oembed.com/) is implemented by many media services around the web, allowing easy
representation of files just by referencing a website URL. For example, a content author can insert a playable youtube
video just by knowing its URL, as opposed to dealing with manual HTML code.

oEmbed powers the "Insert from web" feature available through
[HTMLEditorField](api:SilverStripe\Forms\HTMLEditor\HTMLEditorField). Internally this service is provided
by the [embed](https://github.com/oscarotero/Embed) library.

To disable oembed you will need to follow the below to remove the plugin from TinyMCE, as well
as disabling the internal service via YAML:

```yml
---
Name: oembed-disable
---
SilverStripe\AssetAdmin\Forms\RemoteFileFormFactory:
  enabled: false
```

```php
use SilverStripe\Forms\HTMLEditor\TinyMCEConfig;
TinyMCEConfig::get('cms')->disablePlugins('ssembed');
```

Use the following config if you need to send outbound requests through a proxy:

```yml
---
Name: myembed
After: coreoembed
---
SilverStripe\Core\Injector\Injector:
  Psr\Http\Client\ClientInterface.oembed:
    constructor:
      - proxy: '111.222.333.444:55'
```

### Limiting oembed URLs

HTMLEditorField can have whitelists set on both the scheme (default HTTP & HTTPS) and domains allowed when
inserting files for use with oembed.

This is performed through the config variables on the
[RemoteFileFormFactory](api:SilverStripe\AssetAdmin\Forms\RemoteFileFormFactory) class:

```yml
---
Name: oembed-restrictions
---
SilverStripe\AssetAdmin\Forms\RemoteFileFormFactory:
  fileurl_scheme_whitelist:
    - https
    - http
  fileurl_scheme_blacklist:
    - ftp
  fileurl_domain_whitelist:
    - google.com
  fileurl_domain_blacklist:
    - localhost
  fileurl_port_whitelist:
    - 80
    - 443
  fileurl_port_blacklist:
    - 23
```

This allows a white or blacklist to be applied to schema, domain, or port (if provided). Note that
both blacklist and whitelist need to match, and are only ignored if the rules are empty for any
of the above values.

By default live sites (see [environment types](/developer_guides/debugging/environment_types/)) will not attempt to resolve oembed urls that
point to localhost to protect your site from cross site request forgery.

### Sandboxing oembed HTML

In order to prevent any malicious oembed providers from injecting XSS payloads into the current webpage, HTML content that is returned is sandboxed in an `iframe` tag.

With the [`EmbedShortcodeProvider.domains_excluded_from_sandboxing`](api:SilverStripe\View\Shortcodes\EmbedShortcodeProvider->domains_excluded_from_sandboxing) configuration property, you can explicitly declare domains which should be excluded from sandboxing if you find it is interfering with embeds from specific domains. For example if a YouTube embed was not rendering correctly as a result of the sandboxing you could use this YAML configuration:

```yml
SilverStripe\View\Shortcodes\EmbedShortcodeProvider:
  domains_excluded_from_sandboxing:
    - 'youtube.com'
```

Do not include the protocol (i.e. don't include `https://` or `http://`).

You can also change the attributes of the iframe itself with the [`EmbedShortcodeProvider.sandboxed_iframe_attributes`](api:SilverStripe\View\Shortcodes\EmbedShortcodeProvider->sandboxed_iframe_attributes) configuration property:

```yml
SilverStripe\View\Shortcodes\EmbedShortcodeProvider:
  sandboxed_iframe_attributes:
    allow: 'fullscreen'
```

## Doctypes

Since TinyMCE generates markup, it needs to know which doctype your documents will be rendered in. You can set this
through the [element_format](https://www.tiny.cloud/docs/tinymce/6/content-filtering/#element_format) configuration variable.

In case you want to adhere to the stricter xhtml format (for example rendering self closing tags like `<br/>` instead of `<br>`),
use the following configuration:

```php
use SilverStripe\Forms\HTMLEditor\TinyMCEConfig;
TinyMCEConfig::get('cms')->setOption('element_format', 'xhtml');
```

By default, TinyMCE and Silverstripe CMS will generate valid HTML5 markup, but it will strip out many HTML5 tags like
`<article>` or `<figure>`. If you plan to use those, add them to the
[valid_elements](https://www.tiny.cloud/docs/tinymce/6/content-filtering/#valid_elements) configuration setting.

Also, the [HTMLValue](api:SilverStripe\View\Parsers\HTMLValue) API underpinning the HTML processing parses the markup into a temporary object tree
which can be traversed and modified before saving. The built-in parser supports HTML5 syntax.

## Security groups with their own editor configuration

Different groups of authors can be assigned their own config,
e.g. a more restricted rule set for content reviewers (see [access control](/developer_guides/security/access_control/#the-security-groups-in-silverstripe-cms)).
The config is available on each user record through [Member::getHTMLEditorConfigForCMS()](api:SilverStripe\Security\Member::getHTMLEditorConfigForCMS()).
The group assignment is done through the "Security" interface for each [Group](api:SilverStripe\Security\Group) record.
Note: The dropdown is only available if more than one config exists.

## Customising modal forms

In the standard installation, you can insert links (internal/external/anchor/email),
images as well as flash media files. The forms used for preparing the new content element
are rendered by Silverstripe CMS, but there's some JavaScript involved to transfer
back and forth between a content representation the editor can understand, present and save.

The forms for these actions are created using implementations of the [`FormFactory`](api:SilverStripe\Forms\FormFactory) interface.
For example, the module for embedding remote files gets its form from the [`RemoteFileFormFactory`](api:SilverStripe\AssetAdmin\Forms\RemoteFileFormFactory) class.

All of these forms can be customised by implementing an extension with the appropriate
extension hook method.

Example: Remove field for "Caption" in the embedded image form:

```php
// app/src/Extension/MyToolbarExtension.php
namespace App\Extension;

use SilverStripe\Core\Extension;
use SilverStripe\Forms\Form;

class RemoteFileFormFactoryExtension extends Extension
{
    public function updateForm(Form $form)
    {
        $form->Fields()->removeByName('CaptionText');
    }
}
```

```yml
# app/_config/extensions.yml
SilverStripe\AssetAdmin\Forms\RemoteFileFormFactory:
  extensions:
    - App\Extension\RemoteFileFormFactoryExtension
```

Adding functionality is a bit more advanced, you'll most likely
need to add some fields to the PHP forms, as well as write some
JavaScript to ensure the values from those fields make it into the content
elements (and back out in case an existing element gets edited).

## Using the `HTMLEditorField` outside of the CMS

The `HTMLEditorField` is configured for use in the CMS interface - but it can be used in other contexts
as well, with some additional configuration. Note however that use of the `ssmedia` and `sslink` and
related plugins is not directly supported outside of the CMS. Your best bet is to configure a custom
`HTMLEditorConfig` for this purpose which doesn't include those plugins.

You will also need to provide some basic JavaScript to initialise the TinyMCE field. The below JavaScript
will initialise TinyMCE configuration for every `HTMLEditorField` on the page:

```js
// eslint-disable-next-line no-restricted-syntax
for (const field of document.querySelectorAll('textarea[data-editor="tinyMCE"]')) {
  const id = field.getAttribute('id');
  const config = JSON.parse(field.dataset.config);
  config.selector = `#${id}`;
  if (typeof config.baseURL !== 'undefined') {
    tinymce.EditorManager.baseURL = config.baseURL;
  }
  tinymce.init(config);
}
```

## Developing a wrapper to use a different WYSIWYG editors with `HTMLEditorField`

WYSIWYG editors are complex beasts, so replacing it completely is a difficult task.
The framework provides a wrapper implementation for the basic required functionality,
mainly around selecting and inserting content into the editor view.
Have a look in `HTMLEditorField.js` and the `ss.editorWrapper` object to get you started
on your own editor wrapper. Note that the JavaScript for the `HTMLEditorField` is currently hardwired to support TinyMCE,
and it is likely that a lot of projects and modules will be expecting all `HTMLEditorConfig`
instances to be `TinyMCEConfig` instances.
