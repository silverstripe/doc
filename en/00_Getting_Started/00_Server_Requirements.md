---
title: Server Requirements
icon: server
summary: What you will need to run Silverstripe CMS on a web server
---

# Requirements

Silverstripe CMS needs to be installed on a web server. Content authors and website administrators use their web browser
to access a web-based GUI to do their day-to-day work. Website designers and developers require access to the files on
the server to update templates, website logic, and perform upgrades or maintenance.

## PHP

- PHP 8.3 - 8.4
- PHP extensions: `ctype`, `dom`, `fileinfo`, `hash`, `intl`, `mbstring`, `session`, `simplexml`, `tokenizer`, `xml`
- PHP configuration: `memory_limit` with at least `48M`
- PHP extension for image manipulation: Either `gd` or `imagick`
- PHP extension for a database connector (e.g. `mysql`)

Use [phpinfo()](https://php.net/manual/en/function.phpinfo.php) to inspect your configuration.

Silverstripe CMS tracks the official [PHP release support timeline](https://www.php.net/supported-versions.php). When a PHP version reaches end-of-life, Silverstripe CMS drops support for it in the next minor release.

You also need to install [Composer 2](https://getcomposer.org/).

## Database

We officially support and regression test against the latest LTS releases of MySQL and MariaDB, though we may choose to support additional versions on a case-by-case basis.

- MySQL >=8.0 and MariaDB (built-in, [commercially supported](/project_governance/supported_modules/))
- PostgreSQL ([third party module](https://github.com/silverstripe/silverstripe-postgresql), community
  supported)
- SQL Server ([third party module](https://github.com/silverstripe/silverstripe-mssql), community supported)
- SQLite ([third party module](https://github.com/silverstripe/silverstripe-sqlite3), community supported)

### Default MySQL collation

New projects default to the `utf8mb4_unicode_ci` collation when running
against MySQL, which offers better support for multi-byte characters such as emoji.

### Connection mode (sql_mode) when using MySQL server >=8.0.0

In MySQL versions >=8.0.0, the `ANSI` sql_mode setting includes the `ONLY_FULL_GROUP_BY`
setting. It is generally recommended to leave this setting as-is because it results in deterministic SQL.
However, for some advanced cases, the sql_mode can be configured on the database connection via the configuration API (see `MySQLDatabase::$sql_mode` for more details.)

### MySQL/MariaDB int width in schema

MySQL 8.0.17 stopped reporting the width attribute for integers while MariaDB did not change its behaviour.
This results in constant rebuilding of the schema when MySQLSchemaManager expects a field to look like e.g.
`INT(8)` and MySQL server reports it simply as `INT`. MySQLSchemaManager attempts to detect the MySQL
server implementation and act accordingly. In cases when auto-detection fails, you can force the desired behaviour like this:

```yml
SilverStripe\ORM\Connect\MySQLSchemaManager:
    schema_use_int_width: true # or false when INT widths should be ignored
```

## Webserver configuration

### Overview

Silverstripe CMS needs to handle a variety of HTTP requests, and relies on the hosting environment to be configured securely
to enforce restrictions. There are secure defaults in place for Apache, but you should be aware of the configuration
regardless of your webserver setup.

### Public webroot

The webroot of your webserver should be configured to the `public/` subfolder. Anything in the `public/` directory should
be considered publicly accessible unless there are explicit webserver rules to prevent access (such as for [protected assets](#secure-assets)).

### Filesystem permissions

During runtime, Silverstripe CMS needs read access for the webserver user to your webroot. It
also needs write access for the webserver user to the following locations:

- `public/assets/`: Used by the CMS and other logic to [store uploads](/developer_guides/files/file_storage)
- `TEMP_PATH`: Temporary file storage used for the default filesystem-based cache adapters in
  [Manifests](/developer_guides/execution_pipeline/manifests), [Object Caching](/developer_guides/performance/caching)
  and [Partial Template Caching](/developer_guides/templates/partial_template_caching).
  See [Environment Management](/getting_started/environment_management).

If you aren't explicitly [packaging](#building-packaging-deployment)
your Silverstripe CMS project during your deployment process, additional write access may be required to generate supporting
files on the fly. This is not recommended, because it can lead to extended execution times as well as cause
inconsistencies between multiple server environments when manifest and cache storage isn't shared between servers.

Note that permissions may be required for other directories for specific functionality - for example if you use the
[i18nTextCollector](api:SilverStripe\i18n\TextCollection\i18nTextCollector) you will need to provide write access to the
relevant i18n `lang` directories.

### Assets

Silverstripe CMS allows CMS authors to upload files into the `public/assets/` folder, which should be served by your
webserver. **No PHP execution should be allowed in this folder**. This is configured for Apache by default
via `public/assets/.htaccess`. The file is generated dynamically when building the database.

Additionally, access is whitelisted by file extension through a dynamically generated whitelist based on
the `File.allowed_extensions` setting
(see [File Security](/developer_guides/files/file_security#file-types)). This whitelist uses the same defaults
configured through file upload through Silverstripe CMS, so is considered a second line of defence. If you do not
use apache to serve your website you should find out what equivalent configuration you need to set for your webserver.

### Secure assets {#secure-assets}

Files can be kept in draft stage, and access restricted to certain user groups. These files are stored in a
special `.protected/` folder (defaulting to `public/assets/.protected/`).
**Requests to files in this folder should be denied by your webserver**.

Requests to files in the `.protected/` folder are routed to PHP by default when using Apache,
through `public/assets/.htaccess`. If you are using another webserver, please follow our guides to ensure a secure
setup. See the [other webservers](#other-webservers) section and
[Developer Guides: File Security](/developer_guides/files/file_security) for details.

For additional security, we recommend moving the `.protected/` folder out of `public/assets/`. This removes the
possibility of a misconfigured webserver accidentally exposing these files under URL paths, and forces read access via
PHP.

This can be configured via [.env](/getting_started/environment_management) variable, relative to the `public/`
directory.

```bash
# This will be inside your project root, along-side the public/ directory
SS_PROTECTED_ASSETS_PATH="../.protected/"
```

The resulting folder structure will look as follows:

```text
.protected/
  <hash>/my-protected-file.txt
public/
  index.php
  assets/
    my-public-file.txt
vendor/
app/
```

Don't forget to include this additional folder in any syncing and backup processes!

### Building, packaging and deployment {#building-packaging-deployment}

It is common to build a Silverstripe CMS application into a package on one environment (e.g. a CI server), and then deploy
the package to a (separate) webserver environment(s). This approach relies on all auto-generated files required by
Silverstripe CMS to be included in the package, or generated on the fly on each webserver environment.

The easiest way to ensure this is to commit auto generated files to source control. If those changes are considered too
noisy, here's some pointers for auto-generated files to trigger and include in a deployment package:

- `public/_resources/`: Frontend resources copied from the (inaccessible) `vendor/` folder
  via [silverstripe/vendor-plugin](https://github.com/silverstripe/vendor-plugin).
  See [Templates: Requirements](/developer_guides/templates/requirements#exposing-resources-webroot).
- Various recipes create default files in `app/` and `public/` on `composer install`
  and `composer update` via
  [silverstripe/recipe-plugin](https://github.com/silverstripe/recipe-plugin).

### Web worker concurrency

It's generally a good idea to run multiple workers to serve multiple HTTP requests to Silverstripe CMS concurrently. The
exact number depends on your website needs. The CMS attempts to request multiple views concurrently. It also
routes [protected and draft files](/developer_guides/files/file_security)
through Silverstripe CMS. This can increase your concurrency requirements, e.g. when authors batch upload and view dozens of
draft files in the CMS.

When allowing upload of large files through the CMS (through PHP settings), these files might be used
as [protected and draft files](/developer_guides/files/file_security). Files in this state get served by Silverstripe CMS
rather than your webserver. Since the framework uses [PHP streams](https://www.php.net/manual/en/ref.stream.php), this
allows serving of files larger than your PHP memory limit. Please be aware that streaming operations don't count towards
PHP's [max_execution_time](https://www.php.net/manual/en/function.set-time-limit.php), which can risk exhaustion of web
worker pools for long-running downloads.

### URL rewriting

Silverstripe CMS expects URL paths to be rewritten to `public/index.php`. For Apache, this is preconfigured
through `.htaccess` files, and requires using the `mod_rewrite` module. By default, the relevant configuration files are located
in `public/.htaccess` and `public/assets/.htaccess`.

### HTTP headers

Silverstripe CMS can add HTTP headers to responses it handles directly. These headers are often sensitive, for example
preventing HTTP caching for responses displaying data based on user sessions, or when serving protected assets. You need
to ensure those headers are kept in place in your webserver. For example, Apache allows this
through `Header setifempty` (see [docs](https://httpd.apache.org/docs/current/mod/mod_headers.html#header)).
See [Developer Guide: Performance](/developer_guides/performance/)
and [Developer Guides: File Security](/developer_guides/files/file_security) for more details.

Silverstripe CMS relies on the `Host` header to construct URLs such as "reset password" links, so you'll need to ensure that
the systems hosting it only allow valid values for this header.
See [Developer Guide: Security - Request hostname forgery](/developer_guides/security/secure_coding#request-hostname-forgery).

### CDNs and other reverse proxies

If your Silverstripe CMS site is hosted behind multiple HTTP layers, you're in charge of controlling which forwarded headers
are considered valid, and which IPs can set them.
See [Developer Guide: Security - Request hostname forgery](/developer_guides/security/secure_coding#request-hostname-forgery).

### Symlinks

Silverstripe CMS is a modular system, with modules installed and updated via the `composer` PHP dependency manager. These
are usually stored in `vendor/`, outside of the `public/` webroot. Since many modules rely on serving frontend assets
such as CSS files or images, these are mapped over to the `public/_resources/` folder automatically. If the filesystem
supports it, this is achieved through symlinks. Depending on your hosting and deployment mechanisms, you may need to
configure the plugin to copy files instead.
See [silverstripe/vendor-plugin](https://github.com/silverstripe/vendor-plugin) for details.

### Caches

Silverstripe CMS relies on various [caches](/developer_guides/performance/caching/)
to achieve performant responses. By default, those caches are stored in a temporary filesystem folder, and are not
shared between multiple server instances.

No in-memory cache is used by default. To improve performance, we recommend [configuring an in-memory cache](/developer_guides/performance/caching/#adapters) such as Redis or Memcached.

While cache objects can expire, when using filesystem caching the files are not actively pruned. For long-lived server
instances, this can become a capacity issue over time - see
[workaround](https://github.com/silverstripe/silverstripe-framework/issues/6678).

### Error pages

The default installation
includes [silverstripe/errorpage](https://github.com/silverstripe/silverstripe-errorpage/), which generates
static error pages that bypass PHP execution when those pages are published in the CMS. Once published, the static files
are located in `public/assets/error-404.html` and `public/assets/error-500.html`. The default `public/.htaccess` file is
configured to have Apache serve those pages based on their HTTP status code.

### Other webservers (Nginx, IIS, Lighttpd) {#other-webservers}

Serving through webservers other than Apache requires more manual configuration, since the defaults configured
through `.htaccess` don't apply. Please apply the considerations above to your webserver to ensure a secure hosting
environment. In particular, configure protected assets correctly to avoid exposing draft or protected files uploaded
through the CMS.

There are various community supported installation instructions for different environments. Nginx is a popular choice,
see [Nginx webserver configuration](https://forum.silverstripe.org/t/nginx-webserver-configuration/2246).

Silverstripe CMS is known to work with Microsoft IIS, and generates `web.config` files by default
(see [Microsoft IIS and SQL Server configuration](https://forum.silverstripe.org/t/microsoft-iis-webserver-and-sql-server-support/2245)).

Additionally, there are community supported guides for installing Silverstripe CMS on various environments:

- [Hosting via Bitnami](https://bitnami.com/stack/silverstripe/virtual-machine): In the cloud or as a locally hosted
  virtual machine
- [Vagrant/Virtualbox with CentOS](https://forum.silverstripe.org/t/installing-via-vagrant-virtualbox-with-centos/2248)
- [macOS with Homebrew](https://forum.silverstripe.org/t/installing-on-osx-with-homebrew/2247)
- [macOS with MAMP](https://forum.silverstripe.org/t/installing-on-osx-with-mamp/2249)
- [Windows with WAMP](https://forum.silverstripe.org/t/installing-on-windows-via-wamp/2250)
- [Vagrant with silverstripe-australia/vagrant-environment](https://github.com/silverstripe-australia/vagrant-environment)
- [Vagrant with BetterBrief/vagrant-skeleton](https://github.com/BetterBrief/vagrant-skeleton)

### Email

Silverstripe CMS uses [symfony/mailer](https://github.com/symfony/mailer) to send email messages. [silverstripe/framework](https://github.com/silverstripe/silverstripe-framework) is configured to use a `sendmail` binary (usually found in `/usr/sbin/sendmail`). Alternatively [email can be configured](/developer_guides/email/) to use SMTP or other mail transports instead of sendmail.

You *must* ensure emails are being sent from your *production* environment. You can do this by testing that the ***Lost password*** form available at `/Security/lostpassword` sends an email to your inbox, or with the following code snippet that can be run via a `SilverStripe\Dev\BuildTask`:

```php
use SilverStripe\Control\Email\Email;

$email = Email::create('no-reply@mydomain.com', 'myuser@gmail.com', 'My test subject', 'My email body text');
$email->send();
```

Using the code snippet above also tests that the ability to set the "from" address is working correctly.

See the [email section](/developer_guides/email) for further details, including how to set the administrator "from" email address, change the `sendmail` binary location, and how to use SMTP or other mail transports instead of sendmail.

## PHP requirements for older Silverstripe CMS releases {#php-support}

Silverstripe CMS's PHP support has changed over time and if you are looking to upgrade PHP on your Silverstripe CMS site, this
table may be of use:

| Silverstripe CMS Version | PHP Version |
| ------------------------ | ----------- |
| 6.0 +                    | 8.3 - 8.4   |
| 5.2 +                    | 8.1 - 8.3   |
| 5.0 - 5.1                | 8.1 - 8.2   |

From Silverstripe CMS 5 onwards, the [Silverstripe CMS major release policy](/project_governance/major_release_policy#php-support-commitments) guides which PHP versions are supported by which Silverstripe CMS release.

## CMS browser requirements

Silverstripe CMS uses [browserslist](https://browsersl.ist) default settings to determine which browsers are supported. Note that this only applies for the CMS itself - you can support whatever browsers you want to in the front end of your website.

These settings ensure support for the latest 2 versions of major browsers, plus all versions of those browsers with at least 0.5% worldwide market share, plus the [Firefox Extended Support Release](https://support.mozilla.org/en-US/kb/choosing-firefox-update-channel). They explicitly exclude browser versions which have reached end-of-life.

You can use [browserlist's "check compatible browsers" tool](https://browsersl.ist/#q=defaults) to see specifically which versions of which browsers are supported by these settings.

Silverstripe CMS works well across Windows, Linux, and Mac operating systems - though it is worth noting that most of our development and testing is done in Linux environments.

## End user requirements

Silverstripe CMS is designed to make excellent, standards-compliant websites that are compatible with a wide range of
industry standard browsers and operating systems. A competent developer is able to produce websites that meet W3C
guidelines for HTML, CSS, JavaScript, and accessibility, in addition to meeting specific guide lines, such as
e-government requirements.
