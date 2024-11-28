---
title: Customising the Admin Interface
summary: Extend the admin view to provide custom behavior or new features for CMS and admin users.
introduction: The Admin interface can be extended to provide additional functionality to users and custom interfaces for managing data.
iconBrand: react
---

# Customising the admin interface

The Admin interface is bundled within the Silverstripe CMS but is most commonly used in conjunction with the `cms`
module. The main class for displaying the interface is a specialized [Controller](api:SilverStripe\Control\Controller) called [LeftAndMain](api:SilverStripe\Admin\LeftAndMain), named
as it is designed around a left hand navigation and a main edit form.

The user interface logic has a combination of jQuery and [jQuery.entwine](./jquery_entwine/) with [ReactJS](https://react.dev/). Some admin sections, such as `AssetAdmin`, are powered purely with react components, while others have a combination of both react components and jQuery logic.

[CHILDREN]

## How to's

[CHILDREN Folder="How_Tos"]
