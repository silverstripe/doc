---
title: Email
summary: Send HTML and plain text email from your Silverstripe CMS application.
icon: envelope-open
---

# Email

Creating and sending email in Silverstripe CMS is done using the [Email](api:SilverStripe\Control\Email\Email) class. This document covers how to create an `Email` instance and customise it with an HTML template.

## Configuration

Silverstripe CMS provides an API over the top of [symfony/mailer](https://symfony.com/doc/current/mailer.html) which comes with an extensive list of "transports" for sending mail via different services such as SMTP, Gmail, and Amazon SES.

Email configuration is done using Symfony's [DSN](https://symfony.com/doc/current/mailer.html#transport-setup) configuration string which is used to select which transport is used and any required configuration such as username and password. In Silverstripe, this is done with either an environment variable or yml configuration.

The `Sendmail` transport is the most common one and is used by default in Silverstripe. The `sendmail` binary is widely available across most Linux/Unix servers. By default the sendmail command used is `/usr/sbin/sendmail -bs`, but this [can be configured](#dsn-sendmail) as part of the `DSN`.

Alternatively you can provide a different `DSN` to select any of the Transport classes provided natively by `symfony/mailer` or other compatible third-party transports. For more information and to see what other transports are available see the [symfony/mailer transport types](https://symfony.com/doc/current/mailer.html#using-a-3rd-party-transport).

> [!TIP]
> The format for the DSN is exactly as defined in the symfony docs linked above. Some common examples are listed below.

To set the DSN string in an environment variable (recommended):

```bash
# .env
MAILER_DSN="<my-dsn>"
```

To set the DSN string in project yml:

```yml
# app/_config/mailer-project.yml
---
Name: mailer-project
After: 'mailer'
---
SilverStripe\Core\Injector\Injector:
  Symfony\Component\Mailer\Transport\TransportInterface:
    constructor:
      dsn: '<my-dsn>'
```

The configuration priority order is as follows, from highest to lowest:

- The `MAILER_DSN` environment variable
- Project yml containing `After: 'mailer'`
- The default DSN of `sendmail://default` which will use `/usr/sbin/sendmail -bs`

### Common DSN strings

#### To configure SMTP {#dsn-smtp}

```bash
# .env
MAILER_DSN="smtp://user:pass@smtp.example.com:1234"
```

#### To configure a different sendmail binary and command {#dsn-sendmail}

```bash
# .env
MAILER_DSN="sendmail://default?command=/path/to/mysendmailbinary%20-t"
```

#### To suppress all emails {#dsn-null}

```bash
# .env
MAILER_DSN="null://default"
```

Read more about other available DSN strings in [the symfony documentation](https://symfony.com/doc/current/mailer.html#using-a-3rd-party-transport)

### Testing that email works

You *must* ensure emails are being sent from your *production* environment. You can do this by testing that the
***Lost password*** form available at `/Security/lostpassword` sends an email to your inbox, or with the following code snippet that can be run via a `SilverStripe\Dev\BuildTask`:

```php
use SilverStripe\Control\Email\Email;

$email = Email::create($from, $to, $subject, $body);
$email->send();
```

Using the code snippet above also tests that the ability to set the "from" address is working correctly.

## Usage

### Sending plain text only

```php
use SilverStripe\Control\Email\Email;

$email = Email::create($from, $to, $subject, $body);
$email->sendPlain();
```

### Sending combined HTML and plain text

By default, emails are sent in both HTML and Plaintext format. A plaintext representation is automatically generated
from the system by stripping HTML markup, or transforming it where possible (e.g. `<strong>text</strong>` is converted
to `*text*`).

You can also specify plain text and HTML content separately if you don't want the plain text to be automatically generated from HTML

```php
use SilverStripe\Control\Email\Email;

$email = Email::create('from@mydomain.com', 'to@example.com', 'My subject');
$email->html('<p>My HTML email content</p>');
$email->text('My plain text email content');
$email->send();
```

> [!NOTE]
> The default HTML template for emails is `vendor/silverstripe/framework/templates/SilverStripe/Control/Email/Email`.
> To customise this template, first copy it to a `<project-root>/themes/<my-theme>/SilverStripe/Control/Email/Email` template file. Alternatively, copy it to a different location and use `setHTMLTemplate` when you create the
> `Email` instance. Note - by default the `$EmailContent` variable will escape HTML tags for security reasons. If you feel confident allowing this variable to be rendered as HTML, then update your custom email template to `$EmailContent.RAW`

### Templates

HTML emails can use custom templates using the same template language as your website template. You can also pass the
email object additional information using the `setData` and `addData` methods.

> [!NOTE]
> Calling `setData()` or `addData()` once or more will cause the email to be rendered using Silverstripe templates. This will override any email content set directly via methods such as `setBody()`, `html()`, or `text()`.

```ss
<%-- app/templates/Email/MyCustomEmail.ss --%>
<h1>Hi $Member.FirstName</h1>
<p>You can go to $Link.</p>
```

The PHP Logic..

```php
use SilverStripe\Control\Email\Email;
use SilverStripe\Security\Security;

$email = Email::create()
    ->setHTMLTemplate('Email\\MyCustomEmail')
    ->setData([
        'Member' => Security::getCurrentUser(),
        'Link' => $link,
    ])
    ->from($from)
    ->to($to)
    ->subject($subject);

$email->send();
```

> [!CAUTION]
> As we've added a new template file (`MyCustomEmail`) make sure you clear the Silverstripe CMS cache for your changes to
> take affect.

#### Custom plain templates

By default Silverstripe CMS will generate a plain text representation of the email from the HTML body. However if you'd like
to specify your own own plaintext version/template you can use `$email->setPlainTemplate()` to render a custom view of
the plain email:

```php
use SilverStripe\Control\Email\Email;

$email = Email::create($from, $to, $subject, $body);
$email->setPlainTemplate('MyPlanTemplate');
$email->send();
```

## Administrator emails

You can set the default sender address of emails through the `Email.admin_email` [configuration setting](/developer_guides/configuration).

```yml
# app/_config/app.yml
SilverStripe\Control\Email\Email:
  admin_email: support@example.com
```

To add a display name, set `admin_email` as follow.

```yml
SilverStripe\Control\Email\Email:
  admin_email:
    support@example.com: 'Support team'
```

## Adding display names and sending to multiple recipients

```php
use SilverStripe\Control\Email\Email;

$from = [
  'from@mysite.exmaple.com' => 'Friendly business',
];
$to = [
  'person.a@customer.example.com' => 'Person A',
  'person.b@customer.example.com' => 'Person B',
  'person.c@customer.example.com',
]
$email = Email::create($from, $to, $subject, $body);
```

> [!CAUTION]
> Remember, setting a `from` address that doesn't come from your domain (such as the users email) will likely see your
> email marked as spam. If you want to send from another address think about using the `setReplyTo` method.

You will also have to remove the `SS_SEND_ALL_EMAILS_FROM` environment variable if it is present.

If you need greater control over this email address, for instance if are running the subsites modules, you can implement the `SilverStripe\Control\Email\Email::updateDefaultFrom()` extension hook.

## Redirecting emails

There are several other [configuration settings](/developer_guides/configuration) to manipulate the email server.

- `SilverStripe\Control\Email\Email.send_all_emails_to` will redirect all emails sent to the given address.
All recipients will be removed (including CC and BCC addresses). This is useful for testing and staging servers where
you do not wish to send emails out. For debugging the original addresses are added as `X-Original-*` headers on the email.
- `SilverStripe\Control\Email\Email.cc_all_emails_to` and `SilverStripe\Control\Email\Email.bcc_all_emails_to` will add
an additional recipient in the BCC / CC header. These are good for monitoring system-generated correspondence on the
live systems.

Configuration of those properties looks like the following:

```php
// app/_config.php
use SilverStripe\Control\Director;
use SilverStripe\Control\Email\Email;
use SilverStripe\Core\Config\Config;

if (Director::isLive()) {
    Config::modify()->set(Email::class, 'bcc_all_emails_to', 'client@example.com');
} else {
    Config::modify()->set(Email::class, 'send_all_emails_to', 'developer@example.com');
}
```

### Setting custom "Reply To" email address

For email messages that should have an email address which is replied to that actually differs from the original "from"
email, do the following. This is encouraged especially when the domain responsible for sending the message isn't
necessarily the same which should be used for return correspondence and should help prevent your message from being
marked as spam.

```php
use SilverStripe\Control\Email\Email;

$email = Email::create($from, $to, $subject, $body);
$email->replyTo('reply@example.com');
```

## Catching email send failure exceptions

If you wish to handle email send failures then you can wrap `$email->send()` with a try/catch block that catches the Symfony Mailer `TransportExceptionInterface`.

> [!TIP]
> You might get a Symfony Mailer `RfcComplianceException` when instantiating the `Email` object if the email address you're trying to send to or from is invalid.
> In some cases you'll want to catch and handle that exception as well.

```php
use SilverStripe\Control\Email\Email;
use Symfony\Component\Mailer\Exception\TransportExceptionInterface;

$email = Email::create('from@example.com', 'to@example.com', 'My subject');
$email->text('My plain text email content');
try {
    $email->send();
} catch (TransportExceptionInterface $e) {
    // handle exception
}
```

For more information, refer to [handling sending failures](https://symfony.com/doc/current/mailer.html#handling-sending-failures) in the symfony/mailer docs.

## Advanced customisation

Silverstripe Email is built on top of [symfony/mailer](https://github.com/symfony/mailer). For advanced customisation information refer to the [symfony/mailer docs](https://symfony.com/doc/current/mailer.html)

- [Email](api:SilverStripe\Control\Email\Email)
