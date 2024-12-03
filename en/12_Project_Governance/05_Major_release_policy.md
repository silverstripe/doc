---
title: Major Release Policy
summary: Outline the major release lifecycle and support commitment for Silverstripe CMS
icon: code-branch
---

# Major release policy

## Scope of the policy

This policy applies to all [Silverstripe CMS commercially supported modules](/project_governance/supported_modules/).

Community modules are not covered by this policy. Modules in the `silverstripe` GitHub organisation that are not commercially supported are updated on a best effort basis.

> [!NOTE]
> Refer to our [definition of public API](/project_governance/public_api/).

## General approach to major releases

Silverstripe CMS aims to deliver regular major releases at predefined intervals with a clear support timeline. The key objective of this policy is to allow Silverstripe CMS project owners to plan major upgrades ahead of time.

The main function of major releases is to ship *breaking changes* that can not be shipped in minor releases. These changes are primarily aimed at making sure the major releases can be supported for the full duration of its lifecycle.

The lifecycle for a Silverstripe CMS major release line is:

- pre-release (undefined duration)
- active development (2 years)
- bug and security fixes (1 year)
- security fixes only (1 year).

Most of the changes shipped in a major release will:

- upgrade third party dependencies
- remove deprecated public API and clean up ambiguous API
- implement architectural changes that can not reasonably be introduced in a minor release.

At launch, major releases may contain new features not present in the previous major release line. However, this is a secondary concern. New features will usually ship in minor releases.

## Upgrading to a new major release

Major releases are not backward compatible. Most projects will require at least some development work to upgrade to a new major release.

Silverstripe CMS provides a [clear upgrade path](/upgrading) to new major releases. This includes clear documentation on what breaking changes have been made to each area of Silverstripe CMS.

## Major release lifecycle

A Silverstripe CMS major release line follows a predefined life cycle. New Silverstripe CMS majors are released in odd years (e.g. 2023, 2025, 2027, etc). At any given time, only two major release lines are officially supported.

### Pre-release

A Silverstripe CMS stable major release is preceded by a beta period of at least three months.

The purpose of the beta period is to:

- allow early adopters to plan their upgrade
- identify bugs in the major release
- help community modules add support for the new major release early on.

Once a new major enters the beta period, it should be compatible with the dependencies intended to ship in the stable release. Substantial breaking changes should be avoided post-beta.

Prior to the beta period, alpha releases are published at the discretion of the development team.

### Stable release

New major releases of Silverstripe CMS are tagged between April and June of odd years.

### Active development

Once a major release is stable, it enters a period of active development. A major release in active development receives:

- regular minor releases that ship new features and public API in a backward compatible way
- regular patches for bugs at all impact levels
- security patches for all vulnerabilities.

A major release line stays in *active development* for approximately two years. *Active development* for a major release line ends when the next major release is tagged stable.

Only one major release line is in *active development* at any given time.

### Bug and security fixes

A Silverstripe CMS major release line enters the *bug and security fixes* phase once the follow major release is tagged stable.

A major release line in the *bug and security fixes* phase receives:

- regular patches for bugs at all impact levels
- security patches for all vulnerabilities.

A major release line in the *bug and security fixes* phase does **not** receive:

- new feature
- new public API
- new minor releases.

A major release line stays in the *bug and security fixes* phase for 1 year.

### Security fixes only

At the end of *bug and security fixes* phase, a Silverstripe CMS major release line transitions to the *security fixes only* phase.

In the *security fixes only* phase, a Silverstripe CMS major release line only receives patches for high impact security issues defined has having a CVSS of 7.0 or above.

### End-of-life

A Silverstripe CMS major release line reaches *end-of-life* once the next major release line exits active development.

End-of-life major releases do not receive updates of any kind including security fixes or bug fixes.

## PHP support commitments

Silverstripe CMS major releases track PHP releases.

The Silverstripe CMS release cycle is built around these assumptions:

- new PHP releases are shipped at the end of November on an annual basis
- PHP releases are in full support for two years with one year of limited support.

At launch, a Silverstripe CMS major release supports all PHP versions in full support. PHP versions in limited support are not supported at launch by new Silverstripe CMS major releases.

Following the initial launch of a Silverstripe CMS major release, the development team aims to add forward compatibility for the next PHP release. e.g: Silverstripe CMS 7 at launch will support PHP 8.5 and PHP 8.6. CMS 6 should receive official support for an eventual PHP 8.5 in early 2026.

Support for end-of-life PHP releases is not dropped within a Silverstripe CMS major release line, unless it's necessary to address vulnerabilities or high impact bugs.

Major release lines not in active development do not receive official support for later PHP releases.

## Dependency management

At launch, Silverstripe CMS major releases aim to have all PHP and JavaScript dependencies on their latest stable versions. For dependencies offering a *Long Term Support* version, the latest LTS of that dependency is preferred.

A list of *fixed dependencies* for each Silverstripe CMS major release line is explicitly defined by the development team. *Fixed dependencies* do not change for the lifetime of a Silverstripe CMS major release line.

Support for newer major releases of fixed dependencies is not added to Silverstripe CMS major release line, even if the currently used version is end-of-life. A Silverstripe CMS major release line only supports one major version of each fixed dependency.

The only exceptions are:

- if the upgrade is necessary to fix a high impact bug or security vulnerability
- if the upgrade is necessary to add support for a new PHP version to a Silverstripe CMS major release in active development.

Non-fixed dependencies may be upgraded to new major releases or swapped out altogether within a Silverstripe CMS release line.

### Symfony support commitments

Symfony is a collection of common PHP libraries Silverstripe CMS uses. At launch, Silverstripe CMS major releases track the latest stable major version of Symfony dependencies. A Silverstripe CMS major release line does not add support for later Symfony major versions.

## Commercially supported module

Commercially supported modules are reviewed with each new major release. At launch, the list of commercially supported modules for that release line is explicitly defined.

New modules may be added to the commercially supported modules list during a Silverstripe CMS major release line's lifespan. Modules are not withdrawn from the commercially supported modules list for a Silverstripe CMS major releases line unless external factors make it impractical to continue support. For example the module requires a discontinued third party service to work as intended.

Modules that are supported for one major release line may not be supported for follow-up release lines. Supported modules can be renamed or their namespaces can be changed when a new Silverstripe CMS major release is published.

## Expected Silverstripe CMS major release timeline

This timeline is provided as an indication only.

![Timeline of planned Silverstripe CMS major releases](../_images/major_release_timeline.jpg)
