---
title: CSS coding conventions
summary: The CSS style guidelines we follow in our open source software
iconBrand: css3
---
# CSS and SCSS coding conventions

## Overview

This document provides guidelines for code formatting to developers contributing
to Silverstripe. It applies to all CSS/Sass files in the Silverstripe core modules.

## Browser support

Check our [requirements](/getting_started/server_requirements) documentation.

## Tools and libraries

Styles are written in the [SCSS language](https://sass-lang.com/).
We use [Bootstrap 5](https://getbootstrap.com/) styles where possible.

## Conventions

We follow the [AirBnB CSS Conventions](https://github.com/airbnb/css)
and the [BEM](https://getbem.com/) methodology (block-element-modifier).

Because we use [Bootstrap 5](https://getbootstrap.com/) which
does not follow [BEM](https://getbem.com/) naming convention there will be
a lot of places where class names voilate BEM.
However, please note that they are not a indicator of how to name classes.
Use BEM conventions where possible.

## Linting

We use [sass-lint](https://github.com/sasstools/sass-lint) to ensure all new SCSS
written complies with the rules below. It will be provided as an npm dev dependency.
There are also quite a few [sass-lint IDE integrations](https://github.com/sasstools/sass-lint#ide-integration)
which highlight any linting errors right in your code.

We strongly recommend installing one of these into the editor of your choice, to
avoid the frustration of failed pull requests. You can run the checks on console
via `yarn lint` (see [Build Tooling](/contributing/build_tooling)).

## File and folder naming

- All frontend files (CSS, JavaScript, images) should be placed in
  a `client/` folder on the top level of the module
- The `client/src/components` folder should contain only reusable components
  (e.g. Button, Accordion). Presentation of these components should not rely on
  the markup context they're embedded in.
- The `client/src/containers` folder should contain use-case dependent styles only
  (e.g. AssetAdmin). Styles in here should be kept at a minimum.
- The file name of styles nested within components and containers should inherit their
  respective folder name for easy reference.
  For example, a `components/FormAction` component has styles named `FormAction.scss`).
- The `client/src/styles` folder contains base styles (reset, typography, variables)
  as well as layout-related styles which arranges components together.

## Icons and graphics

Most graphics used in the CMS are vector based, and stored as generated
webfonts in `admin/client/src/font`, which also contains a HTML reference.
The webfonts are generated through the [Fontastic](https://app.fontastic.me) service.
If you need new icons to be added, please ping us on GitHub.

## Legacy conventions

There is some legacy code that may not follow the conventions outlined above. If you modify any of these,
consider porting them over into the new structure. Otherwise, follow these conventions:

- Class naming: Use the `cms-` class prefix for major components in the CMS interface,
  and the `ss-ui-` prefix for extensions to jQuery UI. Don't use the `ui-` class prefix, its reserved for jQuery UI built-in styles.
- Use jQuery UI's built-in styles where possible, e.g. `ui-widget` for a generic container, or `ui-state-highlight`
  to highlight a specific component. See the [jQuery UI Theming API](https://api.jqueryui.com/category/theming/) for a full list.

## Related

- [PHP Coding Conventions](/contributing/php_coding_conventions)
- [JavaScript Coding Conventions](/contributing/javascript_coding_conventions)
- [Browser support](/getting_started/server_requirements/)
