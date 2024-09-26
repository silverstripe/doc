---
title: Template Syntax
summary: A look at the operations, variables and language controls you can use within templates.
icon: code
---

# Template syntax

A template can contain any markup language (e.g HTML, CSV, JSON, etc). The rendering process replaces placeholders such as `$Var` with real content from your
model (see [Model and Databases](../model)) and allows you to use logic like `<% if $Var %>` in your templates.

An example of a Silverstripe CMS template is below:

```ss
<%-- app/templates/Page.ss --%>
<html>
    <head>
        <% base_tag %>
        <title>$Title</title>
        $MetaTags(false)
        <% require themedCSS("screen") %>
    </head>
    <body>
        <header>
            <h1>Bob's Chicken Shack</h1>
        </header>

        <% with $CurrentMember %>
            <p>Welcome $FirstName $Surname.</p>
        <% end_with %>

        <% if $Dishes %>
        <ul>
            <% loop $Dishes %>
                <li>$Title ($Price.Nice)</li>
            <% end_loop %>
        </ul>
        <% end_if %>

        <% include Footer %>
    </body>
</html>
```

> [!NOTE]
> Templates can be used for more than HTML output. You can use them to output your data as JSON, XML, CSV or any other
> text-based format.

## Template file location

Silverstripe CMS templates are plain text files that have an `.ss` extension and are located within the `templates` directory of
a module, theme, or your `app/` folder.

See [Template types and locations](template_inheritance/#template-types-and-locations) for more information.

## Variables

Variables are placeholders that will be replaced with data from the data model or the current
controller. Variables are prefixed with a `$` character. Variable names must start with an
alphabetic character or underscore, with subsequent characters being alphanumeric or underscore:

```ss
$Title
```

This will result in trying to fetch a `Title` property from the data being displayed. This could be a database field, a method (either named `title()` or `getTitle()`), a public property, etc.

Variables can be chained together, and include arguments.

```ss
$Foo("param")
$Foo.Bar
```

These variables will call a method / field on the object and insert the returned value as a string into the template.

Arguments don't have to be literals - you can pass template variables as arguments as well.

> [!WARNING]
> Arguments cannot be literal arrays (e.g `$myMethod(["val1", "val2"])`). The behaviour for this is currently undefined.

- `$Foo("param")` will call `$obj->Foo("param")`
- `$Foo.Bar` will call `$obj->Foo()->Bar`
- `$Foo($Bar)` will call `$obj->Bar` and then pass the result into `$obj->Foo($result)`

> [!NOTE]
> Arguments passed into methods can be almost any type (not just strings), e.g:
>
> - `$Foo(1)` will pass `1` as an int
> - `$Foo(0.5)` will pass `0.5` as a float
> - `$Foo(true)` will pass `true` as a boolean
> - `$Foo(null)` will pass `null` as a null primitive
> - `$Foo("param")`, `$Foo('param')`, and `$Foo(param)` will all pass `'param'` as a string. It is recommended that you always use quotes when passing a string for clarity and for future-proofing.

If a variable returns a string, that string will be inserted into the template. If the variable returns an object, then
the system will attempt to render the object through its `forTemplate()` method. If the `forTemplate()` method has not
been defined, the system will attempt to cast the object to a string.

> [!NOTE]
> For more details around how variables are inserted and formatted into a template see
> [Formatting, Modifying and Casting Variables](/developer_guides/templates/casting/)

Variables can come from your database fields, or custom methods you define on your objects.

```php
// app/src/Model/MyObject.php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyObject extends DataObject
{
    // ...
    public function usersIpAddress()
    {
        return $this->getRequest()->getIP();
    }
}
```

```html
<!-- app/templates/App/Model/MyObject.ss -->
<p>You are coming from $UsersIpAddress.</p>
```

> [!NOTE]
> Method names that begin with `get` will automatically be resolved when their prefix is excluded. For example, if the method `usersIpAddress()` didn't exist, the above reference to `$UsersIpAddress` would invoke a method named `getUsersIpAddress()`.
> Note that a method with the exact name you're referencing will always be used if it exists.

The variables that can be used in a template vary based on the object currently in scope (see [scope](#scope) below). Scope defines what
object the methods get called on. For the standard `Page.ss` template the scope is usually the current [`ContentController`](api:SilverStripe\CMS\Controllers\ContentController)
object. This object provides access to all the public methods on that controller, as well as the public methods, relations, and database fields for its corresponding [`SiteTree`](api:SilverStripe\CMS\Model\SiteTree) record.

```ss
<%-- app/templates/Layout/Page.ss --%>

<%-- calls `SilverStripeNavigator()` on the controller and prints markup for the `SilverStripeNavigator` for this page --%>
$SilverStripeNavigator

<%-- prints the `Content` database field from the page record --%>
$Content
```

### Case sensitivity

Just like in PHP, method names are case insensitive, but property names are case sensitive.

Using `$MyValue` in a template will successfully call a `myValue()` method, or a `getMyValue()` method even though the case doesn't match what you used in the template. But it *will not* match a `myValue` property or database field because the case doesn't match.

## Conditional logic

The simplest conditional block is to check for the presence of a value. This effectively works the same as [`isset()`](https://www.php.net/manual/en/function.isset.php) in PHP - i.e. if there is no variable available with that name, or the variable's value is `0`, `false`, or `null`, the condition will be false.

```ss
<% if $CurrentMember %>
    <p>You are logged in as $CurrentMember.FirstName $CurrentMember.Surname.</p>
<% end_if %>
```

A conditional can also use comparisons.

```ss
<% if $MyDinner == "kipper" %>
    Yummy, kipper for tea.
<% end_if %>
```

> [!WARNING]
> You can technically omit the `$` prefix for variables inside template tags, but this is a legacy behaviour and can result in unexpected behaviour. Variables should have a `$` prefix, and string literals should have quotes.

Conditionals can also provide the `else` case.

```ss
<% if $MyDinner == "kipper" %>
    Yummy, kipper for tea
<% else %>
    I wish I could have kipper :-(
<% end_if %>
```

`else_if` can be used to handle chained conditional statements.

```ss
<% if $MyDinner == "quiche" %>
    I don't like quiche
<% else_if $MyDinner == $YourDinner %>
    We both have good taste
<% else %>
    Can I have some of your chips?
<% end_if %>
```

### Negation

You can check if a variable is false with `<% if not %>`.

```ss
<% if not $DinnerInOven %>
    I'm going out for dinner tonight.
<% end_if %>
```

Note that you cannot combine this with other operators such as `==`.

For more nuanced conditions you can use the `!=` operator.

```ss
<% if $MyDinner != "quiche" %>
    Lets go out
<% end_if %>
```

### Boolean logic

Multiple checks can be done using `||`/`or`, or `&&`/`and`.

> [!NOTE]
> `or` is functionally equivalent to `||` in template conditions, and `and` is functionally equivalent to `&&`.

If *either* of the conditions is true.

```ss
<% if $MyDinner == "kipper" || $MyDinner == "salmon" %>
    yummy, fish for tea
<% end_if %>
```

If *both* of the conditions are true.

```ss
<% if $MyDinner == "quiche" && $YourDinner == "kipper" %>
    Lets swap dinners
<% end_if %>
```

### Inequalities

You can use inequalities like `<`, `<=`, `>`, `>=` to compare numbers.

```ss
<% if $Number >= 5 && $Number <= 10 %>
    Number between 5 and 10
<% end_if %>
```

## Includes

Within Silverstripe CMS templates we have the ability to include other templates using the `<% include %>` tag. The includes
will be searched for using the same filename look-up rules as a regular template. However in the case of the include tag
an additional `Includes` directory will be inserted into the resolved path just prior to the filename.

```ss
<% include SideBar %> <!-- chooses templates/Includes/Sidebar.ss -->
<% include MyNamespace/SideBar %> <!-- chooses templates/MyNamespace/Includes/Sidebar.ss -->
```

The `include` tag can be particularly helpful for nested functionality and breaking large templates up. In this example,
the include only happens if the user is logged in.

```ss
<% if $CurrentMember %>
    <% include MembersOnlyInclude %>
<% end_if %>
```

Includes can't directly access the parent scope when the include is included. However you can pass arguments to the
include.

```ss
<% with $CurrentMember %>
    <% include MemberDetails Top=$Top, Name=$Name %>
<% end_with %>
```

> [!TIP]
> Unlike when passing arguments to a function call in templates, arguments passed to a template include can be literals *or* variables.

## Looping over lists

The `<% loop %>` tag is used to iterate or loop over a collection of items such as a native PHP array, a [DataList](api:SilverStripe\ORM\DataList), or an [ArrayList](api:SilverStripe\Model\List\ArrayList)
collection.

```ss
<h1>Children of $Title</h1>
<ul>
    <% loop $Children %>
        <li>$Title</li>
    <% end_loop %>
</ul>
```

This snippet loops over the children of a page, and generates an unordered list showing the `Title` property from each
page.

> [!WARNING]
> The `$Title` inside the loop refers to the Title property on each object that is looped over, not the current page like
> the reference of `$Title` outside the loop.
>
> This demonstrates the concept of scope ([see scope below](#scope)). When inside a `<% loop %>` the scope of the template has changed to the
> object that is being looped over.

You can also use `<% loop %>` without explicitly saying what to loop over. This will result in looping over whatever is currently in scope.

This can be particularly useful when you have collections within collections.

```ss
<% loop $MyLists %>
  <% loop %>
      This is looping over each of the collections that are contained inside the \$MyLists collection.
  <% end_loop %>
<% end_loop %>
```

### Altering the list

`<% loop %>` statements often iterate over [`SS_List`](api:SilverStripe\Model\List\SS_List) instances. As the template has access to the list object,
templates can call its methods.

Sorting the list by a given field.

```ss
<ul>
    <% loop $Children.Sort('Title', 'ASC') %>
        <li>$Title</li>
    <% end_loop %>
</ul>
```

Limiting the number of items displayed.

```ss
<ul>
    <% loop $Children.Limit(10) %>
        <li>$Title</li>
    <% end_loop %>
</ul>
```

Reversing the loop.

```ss
<ul>
    <% loop $Children.Reverse %>
        <li>$Title</li>
    <% end_loop %>
</ul>
```

Filtering the loop.

```ss
<ul>
    <% loop $Children.Filter('School', 'College') %>
        <li>$Title</li>
    <% end_loop %>
</ul>
```

Methods can also be chained.

```ss
<ul>
    <% loop $Children.Filter('School', 'College').Sort('Score', 'DESC') %>
        <li>$Title</li>
    <% end_loop %>
</ul>
```

### Position indicators

Inside the loop scope, there are many variables at your disposal to determine the current position in the list and
iteration. These are provided by [`SSViewer_BasicIteratorSupport::get_template_iterator_variables()`](api:SilverStripe\View\SSViewer_BasicIteratorSupport::get_template_iterator_variables()).

- `$Even`, `$Odd`: Returns boolean based on the current position in the list (see `$Pos` below). Handy for zebra striping.
- `$EvenOdd`: Returns a string based on the current position in the list, either 'even' or 'odd'. Useful for CSS classes.
- `$IsFirst`, `$IsLast`, `$Middle`: Booleans about the position in the list. All items that are not first or last are considered to be in the middle.
- `$FirstLast`: Returns a string, "first", "last", "first last" (if both), or "" (if middle). Useful for CSS classes.
- `$MiddleString`: Returns a string, "middle" if the item is in the middle, or "" otherwise.
- `$Pos`: The current position in the list (integer).
   Will start at 1, but can take a starting index as a parameter.
- `$FromEnd`: The position of the item from the end (integer).
   Last item defaults to 1, but can be passed as a parameter.
- `$TotalItems`: Number of items in the list (integer).

```ss
<ul>
    <% loop $Children.Reverse %>
        <% if $IsFirst %>
            <li>My Favourite</li>
        <% end_if %>

        <li class="$EvenOdd">Child $Pos of $TotalItems - $Title</li>
    <% end_loop %>
</ul>
```

> [!NOTE]
> A common task is to paginate your lists. See the [Pagination](how_tos/pagination) how to for a tutorial on adding
> pagination.

### `Modulus` and `MultipleOf`

`$Modulus` and `$MultipleOf` can help to build column and grid layouts.

`$Modulus` returns the modulus of the numerical position of the item in the data set. You must pass in the number to perform modulus operations to and an optional offset to start from. It returns an integer.

> [!TIP]
> `$Modulus` is useful for floated grid CSS layouts. If you want 3 rows across, put `column-$Modulus(3)` as a class and add a
> `clear: both` to `.column-1`.

```ss
<% loop $Children %>
    <%-- results in divs with `column-1` up to `column-4`, then repeating from `column-1` again --%>
    <div class="column-{$Modulus(4)}">
        ...
    </div>
<% end_loop %>
```

`$MultipleOf` returns true or false depending on if the pos of the iterator is a multiple of a specific number.
So `<% if $MultipleOf(3) %>` would return true on indexes `3`, `6`, `9`, etc. It also takes an optional offset.

`$MultipleOf` can also be utilized to build column and grid layouts. In this case we want to add a `<br>`
after every 3rd item.

```ss
<% loop $Children %>
    ...
    <% if $MultipleOf(3) %>
        <br>
    <% end_if %>
<% end_loop %>
```

### Escaping

Sometimes you will have template tags which need to roll into one another. Use `{}` to contain variables.

```ss
$Foopx <%-- returns "" (as it looks for a `Foopx` variable) --%>
{$Foo}px  <%-- returns "3px" --%>
```

Or when having a `$` sign in front of the variable such as displaying money.

```ss
$$Foo <%-- returns "" --%>
${$Foo} <%-- returns "$3" --%>
```

You can also use a backslash to escape the name of the variable, such as:

```ss
$Foo <%-- returns "3" --%>
\$Foo <%-- returns "$Foo" --%>
```

> [!TIP]
> For more information on formatting and casting variables see [Formatting, Modifying and Casting Variables](casting)

## Scope

In the `<% loop %>` section, we saw an example of two **scopes**. Outside the `<% loop %>...<% end_loop %>`, we were in
the scope of the top level `Page`. But inside the loop, we were in the scope of an item in the list (i.e.the `Child`).

The scope determines where the value comes from when you refer to a variable. Typically the outer scope of a `Page.ss`
layout template is the [PageController](api:SilverStripe\CMS\Controllers\ContentController\PageController) that is currently being rendered.

When the scope is a `PageController` it will automatically also look up any methods in the corresponding `Page` data
record. In the case of `$Title` the flow looks like

```text
$Title --> [Looks up: Current PageController and parent classes] --> [Looks up: Current Page and parent classes]
```

The list of variables you could use in your template is the total of all the methods in the current scope object, parent
classes of the current scope object, any failovers for the current scope object (e.g. controllers and pages can access each others' methods/properties),
and any [`Extension`](api:SilverStripe\Core\Extension) instances you have applied to any of those.

### Navigating scope

#### Up

When in a particular scope, `$Up` takes the scope back to the previous level.

```ss
<h1>Children of '$Title'</h1>

<% loop $Children %>
    <p>Page '$Title' is a child of '$Up.Title'</p>

    <% loop $Children %>
        <p>Page '$Title' is a grandchild of '$Up.Up.Title'</p>
    <% end_loop %>
<% end_loop %>
```

Given the following structure:

```text
My Page
|
+-+ Child 1
|   |
|   +- Grandchild 1
|
+-+ Child 2
```

It will create this markup:

```html
<h1>Children of 'My Page'</h1>

<p>Page 'Child 1' is a child of 'My Page'</p>
<p>Page 'Grandchild 1' is a grandchild of 'My Page'</p>
<p>Page 'Child 2' is a child of 'MyPage'</p>
```

> [!WARNING]
> Each `<% loop %>` or `<% with %>` block results in a change of scope, regardless of how the objects are traversed in the opening statement. See the example below:

```ss
{$Title} <%-- Page title --%>
<% with $Members.First.Organisation %>
    {$Title} <%-- Organisation title --%>
    {$Up.Title} <%-- Page title --%>
    {$Up.Members.First.Name} <%-- Member name --%>
<% end_with %>
```

#### Top

While `$Up` provides us a way to go up one level of scope, `$Top` is a shortcut to jump to the top most scope of the
template. The previous example could be rewritten to use the following syntax.

```ss
<h1>Children of '$Title'</h1>

<% loop $Children %>
    <p>Page '$Title' is a child of '$Top.Title'</p>

    <% loop $Children %>
        <p>Page '$Title' is a grandchild of '$Top.Title'</p>
    <% end_loop %>
<% end_loop %>
```

### With

The `<% with %>` tag lets you change into a new scope. Consider the following example:

```ss
<% with $CurrentMember %>
    Hello, $FirstName, welcome back. Your current balance is $Balance.
<% end_with %>
```

This is functionalty the same as the following:

```ss
Hello, $CurrentMember.FirstName, welcome back. Your current balance is $CurrentMember.Balance
```

Notice that the first example is much tidier, as it removes the repeated use of the `$CurrentMember` accessor.

Outside the `<% with %>`, we are in the page scope. Inside it, we are in the scope of `$CurrentMember` object. We can
refer directly to properties and methods of the [`Member`](api:SilverStripe\Security\Member) object. `$FirstName` inside the scope is equivalent to
`$CurrentMember.FirstName`.

### `fortemplate()` and `$Me` {#fortemplate}

If you reference some object directly in a template, the `forTemplate()` method on that object will be called. This can be used to provide a default template for an object.
If the `forTemplate()` method isn't implemented, the system will attempt to cast your object to a string.
The default implementation of this method on `ModelData` renders the model using templates named after the class or its superclasses.

```ss
<%-- calls forTemplate() on the first page in the list --%>
$Pages->First
```

You can also use the `$Me` variable, which outputs the current object in scope by calling `forTemplate()` on the object.
This is especially helpful when you want to directly render items in a list you're looping over.

> [!WARNING]
> If the object does not have an appropriate template, implement the `forTemplate()` method, or implement [`__toString()`](https://www.php.net/manual/en/language.oop5.magic.php#object.tostring), this will throw an error.

```ss
<% loop $Pages %>
    <%-- calls forTemplate() on the current object in scope --%>
    $Me
<% end_loop %>
```

> [!NOTE]
> If you use `$Me` in a control block such as `<% if $Me %>` or `<% loop $Me %>` it references the item directly rather than calling `forTemplate()` on it.

## Comments

Using standard HTML comments is supported. These comments will be included in the published site.

```ss
$EditForm <!-- Some public comment about the form -->
```

However you can also use special Silverstripe CMS comments which will be stripped out of the published site. This is useful
for adding notes for other developers but for things you don't want published in the public html.

```ss
$EditForm <%-- Some hidden comment about the form --%>
```

## Related documentation

[CHILDREN Exclude="How_Tos"]

## How to's

[CHILDREN Folder="How_Tos"]

## API documentation

- [SSViewer](api:SilverStripe\View\SSViewer)
- [ThemeManifest](api:SilverStripe\View\ThemeManifest)
