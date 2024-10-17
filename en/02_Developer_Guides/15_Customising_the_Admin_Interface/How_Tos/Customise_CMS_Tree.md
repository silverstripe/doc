---
title: Customise the CMS tree
summary: Learn how to add custom UI elements to the CMS page navigation
icon: sitemap
---

# How to customise the CMS tree

## Overview

The CMS tree for viewing hierarchical structures (mostly pages) is powered
by the [jstree](https://jstree.com) library. It is configured through
`client/src/legacy/LeftAndMain.Tree.js` in the `silverstripe/admin` module, as well as some
HTML5 metadata generated on its container (see the `data-hints` attribute).

The tree is rendered through [LeftAndMain::getTreeFor()](api:SilverStripe\Admin\LeftAndMain::getTreeFor()),
which recursively collects all nodes based on various filtering criteria.
The node strictly just has to implement the [Hierarchy](api:SilverStripe\ORM\Hierarchy\Hierarchy) extension,
but in the CMS usually is a [SiteTree](api:SilverStripe\CMS\Model\SiteTree) object.

## Add status flags to tree nodes

A tree node in CMS could be rendered with lot of extra information but a node title, such as a
link that wraps around the node title, a node's id which is given as id attribute of the node
&lt;li&gt; tag, a extra checkbox beside the tree title, tree icon class or extra &lt;span&gt;
tags showing the node status, etc. Silverstripe CMS tree node will be typically rendered into html
code like this:

```ss
...
<ul>
    ...
    <li id="record-15" class="class-Page closed jstree-leaf jstree-unchecked" data-id="15">
    <ins class="jstree-icon">&nbsp;</ins>
        <a class="" title="Page type: Page" href="$AdminURL('page/edit/show/15')">
            <ins class="jstree-checkbox">&nbsp;</ins>
            <ins class="jstree-icon">&nbsp;</ins>
            <span class="text">
                <span class="jstree-recordicon"></span>
                <span class="item" title="Deleted">New Page</span>
                <span class="badge deletedonlive">Deleted</span>
            </span>
        </a>
    </li>
    ...
</ul>
...
```

By applying the proper style sheet, the snippet html above could produce the look of:
![Page Node Screenshot](../../../_images/tree_node.png "Page Node")

In the above screenshot "DELETED" is the status flag.

To learn how to add, remove, and update status flags see [status flags](/developer_guides/customising_the_admin_interface/status_flags/).

## Customising page icons

The page tree in the CMS is a central element to manage page hierarchies, hence its display of pages can be customised as well. You can specify a custom page icon to make it easier for CMS authors to identify pages of this type, when navigating the tree or adding a new page:

```php
namespace App\PageType;

use Page;

class HomePage extends Page
{
    private static $cms_icon_class = 'font-icon-p-home';

    // ...
}
```

The CMS uses an icon set from [Fontastic](https://fontastic.me/). New icons may be [requested](https://github.com/silverstripe/silverstripe-admin/issues/new) and added to the [core icon set](https://silverstripe.github.io/silverstripe-pattern-lib/?selectedKind=Admin%2FIcons&selectedStory=Icon%20reference&full=0&addons=1&stories=1&panelRight=0&addonPanel=storybook%2Factions%2Factions-panel). The benefit of having icons added to the core set is that you can use icons more consistently across different modules allowing every module to use a different icon with the same style.

You can also add your own icon by specifying an image path to override the Fontastic icon set:

```php
namespace App\PageType;

use Page;

class HomePage extends Page
{
    private static $icon = 'app/images/homepage-icon.svg';

    // ...
}
```
