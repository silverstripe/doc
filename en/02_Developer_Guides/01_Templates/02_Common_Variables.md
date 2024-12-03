---
title: Common Variables
summary: Some of the common variables and methods your templates can use, including Menu, SiteConfig, and more.
---

# Common variables

The page below describes a few of common variables and methods you'll see in a Silverstripe CMS template. This is not an
exhaustive list. From your template you can call any method, database field, or relation on the object which is
currently in scope as well as its subclasses or extensions.

Knowing what methods you can call can be tricky, but the first step is to understand the scope you're in. Scope is
explained in more detail on the [syntax](syntax#scope) page. Many of the methods listed below can be called from any
scope, and you can specify additional static methods to be available globally in templates by implementing the
[TemplateGlobalProvider](api:SilverStripe\View\TemplateGlobalProvider) interface.

> [!WARNING]
> Want a quick way of knowing what scope you're in? Try putting `$ClassName` in your template. You should see a string
> such as `Page` of the object that's in scope. The methods you can call on that object then are any functions, database
> properties or relations on the `Page` class, `PageController` class as well as anything from their parent classes and
> any extensions applied to them.

Outputting these variables is only the start, if you want to format or manipulate them before adding them to the template
have a read of the [Formatting, Modifying and Casting Variables](casting) documentation.

> [!CAUTION]
> Some of the following only apply when you have the `silverstripe/cms` module installed. If you're using `silverstripe/framework` alone, this
> functionality may not be included.

## Base tag

```ss
<head>
    <% base_tag %>
    ...
</head>
```

The `<% base_tag %>` placeholder is replaced with the HTML `<base>` element. Relative links within a document (such as
`<img src="someimage.jpg" alt="">`) will become relative to the URI specified in the base tag. This ensures the
browser knows where to locate your site’s images and CSS files.

It renders in the template as `<base href="https://www.example.com/">`

> [!CAUTION]
> A `<% base_tag %>` is nearly always required or assumed by Silverstripe CMS to exist.

## `CurrentMember`

Returns the currently logged in [Member](api:SilverStripe\Security\Member) instance, if there is one logged in.

```ss
<% if $CurrentMember %>
  Welcome back, $CurrentMember.FirstName
<% end_if %>
```

## Title and menu title

```ss
$Title
$MenuTitle
```

Most objects within Silverstripe CMS will respond to `$Title` (i.e. they should have a `Title` database field or at least a
`getTitle()` method).

The CMS module in particular provides two fields to label a page: `Title` and `MenuTitle`. `Title` is the title
displayed on the web page, while `MenuTitle` can be a shorter version suitable for size-constrained menus.

> [!WARNING]
> If `MenuTitle` is left blank by the CMS author, it'll just default to the value in `Title`.

## Page content

```ss
$Content
```

It returns the database content of the `Content` field. For subclasses of [`SiteTree`](api:SilverStripe\CMS\Model\SiteTree), this is the value of the WYSIWYG editor
but it is also the standard for any object that has a body of content to output.

> [!NOTE]
> Please note that this database content can be "versioned", meaning that draft content edited in the CMS can be different
> from published content shown to your website visitors. In templates, you don't need to worry about this distinction.
>
> The `$Content` variable contains the published content by default, and only preview draft content if explicitly
> requested (e.g. by the "preview" feature in the CMS) (see the [versioning documentation](/../model/versioning) for
> more details).

### `SiteConfig`: global settings

> [!WARNING]
> `SiteConfig` comes from an optional module that is bundled with the CMS. If you wish to include `SiteConfig` in your framework only
> web pages, you'll need to install `silverstripe/siteconfig` via composer.

```ss
$SiteConfig.Title
```

The [`SiteConfig`](api:SilverStripe\SiteConfig\SiteConfig) object allows content authors to modify global data in the CMS, rather
than PHP code. By default, this includes a Website title and a Tagline.

`SiteConfig` can be extended to hold other data, for example a logo image which can be uploaded through the CMS or
global content such as your footer content.

See the [`SiteConfig` documentation](../configuration/siteconfig) for more information.

## Meta tags

The `$MetaTags` placeholder in a template returns a segment of HTML appropriate for putting into the `<head>` tag. It
will set up title, keywords and description meta-tags, based on the CMS content and is editable in the 'Meta-data' tab
on a per-page basis.

> [!WARNING]
> If you don’t want to include the title tag use `$MetaTags(false)`.

By default `$MetaTags` renders (assuming 6.0.0 is the current version of `silverstripe/framework`):

```html
<title>Title of the Page</title>
<meta name="generator" content="Silverstripe CMS 6.0">
<meta http-equiv="Content-type" content="text/html; charset=utf-8" />
```

`$MetaTags(false)` will render

```html
<meta name="generator" content="Silverstripe CMS 6.0">
<meta http-equiv="Content-type" content="text/html; charset=utf-8" />
```

If using `$MetaTags(false)` we can provide a custom `<title> tag`.

```ss
$MetaTags(false)
<title>$Title - Bob's Fantasy Football</title>
```

### Disabling the meta generator tag

The meta generator tag includes the current version of `silverstripe/framework`. This version number
provides aggregate installation numbers to the product team who maintain Silverstripe CMS which is
used to make informed product decisions.

If you dislike this behaviour, the entire meta generator tag can be disabled via:

```yml
SilverStripe\CMS\Model\SiteTree:
  meta_generator: ''
```

The version portion of the metagenerator tag can be disabled via:

```yml
SilverStripe\CMS\Model\SiteTree:
  show_meta_generator_version: false
```

### Modifying meta tags

You can override the `MetaComponents()` method on your `SiteTree` sub-classes or make use of the `updateMetaComponents` extension point to manipulate the underlying data that is rendered by `$MetaTags`. Example (for `Page` class):

```php
namespace App\PageType;

use Page;

class MyPage extends Page
{
    // ...

    public function MetaComponents()
    {
        $tags = parent::MetaComponents();
        // Override the content of the Title tag (needs to be html)
        if ($this->MetaTitle) {
            $tags['title']['content'] = $this->obj('MetaTitle')->forTemplate();
        }
        // Provide a default Meta Description
        if (!$tags['description']['attributes']['content']) {
            // provide raw text as attributes will be escaped later
            $tags['description']['attributes']['content']
                = $this->dbObject('Content')->LimitCharactersToClosestWord(300);
        }
        return $tags;
    }
}
```

## Links

```ss
<a href="$Link">..</a>
```

All objects that could be accessible via a public HTTP request in Silverstripe CMS should define a `Link()` method and an `AbsoluteLink()` method. `Link()`
returns the relative URL for the object and `AbsoluteLink()` outputs your full website address along with the relative
link.

```ss
<%-- prints /about-us/offices/ --%>
$Link

<%-- prints https://www.example.com/about-us/offices/ --%>
$AbsoluteLink
```

### Linking modes

```ss
$isSection
$isCurrent
```

When looping over a list of `SiteTree` instances through a `<% loop $Menu %>` or `<% loop $Children %>`, `$isSection` and `$isCurrent`
will return true or false based on page being looped over relative to the currently viewed page.

For instance, to only show the menu item linked if it's the current one:

```ss
<% if $isCurrent %>
    $Title
<% else %>
    <a href="$Link">$Title</a>
<% end_if %>
```

An example for checking for `current` or `section` is as follows:

```ss
<a class="<% if $isCurrent %>current<% else_if $isSection %>section<% end_if %>" href="$Link">$MenuTitle</a>
```

#### Additional utility method

- `$InSection('<page-url-segment>')`: This if block will pass if we're currently on that page or one of its children.

```ss
<% if $InSection('about-us') %>
    <p>You are viewing the about us section</p>
<% end_if %>
```

### `URLSegment`

This returns the part of the URL segment of the page you're currently on. For example on the `/about-us/offices/` web page the
`URLSegment` will be `offices`. `URLSegment` cannot easily be used to generate a link since it does not output the full path.
It can be used within templates to generate anchors or other CSS classes.

```ss
<%-- prints <div id="section-offices"> --%>
<div id="section-$URLSegment">
    ...
</div>
```

## `ClassName`

Returns the class of the current object in scope ([see "scope" in the syntax section](syntax#scope)) such as `Page` or `App\PageType\HomePage`.

Note that this is backed by [`DBClassName`](SilverStripe\ORM\FieldType\DBClassName), which means you can use the methods in that class from your template
(e.g. `$ClassName.ShortName` prints `HomePage` instead of `App\PageType\HomePage`).

The `$ClassName` can be handy for a number of uses. A common use case is to add to your `<body>` tag to influence CSS styles and JavaScript
behavior based on the page type used:

```ss
<%-- prints <body class="HomePage">, or <body class="BlogPage"> --%>
<body class="$ClassName.ShortName">
```

## `Children` loops

```ss
<% loop $Children %>

<% end_loop %>
```

Will loop over all Children records of the current object context. Children are pages that sit under the current page in
the `CMS` or a custom list of data. This originates in the `Versioned` extension's `getChildren` method.

See [Looping Over Lists](syntax#looping-over-lists) for more information about looping in general.

> [!CAUTION]
> For doing your website navigation most likely you'll want to use `$Menu` since its independent of the page
> context.

### `ChildrenOf`

```ss
<% loop $ChildrenOf('<my-page-url-segment>') %>

<% end_loop %>
```

Will create a list of the children of the given page, as identified by its `URLSegment` value. This can come in handy
because it's not dependent on the context of the current page. For example, it would allow you to list all staff member
pages underneath a "staff" holder on any page, regardless if its on the top level or elsewhere.

> [!WARNING]
> Because variables can't be passed into method calls from templates (see [Syntax > Variables](syntax#variables)), this requires you to hardcode some value into your template - which means you must ensure you have a page added in the CMS with that URL segment
>
> A more robust way to implement this would be to add a helper method in your page controller which dynamically gets the appropriate page (if one exists).

### `AllChildren`

Content authors have the ability to hide pages from menus by un-selecting the `ShowInMenus` checkbox within the CMS.
This option will be honored by `<% loop $Children %>` and `<% loop $Menu %>` however if you want to ignore the user
preference, `AllChildren` does not filter by `ShowInMenus`.

```ss
<% loop $AllChildren %>
    ...
<% end_loop %>
```

### `Menu` loops

```ss
<% loop $Menu(1) %>
    ...
<% end_loop %>
```

`$Menu(1)` returns the top-level menu of the website. You can also create a sub-menu using `$Menu(2)`, and so forth.

> [!WARNING]
> Pages with the `ShowInMenus` property set to `false` will be filtered out.

## Access to a specific page

```ss
<% with $Page('my-page') %>
    $Title
<% end_with %>
```

Page will return a single page from site, looking it up by its `URLSegment` field.

## Access to parent and level pages

### `Level`

```ss
<% with $Level(1) %>
    $Title
<% end_with %>
```

Will return a page in the current path, at the level specified by the numbers. It is based on the current page context,
looking back through its parent pages. `$Level(1)` being the top most level.

For example, imagine you're on the "bob marley" page, which is three levels in: "about us > staff > bob marley".

- `$Level(1).Title` would return "about us"
- `$Level(2).Title` would return "staff"
- `$Level(3).Title` would return "bob marley"

### `Parent`

```ss
<%-- given we're on 'Bob Marley' in "about us > staff > bob marley" --%>

<%-- prints 'staff' --%>
$Parent.Title

<%-- prints 'about us' --%>
$Parent.Parent.Title
```

## Navigating scope

See [scope](syntax#scope) in the syntax documentation.

## `Breadcrumbs`

Breadcrumbs are the path of pages which need to be taken to reach the current page, and can be a great navigation aid
for website users.

While you can achieve breadcrumbs through the `$Level(<level>)` control manually, there's a nicer shortcut: The
`$Breadcrumbs` variable.

```ss
$Breadcrumbs
```

There are a number of arguments that can be passed in - see [SiteTree::Breadcrumbs()](api:SilverStripe\CMS\Model\SiteTree::Breadcrumbs()) for usage.

By default, it uses the template defined in `templates/BreadcrumbsTemplate.ss`
of the `silverstripe/cms` module.

> [!NOTE]
> To customise the markup that `$Breadcrumbs` generates, copy `templates/BreadcrumbsTemplate.ss`
> from the `silverstripe/cms` module to your theme (e.g.: `themes/your-theme/templates/BreadcrumbsTemplate.ss`).
> Modify the newly copied template and flush your Silverstripe CMS cache.

## `SilverStripeNavigator`

The [SilverStripeNavigator](api:SilverStripe\Admin\Navigator\SilverStripeNavigator) can be used on the front end for any page using a [ContentController](api:SilverStripe\CMS\Controllers\ContentController). It provides useful functionality for content authors such as showing whether the page being viewed is in published or draft mode, giving links to swap viewing modes, and a link to the CMS edit form for that page.

> [!WARNING]
> It's recommended to only display this for logged on users who have access to the CMS.

```ss
<% if $HasPerm('CMS_ACCESS') %>$SilverStripeNavigator<% end_if %>
```

## Forms

```ss
$Form
```

A page will normally contain some content and potentially a form of some kind. For example, the log-in page has a
Silverstripe CMS log-in form. If you are on such a page (and the form is implements in a method called `Form()` or `getForm()`),
the `$Form` variable will contain the HTML content of the form.
Placing it just below `$Content` is a good default.

## Related documentation

- [Casting and Formatting Variables](casting)
- [Template Inheritance](template_inheritance)

## API documentation

- [ContentController](api:SilverStripe\CMS\Controllers\ContentController): The main controller responsible for handling pages.
- [Controller](api:SilverStripe\Control\Controller): Generic controller (not specific to pages.)
- [DataObject](api:SilverStripe\ORM\DataObject): Underlying model class for models which store their data in the database.
- [ModelData](api:SilverStripe\Model\ModelData): Underlying object class for pretty much anything which contains data.
