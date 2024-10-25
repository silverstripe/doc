---
title: React and Redux
summary: Learn how to extend and customise the technologies we use for application state and client-rendered UI.
iconBrand: react
---

# Introduction to the "React" layer

Some admin modules render their UI with React, a popular JavaScript library created by Facebook.
For these sections, rendering happens via client side scripts that create and inject HTML
declaratively using data structures.

Even within sections that are *not* primarily rendered in react, several React components may be injected into the DOM.

There are some several members of this ecosystem that all work together to provide a dyanamic UI. They include:

- [ReactJS](https://react.dev/) - A JavaScript UI library
- [Redux](https://redux.js.org/) - A state manager for JavaScript

All of these pillars of the frontend application can be customised, giving you more control over how the admin interface looks, feels, and behaves.

> [!CAUTION]
> These technologies underpin the future of Silverstripe CMS development, but their current implementation is
> *experimental*. Our APIs are not expected to change drastically between releases, but they are excluded from
> our [semantic versioning](https://semver.org) commitments for the time being. Any breaking changes will be
> clearly signalled in release notes.

First, a brief summary of what each of these are:

## React

React's job is to render UI. Its UI elements are known as "components" and represent the fundamental building block of a React-rendered interface. A React component expressed like this:

```js
import React from 'react';
// ...

  <PhotoItem size={200} caption="Angkor Wat" onSelect={openLightbox}>
    <img alt="" src="path/to/image.jpg" />
  </PhotoItem>;
```

Might actually render HTML that looks like this:

```html
<div class="photo-item">
    <div class="photo" style="width:200px;height:200px;">
        <img src="path/to/image.jpg">
    </div>
    <div class="photo-caption">
        <h3><a>Angkor Wat/a></h3>
    </div>
</div>
```

This syntax is known as JSX. It is transpiled at build time into native JavaScript calls
to the React API. While optional, it is recommended to express components this way.

### Recommended: react dev tools

The React Dev Tools extension [available for Chrome](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en) [and for Firefox](https://addons.mozilla.org/en-GB/firefox/addon/react-devtools/) is critical to debugging a React UI. It will let you browse the React UI much like the DOM, showing the tree of rendered components and their current props and state in real time.

## Redux

Redux is a state management tool with a tiny API that affords the developer highly predictable behaviour. All of the application state is stored in a single object, and the only way to mutate that object is by calling an action, which is just a simple object that describes what happened. A function known as a *reducer* mutates the state based on that action and returns a new reference with the updated state.

The following example is taken from the [Redux GitHub page](https://github.com/reactjs/redux):

```js
// reducer
function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1;
    case 'DECREMENT':
      return state - 1;
    default:
      return state;
  }
}

const store = createStore(counter);
// subscribe to an action
store.subscribe(() => {
  const state = store.g.etState();
  // ... do something with the state here
});

// Call an action - in this case increment the state from 0 to 1
store.dispatch({ type: 'INCREMENT' });
```

### Recommended: redux devtools

It's important to be able to view the state of the React application when you're debugging and
building the interface.

To be able to view the state, you'll need to be in a dev environment
and have the [Redux Devtools](https://github.com/zalmoxisus/redux-devtools-extension)
installed on Google Chrome or Firefox, which can be found by searching with your favourite search
engine.

## For more information

This documentation will stop short of explaining React and Redux in-depth, as there is much better
documentation available all over the web. We recommend:

- [The Official React Tutorial](https://react.dev/learn)
- [Build With React](https://buildwithreact.com/tutorial)
- [Getting Started with Redux](https://egghead.io/courses/getting-started-with-redux)

## Build tools and using Silverstripe CMS react components {#using-cms-react-components}

Silverstripe CMS includes react, redux, and many other thirdparty dependencies already, which are exposed using [webpack's expose-loader plugin](https://webpack.js.org/loaders/expose-loader/) for you to use as [webpack externals](https://webpack.js.org/configuration/externals/).

There are also a lot of React components and other custom functionality (such as the injector, mentioned below) available for reuse. These are exposed in the same way.

The recommended way to access these dependencies is by using the [@silverstripe/webpack-config npm package](https://www.npmjs.com/package/@silverstripe/webpack-config). The documentation in the readme for that package explains how to use it.

If you are not using webpack to transpile your JavaScript, see if your build tooling has an equivalent to webpack's `externals` configuration. Alternatively, instead of `import`ing these dependencies, you can access them on the `window` object (for example the injector module is exposed as `window.Injector`).

## The `Injector` API

Much like Silverstripe CMS's [Injector API](../../extending/injector) in PHP,
the client-side framework has its own implementation of dependency injection
known as `Injector`. Using Injector, you can register new services, and
transform existing services.

Injector is broken up into three sub-APIs:

- `Injector.component` for React UI components
- `Injector.reducer` for Redux state management
- `Injector.form` for forms rendered via `FormSchema`.

The frontend Injector works a bit differently than its backend counterpart. Instead of *overriding* a service with your own implementation, you *enhance* an existing service with your own concerns. This pattern is known as [middleware](https://en.wikipedia.org/wiki/Middleware).

Middleware works a lot like a decorator. It doesn't alter the original API of the service,
but it can augment it with new features and concerns. This has the inherent advantage of allowing all thidparty code to have an influence over the behaviour, state, and UI of a component.

### A simple middleware example

Let's say you have an application that features error logging. By default, the error logging service simply outputs to `console.error`. But you want to customise it to send errors to a thirdparty service. For this, you could use middleware to augment the default functionality of the logger.

```js
// LoggingService.js

/* eslint-disable-next-line no-console */
const LoggingService = (error) => console.error(error);

export default LoggingService;
```

Now, let's add some middleware to that service. The signature of middleware is:

```js
const middleware = (next) => (args) => next(args);
```

Where `next()` is the next customisation in the "chain" of middleware. Before invoking the next implementation, you can add whatever customisations you need. Here's how we would use middleware to enhance `LoggingService`.

```js
import thirdPartyLogger from 'third-party-logger';

const addLoggingMiddleware = (next) => (error) => {
  if (error.type === LoggingService.CRITICAL) {
    thirdpartyLogger.send(error.message);
  }
  return next(error);
};
```

Then, we would create a new logging service that merges both implementations.

```js
import LoggingService from './LoggingService';
import addLoggingMiddleware from './addLoggingMiddleware';

const MyNewLogger = addLoggingMiddleware(LoggingService);
```

We haven't overridden any functionality. `LoggingService(error)` will still invoke `console.error`, once all the middleware has run. But what if we did want to kill the original functionality?

```js
import thirdPartyLogger from 'third-party-logger';
import LoggingService from './LoggingService';

const addLoggingMiddleware = (next) => (error) => {
  // Critical errors go to a thirdparty service
  if (error.type === LoggingService.CRITICAL) {
    thirdPartyLogger.send(error.message);
  }
  // Other errors get logged, but not to our thirdparty
  else if (error.type === LoggingService.ERROR) {
    next(error);
  }
  // Minor errors are ignored
  else {
    // Do nothing!
  }
};
```

### Registering new services to the `Injector`

If you've created a module using React, it's a good idea to afford other developers an
API to enhance those components, forms, and state. To do that, simply register them with `Injector`.

> [!WARNING]
> Because of the unique structure of the `form` middleware, you cannot register new services to `Injector.form`.

```js
// my-public-module/js/main.js
import Injector from 'lib/Injector';

Injector.component.register('MyComponent', MyComponent);
Injector.reducer.register('myCustom', MyReducer);
```

Services can then be fetched using their respective `.get()` methods.

```js
const MyComponent = Injector.component.get('MyComponent');
```

> [!CAUTION]
> Overwriting components by calling `register()` multiple times for the same
> service name is discouraged, and will throw an error. Should you really need to do this,
> you can pass `{ force: true }` as the third argument to the `register()` function.

### Transforming services using middleware

Now that the services are registered, other developers can customise your services with `Injector.transform()`.

```js
// someone-elses-module/js/main.js
Injector.transform(
  'my-transformation',
  (updater) => {
    updater.component('MyComponent', MyCustomComponent);
    updater.reducer('myCustom', MyCustomReducer);
  }
);
```

Much like the configuration layer, we need to specify a name for this transformation. This will help other modules negotiate their priority over the injector in relation to yours.

The second parameter of the `transform` argument is a callback which receives an `updater`object. It contains four functions: `component()`, `reducer()`, `form.alterSchema()` and `form.addValidation()`. We'll cover all of these in detail functions in detail further into the document, but briefly, these update functions allow you to mutate the DI container with a wrapper for the service. Remember, this function does not *replace*
the service - it enhances it with new functionality.

#### Helpful tip: name your component middleware

Since multiple enhancements can be applied to the same component, it will be really
useful for debugging purposes to reveal the names of each enhancement on the `displayName` of
 the component. This will really help you when viewing the rendered component tree in
 [React Dev Tools](https://chrome.g.oogle.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en).

 For this, you can use the third parameter of the `updater.component` function. It takes an arbitrary
 name for the enhancement you're applying.

 ```js
 // module-a/js/main.js
 (updater) => updater.component('TextField', CharacterCounter, 'CharacterCounter');
 ```

 ```js
 // module-b/js/main.js
 (updater) => updater.component('TextField', TextLengthChecker, 'TextLengthChecker');
 ```

### Controlling the order of transformations

Sometimes, it's critical to ensure that your customisation happens after another one has been executed. To afford you control over the ordering of transforms, Injector allows `before` and `after` attributes as metadata for the transformation.

```js
// my-module/js/main.js
Injector.transform(
  'my-transformation',
  (updater) => {
    updater.component('MyComponent', MyCustomComponent);
    updater.reducer('myCustom', MyCustomReducer);
  },
  { after: 'another-module' }
);
```

`before` and `after` also accept arrays of constraints.

```js
Injector.transform(
  'my-transformation',
  (updater) => updater.component('MyComponent', MyCustomComponent),
  { before: ['my-transformation', 'some-other-transformation'] }
);
```

#### Using the `*` flag

If you really want to be sure your customisation gets loaded first or last, you can use
`*` as your `before` or `after` reference.

```js
Injector.transform(
  'my-transformation',
  (updater) => updater.component('MyComponent', FinalTransform),
  { after: '*' }
);
```

> [!NOTE]
> This flag can only be used once per transformation.
> The following are not allowed:
>
> - `{ before: ['*', 'something-else'] }`
> - `{ after: '*', before: 'something-else' }`

### Injector context

Because so much of UI design depends on context, dependency injection in the frontend is not necessarily universal. Instead, services are fetched with context.

*example:*

```js
const CalendarComponent = Injector.get('Calendar', 'AssetAdmin.FileEditForm.StartDate');
```

Likewise, services can be applied for specific contexts.

```js
Injector.transform('my-transform', (updater) => {
  // Applies to all text fields in AssetAdmin
  updater.component('TextField.AssetAdmin', MyTextField);

  // Applies to all text fields in AssetAdmin editform
  updater.component('TextField.AssetAdmin.FileEditForm', MyTextField);

  // Applies to any textfield named "Title" in AssetAdmin
  updater.component('TextField.AssetAdmin.*.Title', MyTextField);

  // Applies to any textfield named "Title" in any admin
  updater.component('TextField.*.*.Title', MyTextField);
});
```

To apply context-based transformations, you'll need to know the context of the component you want to customise. To learn this,
open your React Developer Tools (see above) window and inspect the component name. The
context of the component is displayed between two square brackets, appended to the original name, for example:
`TextField[TextField.AssetAdmin.FileEditForm.Title]`. The context description is hierarchical, starting
with the most general category (in this case, "Admin") and working its way down to the most specific
category (Name = 'Title'). You can use Injector to hook into the level of specificity that you want.

## Customising react components with `Injector`

When middleware is used to customise a React component, it is known as a [higher order component](https://facebook.github.io/react/docs/higher-order-components.html).

Using the `PhotoItem` example above, let's create a customised `PhotoItem` that allows a badge, perhaps indicating that it is new to the gallery.

```js
import React from 'react';
// ...

const enhancedPhoto = (PhotoItem) => (props) => {
  const badge = props.isNew ?
    <div className="badge">New!</div> :
    null;

  return (
    <div>
      {badge}
      <PhotoItem {...props} />
    </div>
  );
};

const EnhancedPhotoItem = enhancedPhoto(PhotoItem);

  <EnhancedPhotoItem isNew size={300} />;
```

Alternatively, this component could be expressed with an ES6 class, rather than a simple
function.

```js
import React from 'react';
// ...

const enhancedPhoto = (PhotoItem) => (
  class EnhancedPhotoItem extends React.Component {
    render() {
      const badge = this.props.isNew ?
        <div className="badge">New!</div> :
        null;

      return (
        <div>
          {badge}
          <PhotoItem {...this.props} />
        </div>
      );
    }
  }
);
```

When components are stateless, using a simple function in lieu of a class is recommended.

## Using dependencies within your react component

If your component has dependencies, you can add them via the injector using the `inject()`
higher order component. The function accepts the following arguments:

```js
inject([dependencies], mapDependenciesToProps, getContext)(Component);
```

- **[dependencies]**: An array of dependencies (or a string, if just one)
- **mapDependenciesToProps**: (optional) All dependencies are passed into this function as params. The function
is expected to return a map of props to dependencies. If this parameter is not specified,
the prop names and the service names will mirror each other.
- **getContext**: A callback function with params `(props, currentContext)` that will calculate the context to
use for determining which transformations apply to the dependencies. This defaults to the current context. This
could help when any customisations that may calls for a change (or tweak) to the current context.

The result is a function that is ready to apply to a component.

 ```js
const MyInjectedComponent = inject(
   ['Dependency1', 'Dependency2']
)(MyComponent);
// MyComponent now has access to props.Dependency1 and props.Dependency2
```

Here is its usage with a bit more context:

```js
// my-module/js/components/Gallery.js
import React from 'react';
import { inject } from 'lib/Injector';

class Gallery extends React.Component {
  render() {
    const { SearchComponent, ItemComponent } = this.props;
    return (
      <div>
        <SearchComponent />
        {this.props.items.map(item => (
          <ItemComponent title={item.title} image={item.image} />
        ))}
      </div>
    );
  }
}

export default inject(
  ['GalleryItem', 'SearchBar'],
  (GalleryItem, SearchBar) => ({
    ItemComponent: GalleryItem,
    SearchComponent: SearchBar
  }),
  () => 'Gallery.Search'
)(Gallery);
```

The properties used by `inject()` are soft-supplied. This means a parent calling a component that uses
`inject()` could choose to overwrite the dependencies which `inject()` would have otherwise supplied.
Here is an example using the above `Gallery` component with the dependency `ItemComponent` overwritten by the
calling component. We pull in a previously registered `PreviewItem` to replace the former `GalleryItem`.

```js
// my-module/js/components/PreviewSection.js
import React from 'react';
import { inject } from 'lib/Injector';

class PreviewSection extends React.Component {
  render() {
    const { Gallery, PreviewItem } = this.props;
    return (
      <div className="preview-section">
        <div className="preview-sidebar">Sidebar here</div>
        <Gallery ItemComponent={PreviewItem}/>
      </div>
    );
  }
}

export default inject(
  ['Gallery', 'PreviewItem']
)(PreviewSection);
```

Another way to provide context to injector is by using the `provideContext` HOC, rather than
the `getContext` param in `inject()`.

```js
// my-module/js/components/ContextualSection.js
import React, { Component } from 'react';
import { provideContext, inject } from 'lib/Injector';

class MySection extends Component {
  // ... section code here ...
}

export default compose(
  provideContext('Gallery.Search'),
  inject(['Gallery'])
)(MySection);
```

## Using the injector directly within your component

On rare occasions, you may just want direct access to the injector in your component. If
your dependency requirements are dynamic, for example, you won't be able to explicitly
declare them in `inject()`. In cases like this, use `withInjector()`. This higher order
component puts the `Injector` instance in `context`.

```js
import React from 'react';
// ...

class MyGallery extends React.Component {
  render() {
    return <div>
      {this.props.items.map(item => {
        const Component = this.context.injector.get(item.type, 'Reports.context');
        return <Component title={item.title} image={item.image} />;
      })}
    </div>;
  }
}

export default withInjector(MyGallery);
```

The `Reports.context` in the second parameter provides a context for the injector to determine
which transformations to apply to or remove from the component you're looking to get.
More details about transformations below.

## Using injector to customise forms

Forms in the React layer are built declaratively, using the `FormSchema` API. A component called
`FormBuilderLoader` is given a URL to a form schema definition, and it populates itself with fields
(both structural and data-containing) and actions to create the UI for the form. Each form is required
to have an `identifier` property, which is used to create context for Injector when field components
are fetched. This affords developers the opportunity provide very surgical customisations.

### Updating the form schema

Most behavioural and aesthetic customisations will happen via a mutation of the form schema. For this,
we'll use the `updater.form.alterSchema()` function.

```js
Injector.transform(
  'my-custom-form',
  (updater) => {
    updater.form.alterSchema(
      'AssetAdmin.*',
      (form) =>
        form.updateField('Title', {
          myCustomProp: true
        })
          .getState()
    );
  }
);
```

> [!WARNING]
> It is critical that you end series of mutation calls with `getState()`.

The `alterSchema()` function takes a callback, with an instance of `FormStateManager` (`form` in the
above example) as a parameter. `FormStateMangaer` allows you to declaratively update the form schema
API using several helper methods, including:

- `updateField(fieldName:string, updates:object)`
- `updateFields({ myFieldName: updates:object })`
- `mutateField(fieldName:string, callback:function)`
- `setFieldComponent(fieldName:string, componentName:string)`
- `setFieldClass(fieldName:string, cssClassName:string, active:boolean)`
- `addFieldClass(fieldName:string, cssClassName:string)`
- `removeFieldClass(fieldName:string, cssClassName:string)`

> [!NOTE]
> For a complete list of props that are available to update on a `Field` object,
> see <https://redux-form.com/8.3.0/docs/api/field.md/#props-you-can-pass-to-field>

In addition to mutation methods, several readonly methods are available on `FormSchemaManager` to read the current form state, including:

- `getValues()`: Returns a map of field names to their current values
- `getValue(fieldName:string)`: Returns the value of the given field
- `isDirty()`: Returns true if the form has been mutated from its original state
- `isPristine()`: Returns true if the form is in its original state
- `isValid()`: Returns true if the form has no validation errors
- `isInvalid()`: Returns true if the form has validation errors

### Adding validation to a form

Validation for React-rendered forms is handled by the [redux-form](https://redux-form.com) package. You can inject your own middleware to add custom validation rules using the `updater.form.addValidation()` function.

```js
Injector.transform(
  'my-validation',
  (updater) => {
    updater.form.addValidation(
      'AssetAdmin.*',
      (values, validator) => {
        if (values.PostalCode.length !== 5) {
          validator.addError('PostalCode', 'Invalid postal code');
        }
      }
    );
  }
);
```

The `addValidation()` function takes a callback, with an instance of `FormValidationManager` (`validator` in the above example) as a parameter. `FormValidationMangaer` allows you to manage the validation result using several helper methods, including:

- `addError(fieldName:string, message:string)`
- `addErrors(fieldName:string, messages:Array)`
- `hasError(fieldName:string)`
- `clearErrors(fieldName:string)`
- `getErrors(fieldName:string)`
- `reset(void)`

## Using injector to customise redux state data

Before starting this tutorial, you should become familiar with the concepts of [Immutability](https://www.sitepoint.com/immutability-javascript/) and [Redux](https://redux.js.org).

For example:

```js
newProps = { ...oldProps, name: 'New name' };
```

is the same as

```js
newProps = Object.assign(
  {},
  oldProps,
  { name: 'New name' }
);
```

To start customising, you'll need to transform an existing registered reducer, you can find what reducers are registered by importing Injector and running `Injector.reducer.getAll()`

```js
Injector.transform('customisationName', (updater) => {
  updater.reducer('assetAdmin', MyReducerTransformer);
});
```

As you can see, we use the `reducer()` function on the `update` object to augment Redux state transformations.

### Using redux dev tools

It is important to learn the basics of [Redux dev tools](https://github.com/reduxjs/redux-devtools/tree/main/extension#installation), so that you can find out what ACTIONS and payloads to intercept and modify in your Transformer should target.

Most importantly, it helps to understand the "Action" sub-tab on the right panel (bottom if your dev tools is small), as this will be the data your Transformer will most likely receive, pending other transformers that may run before/after your one.

### Structuring a transformer

We use currying to supply utilities which your transformer may require to handle the transformation.

- `originalReducer` - reducer callback which the transformer is customising, this will need to be called in most cases. This will also callback other transformations down the chain of execution. Not calling this will break the chain.
- `getGlobalState` - A function that gets the state of the global Redux store. There may be data outside the current scope in the reducer which you may need to help determine the transformation.
- `state` - current state of the current scope. This is what should be used to form the new state.
- `type` - the action to fire, like in any reducer in Redux. This helps determine if your transformer should do anything.
- `payload` - the new data sent with the action to mutate the Redux store.

```js
const MyReducerTransformer = (originalReducer) => (globalState) => (state, { type, payload }) => {
  switch (type) {
    case 'EXISTING_ACTION': {
      // recommended to call and return the originalReducer with the payload changed by the transformer
      return originalReducer(/* ... */);
    }

    case 'OVERRIDE_EXISTING_ACTION': {
      // could omit the originalReducer to enforce your change or cancel the originalREducer's change
      return originalReducer(/* ... */);
    }

    default: {
      // it is important to return the originalReducer with original redux parameters.
      return originalReducer(state, { type, payload });
    }
  }
};
```

### A basic transformation

This example we will illustrate modifying the payload to get different data saved into the original reducer.

We will rename anything in the breadcrumbs that is displaying "Files" to display "Custom Files" instead.

```js
const MyReducerTransformer = (originalReducer) => (getGlobalState) => (state, { type, payload }) => {
  switch (type) {
    case 'SET_BREADCRUMBS': {
      return originalReducer(state, {
        type,
        payload: {
          breadcrumbs: payload.breadcrumbs.map((crumb) => (
            (crumb.text === 'Files')
              ? { ...crumb, text: 'Custom Files' }
              : crumb
          )),
        },
      });
    }
    default: {
      return state;
    }
  }
};
```

### Using the globalState

Accessing the globalState is easy, as it is passed in as part of the curried functions definition.

```js
export default (originalReducer) => (getGlobalState) => (state, { type, payload }) => {
  const baseUrl = globalState.config.baseUrl;

  switch (type) {
    /* ... cases here ... */
    default: {
      // ...
    }
  }
};
```

### Setting a different initial state

We can easily define a new initial state by providing the `state` param with a default value.
It is recommended to keep the call for the original initialState for your initialState then override values, so that you do not lose any potentially critical data that would have originally been set.

```js
const MyReducerTransformer = (originalReducer) => () => (state, { type, payload }) => {
  if (typeof state === 'undefined') {
    return {
      ...originalReducer(state, { type, payload }),
      myCustom: 'initial state here',
    };
  }
  return state;
};
```

### Cancelling an action

There are valid reasons to break the chain of reducer transformations, such as cancelling the Redux store update.
However, like an original reducer in redux, you will still need to return the original state.

```js
export default (originalReducer) => (getGlobalState) => (state, { type, payload }) => {
  switch (type) {
    case 'CANCEL_THIS_ACTION': {
      return state;
    }
    default: {
      return state;
    }
  }
};
```

### Calling a different action

You could manipulate the action called by the originalReducer, there isn't an example available but this block of
 code will present the theory of how it can be achieved.

```js
export default (originalReducer) => (getGlobalState) => (state, { type, payload }) => {
  switch (type) {
    case 'REMOVE_ERROR': {
      // we'd like to archive errors instead of removing them
      return originalReducer(state, {
        type: 'ARCHIVE_ERROR',
        payload,
      });
    }
    default: {
      return state;
    }
  }
};
```

### Extensions are only as good as the code they're extending

An important point to remember about these types of deep customisations is that they all depend heavily on the core code they're modifying to follow specific patterns. The more the core code makes use of `Injector` the easier it will be for third party developers to extend. Conversely, if the core is full of hard-coded component definitions and statically written queries, customisation will be at best less surgical and at worst, not possible.
