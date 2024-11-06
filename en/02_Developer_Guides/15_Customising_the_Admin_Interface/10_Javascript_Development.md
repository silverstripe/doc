---
title: Javascript Development
summary: Advanced documentation about writing and customizing javascript within Silverstripe CMS.
iconBrand: js
---

# JavaScript development

The following document is an advanced guide on building rich JavaScript interactions within the Silverstripe CMS and
a list of our best practices for contributing and modifying the core JavaScript framework.

## Build tools

Silverstripe's JavaScript is transpiled, meaning it goes through a transformative process that takes our original source code
and outputs JavaScript that is more efficient, smaller in overall file size, and works on a wider range of browsers.

There are many ways to solve the problem of transpiling. The toolchain we use in core Silverstripe CMS modules includes:

- [Babel](https://babeljs.io) (transpiler)
- [Webpack](https://webpack.js.org) (Module bundler)

## jQuery, jQuery UI and jQuery.Entwine: our libraries of choice

> [!WARNING]
> The following documentation regarding jQuery, jQueryUI and Entwine does not apply to React components or sections powered by React.
> If you're developing new functionality in React powered sections please refer to
> [ReactJS and Redux](./reactjs_and_redux).

We predominantly use [jQuery](https://jquery.com) as our abstraction library for DOM related programming, within the
Silverstripe CMS and certain framework aspects.

For richer interactions such as drag'n'drop, and more complicated interface elements like tabs or accordions,
Silverstripe CMS uses [jQuery UI](https://ui.jquery.com) on top of jQuery.

For any custom code developed with jQuery, you have four choices to structure it: Custom jQuery Code, a jQuery Plugin, a
jQuery UI Widget, or a `jQuery.entwine` behaviour. We'll detail below where each solution is appropriate.

### Custom jQuery code

jQuery allows you to write complex behavior in a couple of lines of JavaScript. Smaller features which aren't likely to
be reused can be custom code without further encapsulation. For example, a button rollover effect doesn't require a full
plugin. See "[How jQuery Works](https://docs.jquery.com/How_jQuery_Works)" for a good introduction.

You should write all your custom jQuery code in a closure.

```js
(function ($) {
  $(document).ready(() => {
    // your code here.
  });
}(jQuery));
```

#### jQuery plugins

A jQuery Plugin is essentially a method call which can act on a collection of DOM elements. It is contained within the
`jQuery.fn` namespace, and attaches itself automatically to all jQuery collections. You can read more about these, including
how to create your own plugins, in the official [jQuery Plugins](https://learn.jquery.com/plugins/) documentation.

#### jQuery UI widgets

UI Widgets are jQuery Plugins with a bit more structure, targeted towards interactive elements. They require jQuery and
the core libraries in jQuery UI, so are generally more heavyweight if jQuery UI isn't already used elsewhere.

Main advantages over simpler jQuery plugins are:

- Exposing public methods on DOM elements (incl. pseudo-private methods)
- Exposing configuration and getters/setters on DOM elements
- Constructor/Destructor hooks
- Focus management and mouse interaction

See the [official API documentation](https://api.jqueryui.com/) and read up about the
[jQuery UI Widget Factory](https://learn.jquery.com/jquery-ui/widget-factory/) to get started.

### jQuery.Entwine

jQuery.entwine is a third-party plugin, though it is effectively only used for Silverstripe CMS development. It is useful in Silverstripe CMS development because a lot of the UI is powered by AJAX requests, which manipulate the DOM instead of loading new pages from scratch. This makes it difficult to add native event handlers directly to the relevant elements in the DOM since at the time your script is executed, there's no guarantee the appropriate element will be in the DOM.

Use jQuery.entwine when your code is likely to be customised by others, or when you need to attach events or functionality to specific DOM elements in the CMS.

Example: Highlighter

```js
(function ($) {
  $(':button').entwine({
    Foreground: 'red',
    Background: 'yellow',
    highlight() {
      this.css('background', this.getBackground());
      this.css('color', this.getForeground());
    }
  });
}(jQuery));
```

Usage:

```js
(function ($) {
  // call with default options
  $(':button').entwine().highlight();

  // set options for existing and new instances
  $(':button').entwine().setBackground('green');

  // get property
  $(':button').entwine().getBackground();
}(jQuery));
```

This is a deliberately simple example, the strength of jQuery.entwine over simple jQuery plugins lies in its public
properties, namespacing, as well as its inheritance based on CSS selectors. Go to our [jQuery Entwine
documentation](jquery_entwine) for more complete examples.

## Architecture and best practices

### Keep things simple

Resist the temptation to build "cathedrals" of complex interrelated components. In general, you can get a lot done in
jQuery with a few lines of code. Your jQuery code will normally end up as a series of event handlers applied with `jQuery.on()` or jQuery.entwine, rather than a complex object graph.

### Don't claim global properties

Global properties are evil. They are accessible by other scripts and might be overwritten or misused. A popular case is the `$` shortcut in different libraries: in PrototypeJS it stands for `document.getElementByID()`, in jQuery for `jQuery()`.

```js
// you can't rely on '$' being defined outside of the closure
(function ($) {
  let myPrivateVar; // only available inside the closure
  // inside here you can use the 'jQuery' object as '$'
}(jQuery));
```

You can run [`jQuery.noConflict()`](https://docs.jquery.com/Core/jQuery.noConflict) to avoid namespace clashes.
NoConflict mode is enabled by default in the Silverstripe CMS JavaScript.

### Initialize at document.Ready

You have to ensure that DOM elements you want to act on are loaded before using them. jQuery provides a wrapper around
the `window.onload` and `document.ready` events.

> [!NOTE]
> This doesn't apply to jQuery entwine declarations, which will apply to elements matching your selectors as they get added to the DOM, even if that happens before or after your code is executed. See [the entwine documentation](jquery_entwine) for more details about this.

```js
(function ($) {
  // DOM elements might not be available here
  $(document).ready(() => {
    // The DOM is fully loaded here
  });
}(jQuery));
```

See [the jQuerydocs on `$( document ).ready()`](https://learn.jquery.com/using-jquery-core/document-ready/).

### Bind events "live"

jQuery supports automatically reapplying event handlers when new DOM elements get inserted, mostly through Ajax calls.
This "binding" saves you from reapplying this step manually.

Caution: Only applies to certain events, see the [jQuery.on() documentation](https://api.jquery.com/on/#direct-and-delegated-events).

Example: Add a 'loading' classname to all pressed buttons

```js
// manual binding, only applies to existing elements
$('input[[type=submit]]').on('click', function () {
  $(this).addClass('loading');
});

// binding, applies to any inserted elements as well
$('.cms-container').on('click', 'input[[type=submit]]', function () {
  $(this).addClass('loading');
});
```

> [!TIP]
> You can do this using entwine as well, which has the added benefit of not requiring your original selector to match a DOM element initially (e.g. for the above example if there are no `.cms-container` elements, or those elements are removed and re-added to the DOM, your native binding won't work but an entwine one will).

### Assume element collections

jQuery is based around collections of DOM elements, the library functions typically handle multiple elements (where it
makes sense). Encapsulate your code by nesting your jQuery commands inside a `jQuery().each()` call.

```js
$('div.MyGridField').each(function () {
  // This is the over code for the tr elements inside a GridField.
  $(this).find('tr').hover(
    // ...
  );
});
```

### Use plain HTML and `jQuery.data()` to store data

The DOM can make JavaScript configuration and state-keeping a lot easier, without having to resort to JavaScript
properties and complex object graphs.

Example: Simple form change tracking to prevent submission of unchanged data

Through CSS properties

```js
$('form :input').bind('change', function (event) {
  $(this.form).addClass('isChanged');
});

$('form').bind('submit', function (event) {
  if ($(this).hasClass('isChanged')) {
    event.preventDefault();
  }
});
```

Through `jQuery.data()`

```js
$('form :input').bind('change', function (event) {
  $(this.form).data('isChanged', true);
});

$('form').bind('submit', function (event) {
  if ($(this).data('isChanged')) {
    event.preventDefault();
  }
});
```

### Return HTML/JSON and `HTTPResponse` class for AJAX responses

Ajax responses will sometimes need to update existing DOM elements, for example refresh a set of search results.
Returning plain HTML is generally a good default behaviour, as it allows you to keep template rendering in one place (in
Silverstripe CMS PHP code), and is easy to deal with in JavaScript.

If you need to process or inspect returned data, consider extracting it from the loaded HTML instead (through id/class
attributes, or the jQuery.metadata plugin). For returning status messages, please use the HTTP status-codes.

Only return evaluated JavaScript snippets if unavoidable. Most of the time you can just pass data around, and let the
clientside react to changes appropriately without telling it directly through JavaScript in AJAX responses. Don't use
the [Form](api:SilverStripe\Forms\Form) Silverstripe CMS class, which is built solely around
this inflexible concept.

Example: Autocomplete input field loading page matches through AJAX

Template:

```ss
<ul>
<% loop $Results %>
  <li id="Result-$ID">$Title</li>
<% end_loop %>
</ul>
```

PHP:

```php
namespace App\Control;

use Page;
use SilverStripe\Control\HTTPResponse;
use SilverStripe\Model\ModelData;

class MyController
{
    private static $url_segment = 'my_controller';
    // ...

    public function autocomplete($request)
    {
        $results = Page::get()->filter('Title', $request->getVar('title'));
        if (!$results) {
            return HTTPResponse::create('Not found', 404);
        }

        // Use HTTPResponse to pass custom status messages
        $this->getResponse()
        ->setStatusCode(200)
        ->addHeader('X-Status', 'Found ' . $results->Count() . ' elements');

        // render all results with a custom template
        $vd = ModelData::create();
        return $vd->customise([
            'Results' => $results,
        ])->renderWith('AutoComplete');
    }
}
```

HTML

```ss
<form action"#">
  <div class="autocomplete {url:'my_controller/autocomplete'}">
    <input type="text" name="title" />
    <div class="results" style="display: none;">
  </div>
  <input type="submit" value="action_autocomplete" />
</form>
```

JavaScript:

```js
$('.autocomplete input').on('change', function () {
  const resultsEl = $(this).siblings('.results');
  resultsEl.load(
    // get form action, using the jQuery.metadata plugin
    $(this).parent().metadata().url,
    // submit all form values
    $(this.form).serialize(),
    // callback after data is loaded
    (data, status) => {
      resultsEl.show();
      // get all record IDs from the new HTML
      const ids = jQuery('.results').find('li').map(
        () => $(this).attr('id').replace(/Record\-/, '')
      );
    }
  );
});
```

Although they are the minority of cases, there are times when a simple HTML fragment isn't enough.  For example, if you
have server side code that needs to trigger the update of a couple of elements in the CMS left-hand tree, it would be
inefficient to send back the HTML of entire tree. Silverstripe CMS can serialize to and from JSON (see the [Convert](api:SilverStripe\Core\Convert) class), and jQuery deals very well with it through
[jQuery.getJSON()](https://docs.jquery.com/Ajax/jQuery.getJSON#urldatacallback), as long as the HTTP content-type is
properly set.

### Use events and observation to link components together

The philosophy behind this JavaScript guide is **component driven development**: your JavaScript should be structured as
a set of components that communicate. Event handlers are a great way of getting components to community, as long as
two-way communication isn't required.  Set up a number of custom event names that your component will trigger.  List
them in the component documentation comment.

jQuery can bind to DOM events and trigger them through custom code. It can also
[trigger custom events](https://docs.jquery.com/Events/trigger), and supports [namespaced
events](https://docs.jquery.com/Namespaced_Events).

Example: Trigger custom 'validationfailed' event on form submission for each empty element

```js
$('form').on('submit', function (e) {
  // $(this) refers to form
  $(this).find(':input').each(function () {
    // $(this) in here refers to input field
    if (!$(this).val()) {
      $(this).trigger('validationfailed');
    }
  });

  return false;
});

// listen to custom event on each <input> field
$('form :input').on('validationfailed', function (e) {
  // $(this) refers to input field
  const fieldName = $(this).attr('name');
});
```

Don't use event handlers in the following situations:

- If two-way communication is required, for example, calling an method in another component, which returns data that
you then use.  Event handlers can't have return values.
- If specific execution order is required.  Event handlers are executed in parallel, which makes it difficult to know
the exact order in which code in different threads will execute.  If the execution order is likely to cause problems, it
is better to use a code structure that is executed sequentially. An example might be two events modifying the same piece
of the DOM.

### Use callbacks to allow customizations

Callbacks are similar to events in that other components can ask your component to execute a piece of code.  The
advantage is that they lack the two problems listed in bullets just above. The disadvantage of callbacks is that you
need to define an custom API for configuring the callbacks; whereas, event observation is a jQuery provided API that
leaves components very loosely coupled.

### Use jQuery.Entwine to define APIs as necessary

By default, most of your JavaScript methods will be hidden in closures like a jQuery plugin, and are not accessible from
the outside. As a best practice, each jQuery plugin should only expose one method to initialize and configure it. If you
need more public methods, consider using either a jQuery UI Widget, or define your behaviour as jQuery.entwine rules
(see above).

### Write documentation

Documentation in JavaScript usually resembles the JavaDoc standard, although there is no agreed standard. Due to the
flexibility of the language it can be hard to generate automated documentation, particularly with the predominant usage
of closure constructs in jQuery and jQuery.entwine.

To generate documentation for Silverstripe CMS code, use [JSDoc toolkit](https://code.google.com/p/jsdoc-toolkit/) (see
[reference of supported tags](https://code.google.com/p/jsdoc-toolkit/wiki/TagReference)). For more class-oriented
JavaScript, take a look at the [jsdoc cookbook](https://code.google.com/p/jsdoc-toolkit/wiki/CookBook). The `@lends`
and `@borrows` properties are particularly useful for documenting jQuery-style code.

JSDoc-toolkit is a command line utility, see [usage](https://code.google.com/p/jsdoc-toolkit/wiki/CommandlineOptions).

Example: jQuery.entwine

```js
/**

 * Available Custom Events:
 * <ul>
 * <li>ajaxsubmit</li>
 * <li>validate</li>
 * <li>reloadeditform</li>
 * </ul>
 *
 * @class Main LeftAndMain interface with some control panel and an edit form.
 * @name ss.LeftAndMain
 */
$('.LeftAndMain').entwine({
  /**
   * Reference to some property
   * @type Number
   */
  MyProperty: 123,

  /**
   * Renders the provided data into an unordered list.
   *
   * @param {Object} data
   * @param {String} status
   * @return {String} HTML unordered list
   */
  publicMethod(data, status) {
    return '<ul> ... </ul>';
  },

  /**
   * Won't show in documentation, but still worth documenting.
   *
   * @return {String} Something else.
   */
  _privateMethod() {
    // ...
  }
});
```

## Related

- [Unobtrusive JavaScript](https://www.onlinetools.org/articles/unobtrusivejavascript/chapter1.html)
- [Quirksmode: In-depth JavaScript Resources](https://www.quirksmode.org/resources.html)
