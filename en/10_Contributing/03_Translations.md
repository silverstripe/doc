---
title: Contributing Translations
summary: Translate interface components like button labels into multiple languages.
icon: globe
---

# Contributing translations

The content for UI elements (button labels, field titles, etc) and instruction texts shown in the CMS and elsewhere is
stored in YAML and JavaScript files (see [i18n](/developer_guides/i18n)). These get
uploaded to [transifex](https://transifex.com) to be edited online.

Silverstripe CMS is already translated in over 60 languages, and we're
relying on native speakers to keep these up to date, and of course add new languages.

Even if a specific language is already translated, we can use helping hands
in reviewing and updating translations. It is perfectly fine if you only have time for a partial translation or quick
review work - our system accommodates many people collaborating on the same language.

Please [register a free translator account](https://app.transifex.com/signup/open-source/) to get started, even if you just feel like fixing up a few sentences.

## The online translation tool

We provide a GUI for translations through [transifex.com](https://app.transifex.com/silverstripe/). If you don't have an account yet,
please follow the links there to sign up.  Select a project from the
[list of translatable modules](https://explore.transifex.com/silverstripe/) and start translating online!

If you need help learning how to edit translations in transifex, check out [transifex's documentation](https://help.transifex.com/).

## FAQ

### How do I translate a module not listed on Transifex?

If a core or supported module is not listed on Transifex, usually that means it has no strings which *can* be translated.
If you find a core or supported module which has strings that can be (or should be able to be) translated, please
[raise an issue on GitHub](./issues_and_bugs) for that module.

### How do I translate substituted strings? (Such as `{my-variable}`)

You don't have to - if the english string reads 'Hello {my-variable}', your german translation would be 'Hallo {my-variable}'. Strings
included in `{}` are automatically replaced by Silverstripe CMS with dynamic content.

### Do I need to convert special characters (such as HTML entities)?

Special characters (such as german umlauts) need to be entered in their native form. Please don't use HTML-entities
(use "ä" instead of "`&auml;`"). Silverstripe stores and renders most strings in UTF8 (Unicode) format.

### How can I check out my translation in the interface?

Currently translated entities are not directly factored into code (for security reasons and release/review-control), so
you can't see them straight away.

If you really want to check your translation out in context before it has been merged into the codebase, you can follow
the instructions in [i18n](/developer_guides/i18n) to add those translations directly to your Silverstripe CMS project.

### Can I change a translation just for one Silverstripe CMS version?

While we version control our translation files like all other source code, the online translation tool doesn't have the
same capabilities. A translated string (as identified by its unique "entity name") is assumed to work well in all
releases. If the interface changes in a non-trivial fashion, the new translations required should have new identifiers
as well.

### How do I change my interface language?

Once you've logged into the CMS, you should see your name near the top left. You can click this to edit
your profile. You can then set the "interface language" from a dropdown.

### I've found a piece of untranslatable text

It is entirely possible that we missed certain strings in preparing Silverstripe for translation-support. If you're
technically minded, please read [i18n](/developer_guides/i18n) on how to make it translatable and [submit a pull request](./code).

Otherwise please [raise a bug report](./issues_and_bugs) so that we can fix it.

### What about right-to-left (RTL) languages (such as arabic)?

Silverstripe CMS doesn't have built-in support for attribute-based RTL-modifications (e.g. `<html dir="rtl">`).

If this is something you'd like to implement, we'd be eager to review a [pull request](./code) for it.

### Can I translate/edit the language files in my favorite text editor (on my local installation)

No, because it causes us a lot of work in merging these files back. Please use the online translation tool for all new and existing translations.

### How does my translation get into a Silverstripe CMS release?

Currently this is a manual process of a core team member downloading approved translations and committing them into our
source tree.

### How does my translation get approved, who is the maintainer?

The online translation tool (transifex.com) is designed to be decentralized and collaborative, so there's no strict
approval or review process. Every logged-in user on the system can flag translations, and discuss them with other
translators.

### I'm seeing lots of duplicated translations, what should I do?

For now, please translate all duplications - sometimes they might be intentional, but mostly the developer just didn't
know their phrase was already translated.

## Contact

Get in touch with translators on our [community Slack](https://silverstripe.org/slack) - please join the `#translations`
channel. For generic translation and Transifex questions you might like to use
[Stack Overflow](https://stackoverflow.com/search?q=transifex). Alternatively you can start a discussion on
[our forum](https://forum.silverstripe.org).
