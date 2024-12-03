---
title: Issues and Bug Reports
summary: Report bugs or problems with Silverstripe CMS, feature requests or other issues.
icon: bug
---

# Issues and bug reports

## Reporting bugs

> [!CAUTION]
> If you think you've found a security issue, please use [the specific process](#reporting-security-issues) for those. Do *not* raise a security issue in GitHub.

If you have discovered a bug in Silverstripe CMS, we'd be glad to hear about it -
well written bug reports can be half of the solution already!

Silverstripe CMS uses [GitHub](https://github.com/) to manage bug reports. If you
want to report a bug, you will need to [create a GitHub account](https://docs.github.com/en/get-started/onboarding/getting-started-with-your-github-account)
and log in.

> [!WARNING]
> We don't provide support through GitHub issues. If you're having trouble using or developing with Silverstripe CMS but you don't think your problem is a bug, please ask for assistance in our [community channels](https://www.silverstripe.org/community).

### Before submitting a bug

- Ask for assistance in our [community channels](https://www.silverstripe.org/community) if you're unsure if it's really a bug.
- Search for similar, existing issues.
   You can [list all issues across modules](https://elvis.silverstripe.org/),
   then add your search phrase at the start of the existing search filters.
- Is this a security issue? Please follow our [security reporting guidelines](#reporting-security-issues) below.
- Try to reproduce your issue on a [clean installation](/getting_started/composer#create-a-new-site) to rule out bugs in your own code.

### If the issue does look like a new bug

- Create an issue on the right module repository in GitHub
  - If you are unsure, [create an issue](https://github.com/silverstripe/silverstripe-framework/issues/new) on the the "framework" repository.
  - Note that [documentation issues](https://github.com/silverstripe/developer-docs/issues) are tracked in the "developer-docs" repository.
- Describe the steps required to reproduce your issue, and the expected outcome. Example code, screenshots, and videos can help here.
   Be as clear as you can, but don't miss any steps out. Simply saying "create a page" is less useful than guiding us through the steps you're taking to create a page, for example.
- If the bug is too complex to reproduce with some short code samples, please reproduce it in a public repository and provide a link to the repository along with steps for setting up and reproducing the bug using that repository.
   A repository like this should only contain code that is required to set up and reproduce the bug.
- Describe your environment in as much detail as possible. Include the versions of relevant modules, the PHP version, webserver, and operating system used to run the project, the browser(s) you see the issue in, etc.
- If part of the bug includes an error or exception, please provide a full stack trace. Be wary that stack traces may contain sensitive information, and if that is the case, be sure to redact them prior to posting your stack trace.
- You are strongly encouraged to [submit a pull request](/contributing/code/#step-by-step-from-forking-to-sending-the-pull-request) which fixes the issue. Bug reports which are accompanied with a pull request are a lot more likely to be resolved quickly.

Lastly, don't get your hopes up too high. Unless your issue is a blocker
affecting a large number of users, don't expect Silverstripe developers to jump
onto it right way. Your issue is a starting point where others with the same
problem can collaborate with you to develop a fix. If this bug is a blocker
for you, then [submitting a pull request](/contributing/code/#step-by-step-from-forking-to-sending-the-pull-request)
is the best way to ensure it gets fixed.

## Feature requests

> [!WARNING]
> Please don't file feature requests as GitHub issues. If there's a new feature
> you'd like to see in Silverstripe CMS, you either need to write it yourself (and
> [submit a pull request](/contributing/code/#step-by-step-from-forking-to-sending-the-pull-request)) or convince somebody else to
> write it for you. Any "wishlist" type issues without code attached can be
> expected to be closed as soon as they're reviewed.

In order to gain interest and feedback in your feature, we encourage you to
present it to the community through the [community channels](https://www.silverstripe.org/community).

## Reporting security issues

> [!WARNING]
> If you think a bug may have security implications, do not create a GitHub issue for it. This may lead to a zero-day vulnerability.

Report potential security issues to [security@silverstripe.org](mailto:security@silverstripe.org). Emails sent to that address are
forwarded to a private mailing list and kick off a specific security process.

If you have any doubts or are unsure whether the bug you've found has security implications or not, please err on the side of caution
and email us about it.

Review our [Managing Security Issues](managing_security_issues) process to understand what happens once a vulnerability is reported.

Silverstripe CMS aims to ship security patches at pre-defined intervals when those issues are not actively exploited in the wild.
Review the [Security patch windows](../Project_Governance/Minor_release_policy#security-patch-windows) section of our minor release policy for more details.

Silverstripe CMS does not operate a *bug bounty* program.

## GitHub labels {#labels}

The current GitHub labels are grouped into five sections:

1. *Impact* - What impact does this issue have, does it break a feature completely, is it just a side effect or is it trivial and not a big problem (but a bit annoying), etc. Impact is evaluated in the context of the CMS as a whole, rather than against the individual module the issue is raised on.
1. *Complexity* - What level of technical proficiency is required to address this issue?
1. *Type* - The type of solution required to address this issue
1. *Affects* - The major release line this issue is relevant to - for instance `silverstripe/admin 3.0.0` is on the CMS 6 major release line, so use `affects/v6`. Do not add this label for unreleased major release lines, instead use `type/api-break`.
1. *RFC* - The issue is a request-for-comment

| Label | Purpose |
| ----- | ------- |
| impact/critical | Website breaking issue with no workarounds. Reserved only for bugs. Bugfix's will target all supported minor release lines. |
| impact/high | Affects a major usage flow. Broken functionality with no obvious workarounds available, or an enhancement that provides a clear benefit to users |
| impact/medium | When affecting a major usage flow, for bugs there is a workaround available and for enhancements there would be a reasonable benefit to users. For a less common usage flow there is broken functionality and for enhancements there is a clear benefit to users. |
| impact/low | A nuisance but doesn't break any functionality (typos, etc). For enhancements there would only be a limited benefit to users. |
| complexity/low | Someone with limited Silverstripe CMS experience could resolve |
| complexity/medium | Someone with a good understanding of Silverstripe CMS could resolve |
| complexity/high | Only an expert with Silverstripe CMS could resolve |
| type/bug | Does not function as intended, or is inadequate for the purpose it was created for |
| type/enhancement | New feature or improvement for either users or developers |
| type/api-break | An breaking API change requiring a new major release |
| type/ux | Impact on the CMS user interface |
| type/docs | A docs change |
| type/userhelp | A userhelp documentation change |
| type/i18n | Relates to localisation/translations |
| type/other | Any issue that does not is not covered by another type label e.g. general maintainence |
| affects/* | Issue has been observed on a specific CMS release line |
| rfc/draft | [RFC](/project_governance/request_for_comment) under discussion |
| rfc/accepted | [RFC](/project_governance/request_for_comment) where agreement has been reached |
