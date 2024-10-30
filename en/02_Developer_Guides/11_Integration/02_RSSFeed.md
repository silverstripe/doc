---
title: RSS Feed
summary: Output records from your database as an RSS Feed.
icon: rss
---

# RSS feed

Generating RSS / Atom-feeds is a matter of rendering a [SS_List](api:SilverStripe\Model\List\SS_List) instance through the [RSSFeed](api:SilverStripe\Control\RSS\RSSFeed) class.

The [RSSFeed](api:SilverStripe\Control\RSS\RSSFeed) class doesn't limit you to generating article based feeds, it is just as easy to create a feed of
your current staff members, comments or any other custom [DataObject](api:SilverStripe\ORM\DataObject) subclasses you have defined. The only
logical limitation here is that every item in the RSS-feed should be accessible through a URL on your website, so it's
advisable to just create feeds from subclasses of [SiteTree](api:SilverStripe\CMS\Model\SiteTree).

> [!WARNING]
> If you wish to generate an RSS feed that contains a [DataObject](api:SilverStripe\ORM\DataObject), ensure you define a `AbsoluteLink` method on
> the object.

## Usage

Including an RSS feed has two steps. First, a `Controller` action which responses with the `XML` and secondly, the other
web pages need to link to the URL to notify users that the RSS feed is available and where it is.

An outline of step one looks like:

```php
use SilverStripe\Control\RSS\RSSFeed;

$feed = RSSFeed::create(
    $list,
    $link,
    $title,
    $description,
    $titleField,
    $descriptionField,
    $authorField,
    $lastModifiedTime,
    $etag
);

$feed->outputToBrowser();
```

To achieve step two include the following code where ever you want to include the `<link>` tag to the RSS Feed. This
will normally go in your `Controllers` `init` method.

```php
RSSFeed::linkToFeed($link, $title);
```

## Examples

### Showing the 10 most recently updated pages

You can use [RSSFeed](api:SilverStripe\Control\RSS\RSSFeed) to easily create a feed showing your latest Page updates. The following example adds a page
`/home/rss/` which displays an XML file the latest updated pages.

```php
// app/src/PageType/HomePageController.php
namespace App\PageType;

use PageController;
use SilverStripe\Control\RSS\RSSFeed;

class HomePageController extends PageController
{
    private static $allowed_actions = [
        'rss',
    ];

    public function init()
    {
        parent::init();

        RSSFeed::linkToFeed($this->Link('rss'), '10 Most Recently Updated Pages');
    }

    public function rss()
    {
        $rss = RSSFeed::create(
            $this->getLatestUpdates(),
            $this->Link(),
            '10 Most Recently Updated Pages',
            'Shows a list of the 10 most recently updated pages.'
        );

        return $rss->outputToBrowser();
    }

    public function getLatestUpdates()
    {
        return HomePage::get()->sort('LastEdited', 'DESC')->limit(10);
    }
}
```

### Rendering `DataObject` records in a RSS feed

DataObjects can be rendered in the feed as well, however, since they aren't explicitly [SiteTree](api:SilverStripe\CMS\Model\SiteTree) subclasses we
need to include a function `AbsoluteLink` to allow the RSS feed to link through to the item.

> [!NOTE]
> If the items are all displayed on a single page you may simply hard code the link to point to a particular page.

Take an example, we want to create an RSS feed of all the `Players` objects in our site. We make sure the `AbsoluteLink`
method is defined and returns a string to the full website URL.

```php
namespace App\Model;

use SilverStripe\Control\Controller;
use SilverStripe\Control\Director;
use SilverStripe\ORM\DataObject;

class Player extends DataObject
{
    public function AbsoluteLink()
    {
        // assumes players can be accessed at www.example.com/players/2
        return Controller::join_links(
            Director::absoluteBaseUrl(),
            'players',
            $this->ID
        );
    }
}
```

Then in our controller, we add a new action which returns a the XML list of `Player` records.

```php
namespace App\PageType;

use App\Model\Player;
use PageController;
use SilverStripe\Control\RSS\RSSFeed;

class HomePageController extends PageController
{
    private static $allowed_actions = [
        'players',
    ];

    public function init()
    {
        parent::init();

        RSSFeed::linkToFeed($this->Link('players'), 'Players');
    }

    public function players()
    {
        $rss = RSSFeed::create(
            Player::get(),
            $this->Link('players'),
            'Players'
        );

        return $rss->outputToBrowser();
    }
}
```

### Customizing the RSS feed template

The default template used for XML view is `vendor/silverstripe/framework/templates/RSSFeed`. This template displays titles and links to
the object. To customise the XML produced use `setTemplate`.

Say from that last example we want to include the Players Team in the XML feed we might create the following XML file.

```ss
<%-- app/templates/PlayersRss.ss --%>
<?xml version="1.0"?>
<rss version="2.0" xmlns:dc="http://purl.org/dc/elements/1.1/" xmlns:atom="http://www.w3.org/2005/Atom">
    <channel>
        <title>$Title</title>
        <link>$Link</link>
        <atom:link href="$Link" rel="self" type="application/rss+xml" />
        <description>$Description.XML</description>

        <% loop $Entries %>
        <item>
            <title>$Title.XML</title>
            <team>$Team.Title</team>
        </item>
        <% end_loop %>
    </channel>
</rss>
```

`setTemplate` can then be used to tell RSSFeed to use that new template.

```php
// app/src/PageType/HomePage.php
namespace App\PageType;

use Page;
use SilverStripe\Control\RSS\RSSFeed;

class HomePage extends Page
{
    public function players()
    {
        $rss = RSSFeed::create(
            Player::get(),
            $this->Link('players'),
            'Players'
        );

        $rss->setTemplate('PlayersRss');

        return $rss->outputToBrowser();
    }
}
```

> [!WARNING]
> As we've added a new template (`PlayersRss`) make sure you clear your Silverstripe CMS cache.

## API documentation

- [RSSFeed](api:SilverStripe\Control\RSS\RSSFeed)
