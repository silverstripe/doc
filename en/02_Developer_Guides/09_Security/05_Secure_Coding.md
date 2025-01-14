---
title: Security
summary: Learn how to minimise vulnerabilities in your code
icon: user-secret
---

# Security

## Introduction

This page details notes on how to ensure that we develop secure Silverstripe CMS applications.
See [Reporting Security Issues](/contributing/issues_and_bugs#reporting-security-issues) on how to report potential vulnerabilities.

## SQL injection

The [coding-conventions](/contributing/coding_conventions) help guard against SQL injection attacks but still require developer
diligence: ensure that any variable you insert into a filter / sort / join clause is either parameterised, or has been
escaped.

See the [OWASP article on SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection) for more information.

### Parameterised queries

Parameterised queries, or prepared statements, allow the logic around the query and its structure to be separated from
the parameters passed in to be executed. Many DB adaptors support these as standard including
[MySQL](https://php.net/manual/en/mysqli.prepare.php), [SQL Server](https://php.net/manual/en/function.sqlsrv-prepare.php),
[SQLite](https://php.net/manual/en/sqlite3.prepare.php), and [PostgreSQL](https://php.net/manual/en/function.pg-prepare.php).

The use of parameterised queries whenever possible will safeguard your code in most cases, but care
must still be taken when working with literal values or table/column identifiers that may
come from user input.

Example:

```php
use SilverStripe\ORM\DB;
use SilverStripe\ORM\DataObject;
use SilverStripe\ORM\Queries\SQLSelect;

$records = DB::prepared_query('SELECT * FROM "MyClass" WHERE "ID" = ?', [3]);
$records = MyClass::get()->where(['"ID" = ?' => 3]);
$records = MyClass::get()->where(['"ID"' => 3]);
$records = DataObject::get_by_id('MyClass', 3);
$records = DataObject::get_one('MyClass', ['"ID" = ?' => 3]);
$records = MyClass::get()->byID(3);
$records = SQLSelect::create()->addWhere(['"ID"' => 3])->execute();
```

Parameterised updates and inserts are also supported, but the syntax is a little different

```php
use SilverStripe\ORM\DB;
use SilverStripe\ORM\Queries\SQLInsert;

SQLInsert::create('"MyClass"')
    ->assign('"Name"', 'Daniel')
    ->addAssignments([
        '"Position"' => 'Accountant',
        '"Age"' => [
            'GREATEST(0,?,?)' => [24, 28],
        ],
    ])
    ->assignSQL('"Created"', 'NOW()')
    ->execute();
DB::prepared_query(
    'INSERT INTO "MyClass" ("Name", "Position", "Age", "Created") VALUES(?, ?, GREATEST(0,?,?), NOW())'
    ['Daniel', 'Accountant', 24, 28]
);
```

### Automatic escaping

Silverstripe CMS internally will use parameterised queries in SQL statements wherever possible.

If necessary Silverstripe performs any required escaping through database-specific methods (see [`Database::addslashes()`](api:SilverStripe\ORM\Connect\Database::addslashes())).
For [`MySQLDatabase`](api:SilverStripe\ORM\Connect\MySQLDatabase), this will be [`mysqli::real_escape_string()`](https://www.php.net/manual/en/mysqli.real-escape-string.php).

- Most [`DataList`](api:SilverStripe\ORM\DataList) accessors (see escaping note in method documentation)
- [`DataObject::get_by_id()`](api:SilverStripe\ORM\DataObject::get_by_id())
- [`DataObject::update()`](api:SilverStripe\ORM\DataObject::update())
- [`DataObject::castedUpdate()`](api:SilverStripe\ORM\DataObject::castedUpdate())
- `$dataObject->SomeField = 'val'`, [`DataObject::setField()`](api:SilverStripe\ORM\DataObject::setField())
- [`DataObject::write()`](api:SilverStripe\ORM\DataObject::write())
- [`DataList::byID()`](api:SilverStripe\ORM\DataList::byID())
- [`Form::saveInto()`](api:SilverStripe\Forms\Form::saveInto())
- [`FormField::saveInto()`](api:SilverStripe\Forms\FormField::saveInto())
- [`DBField::saveInto()`](api:SilverStripe\ORM\FieldType\DBField::saveInto())

Data is not escaped when writing to object-properties, as inserts and updates are normally
handled via prepared statements.

Example:

```php
use SilverStripe\Security\Member;

// automatically escaped/quoted
$members = Member::get()->filter('Name', $_GET['name']);
// automatically escaped/quoted
$members = Member::get()->filter(['Name' => $_GET['name']]);
// parameterised condition
$members = Member::get()->where(['"Name" = ?' => $_GET['name']]);
// needs to be escaped and quoted manually (note raw2sql called with the $quote parameter set to true)
$members = Member::get()->where(sprintf('"Name" = %s', Convert::raw2sql($_GET['name'], true)));
```

> [!WARNING]
> It is NOT good practice to "be sure" and convert the data passed to the functions above manually. This might
> result in *double escaping* and alters the actually saved data (e.g. by adding slashes to your content).

### Manual escaping

As a rule of thumb, whenever you're creating SQL queries (or just chunks of SQL) you should use parameterisation,
but there may be cases where you need to take care of escaping yourself. See [coding-conventions](/getting_started/coding-conventions)
and [datamodel](/developer_guides/model) for ways to parameterise, cast, and convert your data.

- [`SQLSelect`](api:SilverStripe\ORM\Queries\SQLSelect)
- [`DB::query()`](api:SilverStripe\ORM\DB::query())
- [`DB::prepared_query()`](api:SilverStripe\ORM\DB::prepared_query())
- `Controller->requestParams`
- `Controller->urlParams`
- [`HTTPRequest`](api:SilverStripe\Control\HTTPRequest) data
- `GET`/`POST` data passed to a form method

Example:

```php
namespace App\Form;

use App\Model\Player;
use SilverStripe\Core\Convert;
use SilverStripe\Forms\Form;

class MyForm extends Form
{
    public function save($RAW_data, $form)
    {
        // Pass true as the second parameter of raw2sql to quote the value safely
        // works recursively on an array
        $SQL_data = Convert::raw2sql($RAW_data, true);
        $objs = Player::get()->where('Name = ' . $SQL_data['name']);
        // ...
    }
}
```

- `FormField->Value()`
- URLParams passed to a Controller-method

Example:

```php
namespace App\Control;

use App\Model\Player;
use SilverStripe\Control\Controller;
use SilverStripe\Core\Convert;

class MyController extends Controller
{
    private static $allowed_actions = ['myurlaction'];

    public function myurlaction($RAW_urlParams)
    {
        // Pass true as the second parameter of raw2sql to quote the value safely
        // works recursively on an array
        $SQL_urlParams = Convert::raw2sql($RAW_urlParams, true);
        $objs = Player::get()->where('Name = ' . $SQL_data['OtherID']);
        // ...
    }
}
```

As a rule of thumb, you should escape your data **as close to querying as possible**
(or preferably, use parameterised queries). This means if you've got a chain of functions
passing data through, escaping should happen at the end of the chain.

```php
namespace App\Control;

use SilverStripe\Control\Controller;
use SilverStripe\Core\Convert;
use SilverStripe\ORM\DB;

class MyController extends Controller
{
    /**
    * @param array $RAW_data All names in an indexed array (not SQL-safe)
    */
    public function saveAllNames($RAW_data)
    {
        // $SQL_data = Convert::raw2sql($RAW_data); // premature escaping
        foreach ($RAW_data as $item) {
            $this->saveName($item);
        }
    }

    public function saveName($RAW_name)
    {
        $SQL_name = Convert::raw2sql($RAW_name, true);
        DB::query("UPDATE Player SET Name = {$SQL_name}");
    }
}
```

This might not be applicable in all cases - especially if you are building an API thats likely to be customised. If
you're passing unescaped data, make sure to be explicit about it by writing *phpdoc*-documentation and *prefixing* your
variables ($RAW_data instead of $data).

## XSS (cross-site-scripting)

Silverstripe CMS helps you guard any output against clientside attacks initiated by malicious user input, commonly known as
XSS (Cross-Site-Scripting). With some basic guidelines, you can ensure your output is safe for a specific use case (e.g.
displaying a blog post in HTML from a trusted author, or escaping a search parameter from an untrusted visitor before
redisplaying it).

> [!WARNING]
> Note: Make sure you use the [correct casting](/developer_guides/model/data_types_and_casting/#casting) when including content in a Silverstripe CMS template.

See the [OWASP article on XSS](https://owasp.org/www-community/attacks/xss/) for more information.

### Additional options

For `HTMLText` database fields which aren't edited through `HtmlEditorField`, you also
have the option to explicitly whitelist allowed tags in the field definition, e.g. `"MyField" => "HTMLText('meta','link')"`.
The `SiteTree.ExtraMeta` property uses this to limit allowed input.

You can also use the [`XssSanitiser`](api:SilverStripe\Core\XssSanitiser) to remove some known XSS attack vectors from HTML content. Note that this should be used only in scenarios where the HTML content can't be completely removed, and should not be considered a complete protection against all XSS attack vectors but rather as simply one of many tools in your security tool box.

### What if I need to allow script or style tags?

The default configuration of Silverstripe CMS uses a santiser to enforce TinyMCE whitelist rules on the server side,
and is sufficient to eliminate the most common XSS vectors. Notably, this will remove script and style tags.

If your site requires script or style tags to be added via TinyMCE, Silverstripe CMS can be configured to disable the
server side santisation. You will also need to update the TinyMCE whitelist [settings](/developer_guides/forms/field_types/htmleditorfield/#setting-options) to remove the frontend sanitisation.

However, it's strongly discouraged as it opens up the possibility of malicious code being added to your site through the CMS.

To disable filtering, set the `HtmlEditorField::$sanitise_server_side` [configuration](/developer_guides/configuration/configuration) property to `false`, i.e.

```yml
---
Name: project-htmleditor
After: htmleditor
---
SilverStripe\Forms\HTMLEditor\HTMLEditorField:
  sanitise_server_side: false
```

Note it is not currently possible to allow editors to provide JavaScript content and yet still protect other users
from any malicious code within that JavaScript.

We recommend configuring [shortcodes](/developer_guides/extending/shortcodes) that can be used by editors in place of using JavaScript directly.

### Escaping model properties

Before outputting values to the template layer, [`ViewLayerData`](api:SilverStripe\View\ViewLayerData) automatically takes care of escaping HTML tags from specific
object-properties by [casting](/developer_guides/model/data_types_and_casting) its string value into a [`DBField`](api:SilverStripe\ORM\FieldType\DBField) object.

PHP:

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyObject extends DataObject
{
    private static $db = [
        // Example value: <b>not bold</b>
        'MyEscapedValue' => 'Text',
        // Example value: <b>bold</b>
        'MyUnescapedValue' => 'HTMLText',
    ];
}
```

Template:

```ss
 <ul>
    <li>$MyEscapedValue</li> <%-- output: &lt;b&gt;not bold&lt;b&gt; --%>
    <li>$MyUnescapedValue</li> <%-- output: <b>bold</b> --%>
</ul>
```

The example below assumes that data wasn't properly filtered when saving to the database, but are escaped before
outputting rendered template results through `SSViewer`.

### Overriding default escaping in templates

You can force escaping on a casted value/object by using an [escape type](/developer_guides/model/data_types_and_casting) method in your template, e.g.
"XML" or "ATT".

Template (see above):

```ss
 <ul>
    <%-- output: <a href="#" title="foo &amp; &#quot;bar&quot;">foo &amp; "bar"</a> --%>
    <li><a href="#" title="$Title.ATT">$Title </a></li>
    <li>$MyEscapedValue</li> <%-- output: &lt;b&gt;not bold&lt;b&gt; --%>
    <li>$MyUnescapedValue</li> <%-- output: <b>bold</b> --%>
    <li>$MyUnescapedValue.XML</li> <%-- output: &lt;b&gt;bold&lt;b&gt; --%>
</ul>
```

### Escaping custom attributes and getters

Every object attribute or getter method used for template purposes should have its escape type defined through the
`casting` configuration property (assuming you're using a `ModelData` subclass). Caution: Casting only applies when using values in a template, not in PHP.

PHP:

```php
namespace App\Model;

use SilverStripe\ORM\DataObject;

class MyObject extends DataObject
{
    private string $title = '<b>not bold</b>';

    private static $casting = [
        // forcing a casting
        'Title' => 'Text',
        // optional, as HTMLText is the default casting
        'TitleWithHTMLSuffix' => 'HTMLText',
    ];

    public function getTitle()
    {
        // will be escaped due to Text casting
        return $this->title;
    }

    public function getTitleWithHTMLSuffix($suffix)
    {
        // $this->getTitle() is not casted in PHP
        return $this->getTitle() . '<small>(' . $suffix . ')</small>';
    }
}
```

> [!NOTE]
> If `$title` was a public property called `$Title`, it would also be casted the same way that the result of
> `$getTitle()` is casted.

Template:

```ss
 <ul>
    <li>$Title</li> <%-- output: &lt;b&gt;not bold&lt;b&gt; --%>
    <li>$Title.RAW</li> <%-- output: <b>not bold</b> --%>
    <li>$TitleWithHTMLSuffix</li> <%-- output: <b>not bold</b>: <small>(...)</small> --%>
</ul>
```

Note: Avoid generating HTML by string concatenation in PHP wherever possible to minimize risk and separate your
presentation from business logic.

### Manual escaping in PHP

When using *customise()* or *renderWith()* calls in your controller, or otherwise forcing a custom context for your
template, you'll need to take care of casting and escaping yourself in PHP.

The [Convert](api:SilverStripe\Core\Convert) class has utilities for this, mainly *Convert::raw2xml()* and *Convert::raw2att()* (which is
also used by *XML* and *ATT* in template code).

> [!WARNING]
> Most of the `Convert::raw2` methods accept arrays and do not affect array keys.
> If you serialize your data, make sure to do that before you pass it to `Convert::raw2` methods.
>
> For example:
>
> ```php
> use SilverStripe\Core\Convert;
>
> // WRONG!
> json_encode(Convert::raw2sql($request->getVar('multiselect')));
>
> // Correct!
> Convert::raw2sql(json_encode($request->getVar('multiselect')));
> ```

PHP:

```php
namespace App\Control;

use SilverStripe\Control\Controller;
use SilverStripe\Core\Convert;
use SilverStripe\ORM\FieldType\DBHTMLText;
use SilverStripe\ORM\FieldType\DBText;

class MyController extends Controller
{
    private static $allowed_actions = ['search'];

    public function search($request)
    {
        $htmlTitle = '<p>Your results for:' . Convert::raw2xml($request->getVar('Query')) . '</p>';
        return $this->customise([
            'Query' => DBText::create($request->getVar('Query')),
            'HTMLTitle' => DBHTMLText::create($htmlTitle),
        ]);
    }
}
```

Template:

```ss
 <h2 title="Searching for $Query.ATT">$HTMLTitle</h2>
```

Whenever you insert a variable into an HTML attribute within a template, use $VarName.ATT, no not $VarName.

You can also use the built-in casting in PHP by using the *obj()* wrapper, see [datamodel](/developer_guides/model/data_types_and_casting).

### Escaping URLs

Whenever you are generating a URL that contains querystring components based on user data, use urlencode() to escape the
user data, not *Convert::raw2att()*.  Use raw ampersands in your URL, and cast the URL as a "Text" DBField:

PHP:

```php
namespace App\Control;

use SilverStripe\Control\Controller;
use SilverStripe\ORM\FieldType\DBText;

class MyController extends Controller
{
    private static $allowed_actions = ['search'];

    public function search($request)
    {
        $rssRelativeLink = '/rss?Query=' . urlencode($_REQUEST['query']) . '&sortOrder=asc';
        $rssLink = Controller::join_links($this->Link(), $rssRelativeLink);
        return $this->customise([
            'RSSLink' => DBText::create($rssLink),
        ]);
    }
}
```

Template:

```ss
<a href="$RSSLink.ATT">RSS feed</a>
```

Some rules of thumb:

- Don't concatenate URLs in a template.  It only works in extremely simple cases that usually contain bugs.
- Use *Controller::join_links()* to concatenate URLs.  It deals with query strings and other such edge cases.

## Cross-Site request forgery (CSRF)

Silverstripe CMS has built-in countermeasures against CSRF identity theft for all form submissions. A form object
will automatically contain a `SecurityID` parameter which is generated as a secure hash on the server, connected to the
currently active session of the user. If this form is submitted without this parameter, or if the parameter doesn't
match the hash stored in the users session, the request is discarded.
You can disable this behaviour through [Form::disableSecurityToken()](api:SilverStripe\Forms\Form::disableSecurityToken()).

It is also recommended to limit form submissions to the intended HTTP verb (mostly `GET` or `POST`)
through [Form::setStrictFormMethodCheck()](api:SilverStripe\Forms\Form::setStrictFormMethodCheck()).

Sometimes you need to handle state-changing HTTP submissions which aren't handled through
Silverstripe CMS's form system. In this case, you can also check the current HTTP request
for a valid token through [SecurityToken::checkRequest()](api:SilverStripe\Security\SecurityToken::checkRequest()).

See the [OWASP article about CSRF](https://owasp.org/www-community/attacks/csrf) for more information.

## Casting user input

When working with `$_GET`, `$_POST` or `Director::urlParams` variables, and you know your variable has to be of a
certain type, like an integer, then it's essential to cast it as one. *Why?* To be sure that any processing of your
given variable is done safely, with the assumption that it's an integer.

To cast the variable as an integer, place `(int)` or `(integer)` before the variable.

For example: a page with the URL parameters *example.com/home/add/1* requires that ''Director::urlParams['ID']'' be an
integer. We cast it by adding `(int)` - ''(int)Director::urlParams['ID']''. If a value other than an integer is
passed, such as *example.com/home/add/dfsdfdsfd*, then it returns 0.

Below is an example with different ways you would use this casting technique:

```php
namespace App\PageType;

use App\Model\CaseStudy;
use Page;
use SilverStripe\Control\Director;

class CaseStudyPage extends Page
{
    public function getCaseStudies()
    {
        // cast an ID from URL parameters e.g. (example.com/home/action/ID)
        $anotherID = (int) Director::urlParam['ID'];

        // perform a calculation, the prerequisite being $anotherID must be an integer
        $calc = $anotherID + (5 - 2) / 2;

        // cast the 'category' GET variable as an integer
        $categoryID = (int) $_GET['category'];

        // perform a byID(), which ensures the ID is an integer before querying
        return CaseStudy::get()->byID($categoryID);
    }
}
```

The same technique can be employed anywhere in your PHP code you know something must be of a certain type. A list of PHP
cast types can be found here:

- `(int)`, `(integer)` - cast to integer
- `(bool)`, `(boolean)` - cast to boolean
- `(float)`, `(double)`, `(real)` - cast to float
- `(string)` - cast to string
- `(array)` - cast to array
- `(object)` - cast to object

Note that there is also a 'Silverstripe CMS' way of casting fields on a class, this is a different type of casting to the
standard PHP way. See [casting](/developer_guides/model/data_types_and_casting).

## Filesystem

### Don't allow script-execution in /assets

Please refer to the article on [file security](/developer_guides/files/file_security)
for instructions on how to secure the assets folder against malicious script execution.

### Don't run Silverstripe in the webroot

Silverstripe routes all execution through a [`public/` subfolder](/getting_started/directory_structure)
by default. This enables you to keep application code and configuration outside of webserver routing.

```text
.htaccess <- fallback, shouldn't be used
public/ <- this should be your webroot
  .htaccess
  index.php
app/
  _config/
    secrets.yml <- your webserver shouldn't be able to serve this, as it's outside of the public/ folder
```

### Don't place protected files in the webroot

Protected files are stored in `public/assets/.protected` by default
(assuming you're using the [public/ subfolder](/getting_started/directory_structure)).
While default configuration is in place to avoid the webserver serving these files,
we recommend moving them out of the webroot altogether -
see [Server Requirements: Secure Assets](/getting_started/server_requirements#secure-assets).

### User uploaded files

Certain file types are by default excluded from user upload. html, xhtml, htm, and xml files may have embedded,
or contain links to, external resources or scripts that may hijack browser sessions and impersonate that user.
Even if the uploader of this content may be a trusted user, there is no safeguard against these users being
deceived by the content source.

Users with ADMIN priveledges may be allowed to override the above upload restrictions if the
`File.apply_restrictions_to_admin` config is set to false. By default this is true, which enforces these
restrictions globally.

Additionally, if certain file uploads should be made available to non-privileged users, you can add them to the
list of allowed extensions by adding these to the `File.allowed_extensions` config.

## Passwords

Silverstripe CMS stores passwords with a strong hashing algorithm (blowfish) by default
(see [PasswordEncryptor](api:SilverStripe\Security\PasswordEncryptor)). It adds randomness to these hashes via
salt values generated with the strongest entropy generators available on the platform
(see [RandomGenerator](api:SilverStripe\Security\RandomGenerator)). This prevents brute force attacks with
[Rainbow tables](https://en.wikipedia.org/wiki/Rainbow_table).

Strong passwords are a crucial part of any system security.

The default password validator uses the [`PasswordStrength` constraint](https://symfony.com/doc/current/reference/constraints/PasswordStrength.html) in `symfony/validator`, which determines a password's strength based on its level of entropy.

You can change the required strength of valid passwords by setting the [`EntropyPasswordValidator.password_strength`](api:SilverStripe\Security\Validation\EntropyPasswordValidator->password_strength) configuration property to one of the valid [minScore values](https://symfony.com/doc/current/reference/constraints/PasswordStrength.html#minscore):

```yml
SilverStripe\Security\Validation\EntropyPasswordValidator:
  password_strength: 4
```

You can also enforce that passwords are not repeated by setting the [`PasswordValidator.historic_count`](api:SilverStripe\Security\Validation\PasswordValidator->historic_count) configuration property:

```yml
SilverStripe\Security\Validation\PasswordValidator:
  historic_count: 6
```

The above example will check that the password wasn't used within the previous 6 passwords set for the member.

### Rule-based password validation

If you want more finegrained control over exactly how a "strong" password is determined, you can use the [`RulesPasswordValidator`](api:SilverStripe\Security\Validation\RulesPasswordValidator) which uses an array of regular expressions to validate a password. You can swap to using that validator and configure its options with YAML configuration:

```yml
---
Name: mypasswords
After: '#corepasswords'
---
SilverStripe\Core\Injector\Injector:
  SilverStripe\Security\Validation\PasswordValidator:
    class: 'SilverStripe\Security\Validation\RulesPasswordValidator'

SilverStripe\Security\Validation\RulesPasswordValidator:
  min_length: 7
  min_test_score: 3
```

> [!NOTE]
> The [`PasswordValidator.historic_count`](api:SilverStripe\Security\Validation\PasswordValidator->historic_count) configuration property also applies to the `RulesPasswordValidator`.

You can also add additional regular expression tests to the validator:

```yml
SilverStripe\Security\Validation\RulesPasswordValidator:
  character_strength_tests:
    at-least-three-special-chars: '/[\(\)\&\^\%\$\#\@\!]{3,}/'
```

The above example requires at least 3 of the characters `()&^%$#@!` to be included in the password.

Note that the [`RulesPasswordValidator.min_test_score`](api:SilverStripe\Security\Validation\RulesPasswordValidator->min_test_score) configuration property determines how many of the regular expression tests must pass for a password to be valid. If the test score is lower than the number of tests you have, the password *doesn't* have to match all of them to be valid.

### More password security options

In addition, you can tighten password security with the following configuration settings:

- `Member.password_expiry_days`: Set the number of days that a password should be valid for.
- `Member.lock_out_after_incorrect_logins`: Number of incorrect logins after which
    the user is blocked from further attempts for the timespan defined in `$lock_out_delay_mins`
- `Member.lock_out_delay_mins`: Minutes of enforced lockout after incorrect password attempts. Only applies if `lock_out_after_incorrect_logins` is greater than 0.
- `Security.remember_username`: Set to false to disable autocomplete on login form
- `Session.timeout`: Set timeout to attenuate the risk of active sessions being exploited

## Clickjacking: prevent iframe inclusion

"[Clickjacking](https://owasp.org/www-community/attacks/Clickjacking)"  is a malicious technique
where a web user is tricked into clicking on hidden interface elements, which can
lead to the attacker gaining access to user data or taking control of the website behaviour.

You can signal to browsers that the current response isn't allowed to be
included in HTML "frame" or "iframe" elements, and thereby prevent the most common
attack vector. This is done through a HTTP header, which is usually added in your
controller's `init()` method:

```php
namespace App\Control;

use SilverStripe\Control\Controller;

class MyController extends Controller
{
    public function init()
    {
        parent::init();
        $this->getResponse()->addHeader('X-Frame-Options', 'SAMEORIGIN');
    }
}
```

This is a recommended option to secure any controller which displays
or submits sensitive user input, and is enabled by default in all CMS controllers,
as well as the login form.

## Request hostname forgery {#request-hostname-forgery}

To prevent a forged hostname appearing being used by the application, Silverstripe CMS
allows the configure of a whitelist of hosts that are allowed to access the system. By defining
this whitelist in your `.env` file, any request presenting a `Host` header that is
*not* in this list will be blocked with a HTTP 400 error:

```bash
SS_ALLOWED_HOSTS="www.example.com,example.com,subdomain.example.com"
```

Please note that if this configuration is defined, you *must* include *all* subdomains (eg `<www>.`)
that will be accessing the site.

When Silverstripe CMS is run behind a reverse proxy, it's normally necessary for this proxy to
use the `X-Forwarded-Host` request header to tell the webserver which hostname was originally
requested. However, when Silverstripe CMS is not run behind a proxy, this header can still be
used by attackers to fool the server into mistaking its own identity.

The risk of this kind of attack causing damage is especially high on sites which utilise caching
mechanisms, as rewritten urls could persist between requests in order to misdirect other users
into visiting external sites.

In order to prevent this kind of attack, it's necessary to whitelist trusted proxy
server IPs using the SS_TRUSTED_PROXY_IPS define in your `.env`.

```bash
SS_TRUSTED_PROXY_IPS="127.0.0.1,192.168.0.1"
```

You can also whitelist subnets in CIDR notation if you don't know the exact IP of a trusted proxy.
For example, some cloud provider load balancers don't have fixed IPs.

```bash
SS_TRUSTED_PROXY_IPS="10.10.0.0/24,10.10.1.0/24,10.10.2.0/24"
```

If you wish to change the headers that are used to find the proxy information, you should reconfigure the
TrustedProxyMiddleware service:

```yml
SilverStripe\Control\TrustedProxyMiddleware:
  properties:
    ProxyHostHeaders: X-Forwarded-Host
    ProxySchemeHeaders: X-Forwarded-Protocol
    ProxyIPHeaders: X-Forwarded-Ip
```

```bash
SS_TRUSTED_PROXY_HOST_HEADER="HTTP_X_FORWARDED_HOST"
SS_TRUSTED_PROXY_IP_HEADER="HTTP_X_FORWARDED_FOR"
SS_TRUSTED_PROXY_PROTOCOL_HEADER="HTTP_X_FORWARDED_PROTOCOL"
```

At the same time, you'll also need to define which headers you trust from these proxy IPs. Since there are multiple ways through which proxies can pass through HTTP information on the original hostname, IP and protocol, these values need to be adjusted for your specific proxy. The header names match their equivalent `$_SERVER` values.

If there is no proxy server, 'none' can be used to distrust all clients.
If only trusted servers will make requests then you can use '*' to trust all clients.
Otherwise a comma separated list of individual IP addresses (or subnets in CIDR notation) should be declared.

This behaviour is enabled whenever `SS_TRUSTED_PROXY_IPS` is defined, or if the
`BlockUntrustedIPs` environment variable is declared. It is advisable to include the
following in your .htaccess to ensure this behaviour is activated.

```text
<IfModule mod_env.c>
    # Ensure that X-Forwarded-Host is only allowed to determine the request
    # hostname for servers ips defined by SS_TRUSTED_PROXY_IPS in your .env
    # Note that in a future release this setting will be always on.
    SetEnv BlockUntrustedIPs true
</IfModule>
```

This behaviour is on by default; the environment variable is not required. For correct operation, it is necessary to always set `SS_TRUSTED_PROXY_IPS` if using a proxy.

## Secure sessions, cookies and TLS (HTTPS)

Silverstripe CMS recommends the use of TLS (HTTPS) for your application, and you can easily force the use through the
director function `forceSSL()`

```php
use SilverStripe\Control\Director;

if (!Director::isDev()) {
    Director::forceSSL();
}
```

`forceSSL()` will only take effect in environment types that `CanonicalURLMiddleware` is configured to apply to (by
default, only `LIVE`). To apply this behaviour in all environment types, you'll need to update that configuration:

```php
use SilverStripe\Control\Director;
use SilverStripe\Control\Middleware\CanonicalURLMiddleware;

if (!Director::isDev()) {
    // You can also specify individual environment types
    CanonicalURLMiddleware::singleton()->setEnabledEnvs(true);
    Director::forceSSL();
}
```

Forcing HTTPS so requires a certificate to be purchased or obtained through a vendor such as
[lets encrypt](https://letsencrypt.org/) and configured on your web server.

Note that by default enabling SSL will also enable `CanonicalURLMiddleware::forceBasicAuthToSSL` which will detect
and automatically redirect any requests with basic authentication headers to first be served over HTTPS. You can
disable this behaviour using `CanonicalURLMiddleware::singleton()->setForceBasicAuthToSSL(false)`, or via Injector
configuration in YAML.

We also want to ensure cookies are not shared between secure and non-secure sessions, so we must tell Silverstripe CMS to
use a [secure session](/developer_guides/cookies_and_sessions/sessions/#secure-session-cookie).
To do this, you may set the `cookie_secure` parameter to `true` in your `config.yml` for `Session`.

It is also a good idea to set the `samesite` attribute for the session cookie to `Strict` unless you have a specific use case for
sharing the session cookie across domains.

```yml
SilverStripe\Control\Session:
  cookie_samesite: 'Strict'
  cookie_secure: true
```

The same treatment should be applied to the cookie responsible for remembering logins across sessions:

```yml
---
Name: secure-alc
Except:
  environment: dev
---
SilverStripe\Core\Injector\Injector:
  SilverStripe\Security\MemberAuthenticator\CookieAuthenticationHandler:
    properties:
      TokenCookieSecure: true
```

> [!NOTE]
> There is not currently an easy way to pass a `samesite` attribute value for setting this cookie - but you can set the
> default value for the attribute for all cookies. See [the main cookies documentation](/developer_guides/cookies_and_sessions/cookies#samesite-attribute) for more information.

For other cookies set by your application we should also ensure the users are provided with secure cookies by setting
the "Secure" and "HTTPOnly" flags. These flags prevent them from being stolen by an attacker through JavaScript.

- The `Secure` cookie flag instructs the browser not to send the cookie over an insecure HTTP connection. If this
flag is not present, the browser will send the cookie even if HTTPS is not in use, which means it is transmitted in
clear text and can be intercepted and stolen by an attacker who is listening on the network.

- The `HTTPOnly` flag lets the browser know whether or not a cookie should be accessible by client-side JavaScript
code. It is best practice to set this flag unless the application is known to use JavaScript to access these cookies
as this prevents an attacker who achieves cross-site scripting from accessing these cookies.

```php
use SilverStripe\Control\Cookie;

Cookie::set(
    'cookie-name',
    'chocolate-chip',
    $expiry = 30,
    $path = null,
    $domain = null,
    $secure = true,
    $httpOnly = false
);
```

### Using SSL in database connections

In some circumstances, like connecting to a database on a remote host for example, you may wish to enable SSL encryption to ensure the protection of sensitive information and database access credentials.
You can configure that by setting the following environment variables:

| Name  | Description |
| ----  | ----------- |
| `SS_DATABASE_SSL_KEY` | Absolute path to SSL key file (optional - but if set, `SS_DATABASE_SSL_CERT` must also be set) |
| `SS_DATABASE_SSL_CERT` | Absolute path to SSL certificate file (optional - but if set, `SS_DATABASE_SSL_KEY` must also be set) |
| `SS_DATABASE_SSL_CA` | Absolute path to SSL Certificate Authority bundle file (optional) |
| `SS_DATABASE_SSL_CIPHER` | Custom SSL cipher for database connections (optional) |

## Security headers

In addition to forcing HTTPS browsers can support additional security headers which can only allow access to a website
via a secure connection. As browsers increasingly provide negative feedback regarding unencrypted HTTP connections,
ensuring an HTTPS connection will provide a better and more secure user experience.

- The `Strict-Transport-Security` header instructs the browser to record that the website and assets on that website
MUST use a secure connection. This prevents websites from becoming insecure in the future from stray absolute links
or references without https from external sites. Check if your browser supports [HSTS](https://hsts.badssl.com/)
- `max-age` can be configured to anything in seconds: `max-age=31536000` (1 year), for roll out, consider something
  lower
- `includeSubDomains` to ensure all present and future sub domains will also be HTTPS

For sensitive pages, such as members areas, or places where sensitive information is present, adding cache control
 headers can explicitly instruct browsers not to keep a local cached copy of content and can prevent content from
 being cached throughout the infrastructure (e.g. Proxy, caching layers, WAF etc).

- The headers `Cache-control: no-store` and `Pragma: no-cache` along with expiry headers of `Expires: <current date>`
and `Date: <current date>` will ensure that sensitive content is not stored locally or able to be retrieved by
unauthorised local persons. Silverstripe CMS adds the current date for every request, and we can add the other cache
 headers to the request for our secure controllers:

```php
namespace App\Control;

use SilverStripe\Control\Controller;
use SilverStripe\Control\HTTP;

class MySecureController extends Controller
{
    public function init()
    {
        parent::init();

        // Add cache headers to ensure sensitive content isn't cached.
        $this->response->addHeader('Cache-Control', 'max-age=0, must-revalidate, no-transform');
        // for HTTP 1.0 support
        $this->response->addHeader('Pragma', 'no-cache');

        HTTP::set_cache_age(0);
        HTTP::add_cache_headers($this->response);

        // Add HSTS header to force TLS for document content
        $this->response->addHeader('Strict-Transport-Security', 'max-age=86400; includeSubDomains');
    }
}
```

## HTTP caching headers

Caching is hard. If you get it wrong, private or draft content might leak
to unauthenticated users. We have created an abstraction which allows you to express
your intent around HTTP caching without worrying too much about the details.
See [HTTP Cache Headers](/developer_guides/performance/http_cache_headers/)
for details on how to apply caching safely, and read Google's
[Web Fundamentals on Caching](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching).

## Related

- [Silverstripe CMS security vulnerability advisories](https://silverstripe.org/security-releases/)
- [MySQL security documentation](https://dev.mysql.com/doc/refman/8.4/en/security.html)
- [OWASP Top Ten](https://owasp.org/www-project-top-ten/)
- [OWASP List of Attacks](https://owasp.org/www-community/attacks/)
