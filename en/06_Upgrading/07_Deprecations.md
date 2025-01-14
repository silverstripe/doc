---
title: Deprecations
---

# Deprecations

You'll need to go through your codebase to remove references to deprecated APIs and update your project logic.

Major releases of Silverstripe CMS introduce many API changes. They include a deprecation message that will either tell you:

- to simply use a different yet equivalent API instead, or
- the API in question has no replacement. This is usually for API that was mostly irrelevant.

## Enabling deprecation warnings

To enable deprecation warnings, set the `SS_DEPRECATION_ENABLED` environment variable in your project's `.env` file.

> [!NOTE]
> If the `SS_DEPRECATION_ENABLED` environment variable is set, this takes precedence over use of the `Deprecation::enable()` static method.

```bash
SS_DEPRECATION_ENABLED=true
```

Alternatively, add the following line to your project's `app/_config.php`.

```php
Deprecation::enable();
```

> [!NOTE]
> Deprecation warnings will only ever show if your `SS_ENVIRONMENT_TYPE` is set to `dev`.

Once you have resolved all of the deprecation warnings you can, it is recommended to turn off deprecation warnings again.

Not all API that gets deprecated will have an equivalent replacement API in that same major version; some of the API is only available from the next major release. In these cases, you'll need to upgrade to the new major version before you can access the replacement API.

Some code that has been deprecated with no immediate replacement will not emit deprecation notices by default. If you wish to also see notices for deprecated code with no immediate replacement, add the following line to you project's `app/_config.php` file:

```php
Deprecation::enable(true);
```

By default, deprecation warnings will not be emitted if the deprecated code was called by code in [supported modules](/project_governance/supported_modules/). This is because these warnings are simply not relevant for most developers. If you wish to see deprecation warnings for code called by core code, add the following line to your project's `app/_config.php` file:

```php
Deprecation::setShowNoticesCalledFromSupportedCode(true);
```

## How to view deprecation warnings

By default, deprecation warnings will be emitted to the error logger, and will be output at the end of CLI responses. They will not be included in HTTP responses by default.

### Viewing deprecation warnings in the logs

Deprecation warnings are output to the same error logger as all other warnings and errors. You will need to make sure you have a logging handler attached to the default `Psr\Log\LoggerInterface` singleton. For example, to log to a file you can add this to your YAML configuration:

```yml
SilverStripe\Core\Injector\Injector:
  ErrorLogFileHandler:
    class: Monolog\Handler\StreamHandler
    constructor:
      - "/var/www/silverstripe.log"
      - "warning" # warning is the level deprecation warnings are logged as
  Psr\Log\LoggerInterface:
    calls:
      ErrorLogFileHandler: [ pushHandler, [ '%$ErrorLogFileHandler' ] ]
```

> [!WARNING]
> The log file path must be an absolute file path, as relative paths may behave differently between CLI and HTTP requests. If you want to use a *relative* path, you can use the `SS_ERROR_LOG` environment variable to declare a file path that is relative to your project root:
>
> ```bash
> SS_ERROR_LOG="./silverstripe.log"
> ```
>
> You don't need any of the YAML configuration above if you are using the `SS_ERROR_LOG` environment variable - but you can use a combination of the environment variable and YAML configuration if you want to configure multiple error log files.
>
> You will also need to make sure the user running the PHP process has write access to the log file, wherever you choose to put it.

See [Configuring error logging](/developer_guides/debugging/error_handling/#configuring-error-logging) to learn about other ways you can handle error logs.

### Deprecation warnings in your browser

Deprecation warnings won't be output to HTTP responses by default because it can be difficult to parse and collate information this way, and deprecation warnings in XHR/AJAX responses can result in unexpected behaviour in some situations. That said, you can choose to enable this output.

Deprecation warnings can be set to output in HTTP responses by setting `SS_DEPRECATION_SHOW_HTTP` to a truthy value in your .env file.

> [!NOTE]
> If the `SS_DEPRECATION_SHOW_HTTP` environment variable is set, this takes precedence over use of the `Deprecation::setShouldShowForHttp()` static method.

```bash
SS_DEPRECATION_SHOW_HTTP=true
```

Alternatively, add the following line to your project's `app/_config.php`.

```php
Deprecation::setShouldShowForHttp(true);
```

Note that the output for this won't be very easy to read. You might prefer instead to install [lekoala/silverstripe-debugbar](https://github.com/lekoala/silverstripe-debugbar) as a dev dependency. Deprecation warnings will be logged in the "messages" tab of the debugbar.

> [!WARNING]
> The debugbar will *not* show you any deprecation warnings that are triggered from XHR/AJAX requests or which are triggered after the middleware has finished generating the debugbar for the response.

## Deprecation warnings in the CLI

Deprecation warnings are output for CLI responses by default (assuming they're enabled in general). The warnings are always output at the *end* of the request, so you don't have to go looking through the output for them. You might want to disable outputting these warnings in CLI responses if, for example, you need to validate the output via code and don't want to add special cases for deprecation warnings.

You can suppress deprecation warnings from CLI output by setting `SS_DEPRECATION_SHOW_CLI` to a falsy value in your .env file.

> [!NOTE]
> If the `SS_DEPRECATION_SHOW_CLI` environment variable is set, this takes precedence over use of the `Deprecation::SS_DEPRECATION_SHOW_CLI()` static method.

```bash
SS_DEPRECATION_SHOW_CLI=false
```

Alternatively, add the following line to your project's `app/_config.php`.

```php
Deprecation::setShouldShowForCli(false);
```
