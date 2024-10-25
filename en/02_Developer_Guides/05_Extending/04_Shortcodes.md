---
title: Shortcodes
summary: Flexible content embedding
icon: code
---

# Shortcodes

The [ShortcodeParser](api:SilverStripe\View\Parsers\ShortcodeParser) API is simple parser that allows you to map specifically formatted content to a callback to
transform them into something else. You might know this concept from forum software which don't allow you to insert
direct HTML, instead resorting to a custom syntax.

In the CMS, authors often want to insert content elements which go beyond standard formatting, at an arbitrary position
in their WYSIWYG editor. Shortcodes are a semi-technical solution for this. A good example would be embedding a 3D file
viewer or a Google Map at a certain location.

In this example, `[map]` represents the shortcode from [How to Create a Google Maps Shortcode](how_tos/create_a_google_maps_shortcode).

```php
$text = '<h1>My Map</h1>[map]'

// Will output
// <h1>My Map</h1><iframe ... ></iframe>
```

The following are all valid syntax for shortcodes. Some shortcodes accept parameters or wrap content, while others don't.

```text
# The most simple shortcodes don't need any parameters or wrap any content
[my_shortcode]

# A closing slash is allowed for legacy reasons
[my_shortcode /]

# Parameters can be included like so:
[my_shortcode,myparameter="value"]

# If you wrap any content in a shortcode, you need to have a closing tag
[my_shortcode,myparameter="value"]Enclosed Content[/my_shortcode]
```

Shortcodes are automatically parsed on any database field which is declared as [`HTMLText`](api:SilverStripe\ORM\FieldType\DBHTMLText)
when rendered into a template (if you want a `HTMLText` field that doesn't parse shortcodes, use `HTMLFragment`).
This can also be enabled for [`HTMLVarchar`](api:SilverStripe\ORM\FieldType\DBHTMLVarchar). This means you can use shortcodes on common fields like `SiteTree.Content`, and any
other [`$db`](api:SilverStripe\ORM\DataObject::$db) definitions of these types.

Other fields can be manually parsed with shortcodes through the `parse` method.

```php
use SilverStripe\View\Parsers\ShortcodeParser;

$text = 'My awesome [my_shortcode] is here.';
ShortcodeParser::get_active()->parse($text);
```

## Enabling shortcode parsing for `HTMLVarchar`

If you want *all* `HTMLVarchar` fields to automatically process shortcodes when rendered in templates, you can enable this globally like so:

```yml
SilverStripe\Core\Injector\Injector:
  HTMLVarchar:
    properties:
      ProcessShortcodes: true
```

If you want only some specific `HTMLVarchar` fields to process shortcodes, you can either do that by declaring a *new* field type:

```yml
SilverStripe\Core\Injector\Injector:
  HTMLVarcharWithShortcodes:
    class: SilverStripe\ORM\FieldType\DBHTMLVarchar
    properties:
      ProcessShortcodes: true
```

or by defining a "getter" method for the relevant fields. This example assumes you have a db field named `MyHtmlVarcharField`:

```php
public function getMyHtmlVarcharField()
{
    $field = $this->dbObject('MyHtmlVarcharField');
    $field->setProcessShortcodes(true);
    return $field;
}
```

> [!NOTE]
> See [Data types and Casting](/developer_guides/model/data_types_and_casting/#overriding) for more information about getter methods.

## Defining custom shortcodes
With custom shortscodes it is not only possible to add new shortcodes but it is also possible to edit the default behaviour of the CMS.


First we need to define a callback for the shortcode. These callbacks are usually static methods on some class, but they can also be anonymous functions or any other valid PHP callback.

```php
// app/src/ShortCode/MyShortCodeProvider.php
namespace App\ShortCode;

class MyShortCodeProvider
{
    public static function parseMyShortCode($arguments, $content = null, $parser = null, $tagName = null)
    {
        return '<em>' . $tagName . '</em> ' . $content . '; ' . count($arguments) . ' arguments.';
    }
}
```

Note that the `$casting` configuration here is optional - it's used in this case to allow directly calling this method from a template. It doesn't affect the actual shortcode functionality at all.

> [!WARNING]
> Note that the `$arguments` parameter potentially contains any arbitrary key/value pairs the user has chosen to include.
> It is strongly recommended that you don't directly convert this array into a list of attributes for your final HTML markup
> as that could lead to XSS vulnerabilities in your project.
>
> If you want to use the `$arguments` parameter as a list of attributes for your final HTML markup, it is strongly recommended that you
> pass the array through a filter of allowed arguments using [array_filter()](https://www.php.net/manual/en/function.array-filter.php)
> or similar.

These parameters are passed to the `parseMyShortCode` callback:

- Any parameters attached to the shortcode as an associative array (keys are lower-case).
- Any content enclosed within the shortcode (if it is an enclosing shortcode). Note that any content within this
   will not have been parsed, and can optionally be fed back into the parser.
- The `ShortcodeParser` instance used to parse the content.
- The shortcode tag name that was matched within the parsed content.
- An associative array of extra information about the shortcode being parsed. For example, if the shortcode is
   is inside an attribute, the `element` key contains a reference to the parent `DOMElement`, and the `node`
   key the attribute's `DOMNode`.

To register a shortcode you call the following.

```php
// app/_config.php
use App\ShortCode\MyShortCodeProvider;
use SilverStripe\View\Parsers\ShortcodeParser;

ShortcodeParser::get('default')->register('my_shortcode', [MyShortCodeProvider::class, 'parseMyShortCode']);
```

> [!NOTE]
> Note that `my_shortcode` is an arbitrary name which can be made up of alphanumeric characters and the underscore (`_`) character. If you try to register a shortcode with a name using any other characters, it will not work.

## Built-in shortcodes

Silverstripe CMS comes with several shortcode built-in.

### Links

Internal page links keep references to their database IDs rather than the URL, in order to make these links resilient
against moving the target page to a different location in the page tree. This is done through the `[sitetree_link]`
shortcode, which takes an `id` parameter.

```html
 <a href="[sitetree_link,id=99]">...</a>
```

Links to internal `File` database records work exactly the same, but with the `[file_link]` shortcode.

```html
 <a href="[file_link,id=99]">...</a>
```

### Images

Images inserted through the "Insert Media" form (WYSIWYG editor) need to retain a relationship with
the underlying [Image](api:SilverStripe\Assets\Image) database record. The `[image]` shortcode saves this database reference
instead of hard-linking to the filesystem path of a given image.

```html
[image id="99" alt="My text"]
```

### Media (photo, video and rich content)

Many media formats can be embedded into websites through the `<object>` tag, but some require plugins like Flash or
special markup and attributes. OEmbed is a standard to discover these formats based on a simple URL, for example a
Youtube link pasted into the "Insert Media" form of the CMS.

Some of these variations are likely to be explicitly not allowed in your TinyMCE configuration, so the embed plugin shows a placeholder instead, and the embed details such as the URL are stored with a
custom `[embed]` shortcode.

```html
[embed width=480 height=270 class=left thumbnail=https://i1.ytimg.com/vi/lmWeD-vZAMY/hqdefault.jpg?r=8767]
  https://www.youtube.com/watch?v=lmWeD-vZAMY
[/embed]
```

### Attribute and element scope

HTML with unprocessed shortcodes in it is still valid HTML. As a result, shortcodes can be in two places in HTML:

- In an attribute value, like so: `<a title="[title]">link</a>`
- In an element's text, like so: `<p>Some text [shortcode] more text</p>`

The first is called "element scope", the second "attribute scope"

You may not use shortcodes in any other location. Specifically, you can not use shortcodes to generate new attributes in an existing element or
change the name of a tag. These usages are forbidden:

```html
<[paragraph]>Some test</[paragraph]>

<a [titleattribute]>link</a>
```

You may need to escape text inside attributes `>` becomes `&gt;`, You can include HTML tags inside a shortcode tag, but
you need to be careful of nesting to ensure you don't break the output.

```html
<!-- Good -->
<div>
    [shortcode]
        <p>Caption</p>
    [/shortcode]
</div>

<!-- Bad: -->

<div>
    [shortcode]
</div>
<p>
    [/shortcode]
</p>
```

### Location

Element scoped shortcodes have a special ability to move the location they are inserted at to comply with HTML lexical
rules. Take for example this basic paragraph tag:

```html
<p><a href="#">Head [figure,src="assets/a.jpg",caption="caption"] Tail</a></p>
```

When converted naively would become:

```html
<p><a href="#">Head <figure><img src="assets/a.jpg" /><figcaption>caption</figcaption></figure> Tail</a></p>
```

However this is not valid HTML - P elements can not contain other block level elements.

To fix this you can specify a "location" attribute on a shortcode. When the location attribute is "left" or "right"
the inserted content will be moved to immediately before the block tag.

```html
<p><a href="#">Head [figure,location="left",src="assets/a.jpg",caption="caption"] Tail</a></p>
```

The result is this:

```html
<figure><img src="assets/a.jpg" /><figcaption>caption</figcaption></figure><p><a href="#">Head  Tail</a></p>
```

### Parameter values

Here is a summary of the callback parameter values based on some example shortcodes.

```php
namespace App\ShortCode;

use SilverStripe\View\Parsers\ShortcodeParser;

class MyShortCodeProvider
{
    public static function myCustomShortCode(
        array $arguments,
        ?string $content,
        ShortcodeParser $parser,
        string $tagName
    ) {
        // ...
    }
}
```

```text
[my_shortcode]
$attributes      => [];
$content         => null;
$parser          => ShortcodeParser instance,
$tagName         => 'my_shortcode'
```

```text
[my_shortcode,attribute="foo",other="bar"]
$attributes      => ['attribute' => 'foo', 'other' => 'bar']
$enclosedContent => null
$parser          => ShortcodeParser instance
$tagName         => 'my_shortcode'
```

```text
[my_shortcode,attribute="foo"]content[/my_shortcode]
$attributes      => ['attribute' => 'foo']
$enclosedContent => 'content'
$parser          => ShortcodeParser instance
$tagName         => 'my_shortcode'
```

## Limitations

Since the shortcode parser is based on a simple regular expression it cannot properly handle nested shortcodes. For
example the below code will not work as expected:

```html
[shortcode]
[shortcode][/shortcode]
[/shortcode]
```

The parser will raise an error if it can not find a matching opening tag for any particular closing tag

## Related documentation

- [Wordpress Implementation](https://codex.wordpress.org/Shortcode_API)
- [How to Create a Google Maps Shortcode](how_tos/create_a_google_maps_shortcode)

## API documentation

- [ShortcodeParser](api:SilverStripe\View\Parsers\ShortcodeParser)
