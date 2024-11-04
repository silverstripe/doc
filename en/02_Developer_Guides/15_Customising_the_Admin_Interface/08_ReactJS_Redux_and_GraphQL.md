---
title: React, Redux, and GraphQL
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
- [GraphQL](https://graphql.org/) - A query language for your API
- [Apollo Client](https://www.apollographql.com/apollo-client) - A framework for using GraphQL in your application

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

## GraphQL and apollo

[GraphQL](https://graphql.org/learn/) is a strictly-typed query language that allows you to describe what data you want to fetch from your API. Because it is based on types, it is self-documenting and predictable. Further, it's structure lends itself nicely to fetching nested objects. Here is an example of a simple GraphQL query:

```graphql
query GetUser($ID: Int!) {
    user {
        name
        email
        blogPosts {
            title
            comments(Limit: 5) {
                author
                comment
            }
        }

    }
}
```

The above query is almost self-descriptive. It gets a user by ID, returns his or her name and email address, along with the title of any blog posts he or she has written, and the first five comments for each of those. The result of that query is, very predictably, JSON that takes on the same structure.

```json
{
    "user": {
        "name": "Test user",
        "email": "test@example.com",
        "blogPosts": [
            {
                "title": "How to be awesome at GraphQL",
                "comments": [
                    {
                        "author": "Uncle Cheese",
                        "comment": "Nice stuff, bro"
                    }
                ]
            }
        ]
    }
}
```

On its own, GraphQL offers nothing functional, as it's just a query language. You still need a service that will invoke queries and map their results to UI. For that, Silverstripe CMS uses an implementation of [Apollo Client](https://www.apollographql.com/docs/react/) that works with React.

## For more information

This documentation will stop short of explaining React, Redux, and GraphQL/Apollo in-depth, as there is much better
documentation available all over the web. We recommend:

- [The Official React Tutorial](https://react.dev/learn)
- [Build With React](https://buildwithreact.com/tutorial)
- [Getting Started with Redux](https://egghead.io/courses/getting-started-with-redux)
- [The React Apollo docs](https://www.apollographql.com/docs/react/)
- [GraphQL in Silverstripe](/developer_guides/graphql/)

## Build tools and using Silverstripe CMS react components {#using-cms-react-components}

Silverstripe CMS includes react, redux, GraphQL, apollo, and many other thirdparty dependencies already, which are exposed using [webpack's expose-loader plugin](https://webpack.js.org/loaders/expose-loader/) for you to use as [webpack externals](https://webpack.js.org/configuration/externals/).

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

## Using injector to customise GraphQL queries

One of the strengths of GraphQL is that it allows us to declaratively state exactly what data a given component needs to function. Because GraphQL queries and mutations are considered primary concerns of a component, they are not abstracted away somewhere in peripheral asynchronous functions. Rather, they are co-located with the component definition itself.

The downside of this is that, because queries are defined statically at compile time, they don't adapt well to the extension patterns that are inherent to Silverstripe CMS projects. For instance, a query for a [`Member`](api:SilverStripe\Security\Member) record may include fields for `FirstName` and `Email`, but if you have customised that class via extensions, and would like the component using that query to display your custom fields, your only option would be to override the entire query and the component with a custom implementation. In backend code, this would be tantamount to replacing the entire `Member` class and `SecurityAdmin` section just because you had a new field. You would never do that, right? It's an over-aggressive hack! We need APIs that make extension easy.

To that end, the `Injector` library provides a container for abstract representations of GraphQL queries and mutations. You can register and transform them as you do components and reducers. They exist merely as abstract concepts until `Injector` loads, at which time all transformations are applied, and each registered query and mutation is composed and attached to their assigned components.

### Extensions are only as good as the code they're extending

An important point to remember about these types of deep customisations is that they all depend heavily on the core code they're modifying to follow specific patterns. The more the core code makes use of `Injector` the easier it will be for third party developers to extend. Conversely, if the core is full of hard-coded component definitions and statically written queries, customisation will be at best less surgical and at worst, not possible. For this reason, we'll look at GraphQL customisations from two sides - making code extensible, and then extending that code.

### Building an extensible GraphQL component

Let's imagine that we have a module that adds a tab where the user can write "notes" about the content they are editing. We'll use GraphQL and React to render this UI. We have a dataobject called "Note" where we store these in the database.

Here's what that might look like:

```js
// my-module/client/src/components/Notes.js
import React from 'react';
import gql from 'graphql-tag';
import { graphql } from '@apollo/client/react/hoc';

export const Notes = ({ notes }) => (
  <ul className="notes">
    {notes.map(note => <li key={note.id}>{note.content}</li>)}
  </ul>
);

const getNotesQuery = gql`
query ReadNotes {
  readNotes {
    id
    content
  }
}
`;

const apolloConfig = {
  props({ data: { readNotes } }) {
    return {
      notes: readNotes || []
    };
  }
};

const NotesWithData = graphql(getNotesQuery, apolloConfig)(Notes);

export default NotesWithData;
```

Next we'll expose the model to GraphQL:

```yml
# my-module/_config/graphql.yml

# Tell graphql that we're adding to the admin graphql schema
SilverStripe\GraphQL\Schema\Schema:
  schemas:
    admin:
      src:
        - my-module/_graphql
```

```yml
# my-module/_graphql/models.yml

# Tell graphql how to scaffold the schema for our model
App\Model\Note:
  fields:
    id: true
    content: true
  operations:
    read:
      plugins:
        paginateList: false
```

#### Define the app

Finally, let's make a really simple container app which holds a header and our notes component, and inject it into the DOM using entwine.

```js
// my-module/client/src/App.js
import React from 'react';
import Notes from './components/Notes';

const App = () => (
  <div>
    <h3>Notes</h3>
    <Notes />
  </div>
);

export default App;
```

```js
import { createRoot } from 'react-dom/client';
import React from 'react';
import { ApolloProvider } from '@apollo/client';
import Injector from 'lib/Injector';
import App from './App';

Injector.ready(() => {
  const { apolloClient, store } = window.ss;

  // Assuming you've got some element in the DOM with the id "notes-app"
  $('#notes-app').entwine({
    ReactRoot: null,

    onmatch() {
      const root = createRoot(this[0]);
      this.setReactRoot(root);
      root.render(
        <ApolloProvider client={apolloClient} store={store}>
          <App />
        </ApolloProvider>
      );
    },

    onunmatch() {
      const root = this.getReactRoot();
      if (root) {
        root.unmount();
        this.setReactRoot(null);
      }
    },
  });
});
```

> [!NOTE]
> `this[0]` is how we get the underlying DOM element that the jQuery object is wrapping. We can't pass `this` directly into the `createRoot()` function because react doesn't know how to deal with a jQuery object wrapper. See [the jQuery documentation](https://api.jquery.com/Types/#jQuery) for more information about that syntax.

The `silverstripe/admin` module provides `apolloClient` and `store` objects in the global namespace to be shared by other modules. We'll make use of those, and create our own app wrapped in `<ApolloProvider />`.

We register a callback with `Injector.ready()` because the `apolloClient` and `store` are ultimately coming from the injector, so we need to make sure those are ready before mounting our component.

To mount the app, we use the `onmatch()` event fired by entwine, and we're off and running. Just don't forget to unmount the component in `onunmatch()`.

What we've just built may work, but we've made life very difficult for other developers. They have no way of customising this. Let's change that.

#### Register as much as possible with injector

The best thing you can do to make your code extensible is to use `Injector` early and often. Anything that goes through Injector is easily customisable.

First, let's break up the list into smaller components.

```js
// my-module/client/src/components/NotesList.js
import React from 'react';
import { inject } from 'lib/Injector';

const NotesList = ({ notes = [], ItemComponent }) => (
  <ul className="notes">
    {notes.map(note => <ItemComponent key={note.id} note={note} />)}
  </ul>
);

// This tells the injector we want a component named "NotesListItem".
// We'll register our version of that component, and then other people can make
// any transformations that they like.
export default inject(
  ['NotesListItem'],
  // This second argument remaps the injected component name (NotesListItem) with our prop name
  // (ItemComponent). If the prop is named the same as the injected name, we can ommit this second
  // argument.
  (NotesListItem) => ({
    ItemComponent: NotesListItem
  })
)(NotesList);
```

```js
// my-module/client/src/components/NotesListItem.js
import React from 'react';

const NotesListItem = ({ note }) => <li>{note.content}</li>;

export default NotesListItem;
```

#### Creating an abstract query definition

The next piece is the query. We'll need to register that with `Injector`. Unlike components and reducers, this is a lot more abstract. We're actually not going to write any GraphQL at all. We'll just build the concept of the query in an abstraction layer, and leave `Injector` to build the GraphQL syntax at runtime.

```js
// my-module/client/src/state/readNotes.js
import { graphqlTemplates } from 'lib/Injector';

const { READ } = graphqlTemplates;

const query = {
  apolloConfig: {
    props({ data: { readNotes } }) {
      return {
        notes: readNotes || [],
      };
    }
  },
  templateName: READ,
  pluralName: 'Notes',
  pagination: false,
  params: {},
  fields: [
    'id',
    'content',
  ],
};

export default query;
```

Dynamic GraphQL queries are generated by populating pre-baked templates with specific pieces of data, including fields, fragments, variables, parameters, and more. By default, the templates available to you follow the GraphQL scaffolding API (`readMyObjects`, `readOneMyObject`, `createMyObject`, `deleteMyObject`, and `updateMyObject`).

In this example, we're using the `READ` template, which needs to know the plural name of the object (e.g. `READ` with `Notes` makes a `readNotes` query), whether pagination is activated, and which fields you want to query.

> [!TIP]
> For simplicity, we're not querying any relations or otherwise nested data here. If we had, for example, a `foo` relation with a `title` field and this was exposed in the schema, we would need to add it to the fields array like this:
>
> ```js
> const query = {
>   // ...
>   fields: [
>     'foo', [
>       'title',
>     ]
>   ],
> };
> ```
>
> You might instinctively try to use JSON object notation for this instead, but that won't work.

#### Register all the things

Let's now register all of this with Injector.

```js
// my-module/client/src/boot/registerDependencies.js
import Injector, { injectGraphql } from 'lib/Injector';
import NotesList from '../components/NotesList';
import NotesListItem from '../components/NotesListItem';
import readNotes from '../state/readNotes';

const registerDependencies = () => {
  Injector.component.register('NotesList', NotesList);
  Injector.component.register('NotesListItem', NotesListItem);
  Injector.query.register('ReadNotes', readNotes);
};

export default registerDependencies;
```

> [!TIP]
> If you have a lot of components or queries to add, you can use `registerMany` instead:
>
> ```js
> Injector.component.registerMany({
>   NotesList,
>   NotesListItem,
>   // ...etc
> });
> ```

We use `Injector.query.register()` to register our `readNotes` query so that other projects can extend it.

#### Applying the injected query as a transformation

The only missing piece now is to attach the `ReadNotes` injected query to the `NotesList` component. We could have done this using `injectGraphql` in the `NotesList` component itself, but instead, we'll do it as an Injector transformation. Why? There's a good chance whoever is customising the query will want to customise the UI of the component that is using that query. If someone adds a new field to a query, it is likely the component should display that new field. Registering the GraphQL injection as a transformation will allow a thirdparty developer to override the UI of the component explicitly *after* the GraphQL query is attached. This is important, because otherwise the customised component wouldn't use the query.

```js
// my-module/client/src/boot/registerDependencies.js

// ...
const registerDependencies = () => {
  // ...
  Injector.transform(
    'noteslist-graphql',
    (updater) => {
      updater.component('NotesList', injectGraphql('ReadNotes'));
    }
  );
};

export default registerDependencies;
```

The transformation adds the higher-order component `injectGraphQL`, using the query we have just registered, `ReadNotes` as a dependency - basically, we're injecting the result of the query into the component.

All of this feels like a lot of extra work, and, to be fair, it is. You're probably used to simply inlining one or many higher-order component compositions in your components. That works great when you're not concerned about making your components extensible, but if you want others to be able to customise your app, you really need to be sure to follow these steps.

#### Update the app

Our container app needs to have the `NotesList` component injected into it.

```js
// my-module/client/src/App.js
import React from 'react';
import { inject } from 'lib/Injector';

const App = ({ NotesList }) => (
  <div>
    <h3>Notes</h3>
    <NotesList />
  </div>
);

export default inject(['NotesList'])(App);
```

You can register the `App` component with `Injector`, too, but since it's already injected with dependencies it could get pretty convoluted. High level components like this are best left uncustomisable.

#### Use the injector from an entwine context

Since almost everything is in `Injector` now, we need to update our mounting logic to inject the dependencies into our app.

```js
import { createRoot } from 'react-dom/client';
import React from 'react';
import { ApolloProvider } from '@apollo/client';
import Injector, { provideInjector } from 'lib/Injector';
import registerDependencies from './boot/registerDependencies';
import App from './App';

registerDependencies();

Injector.ready(() => {
  const { apolloClient, store } = window.ss;
  const MyApp = () => (
    <ApolloProvider client={apolloClient} store={store}>
      <App />
    </ApolloProvider>
  );
  const MyAppWithInjector = provideInjector(MyApp);

  $('#notes-app').entwine({
    ReactRoot: null,

    onmatch() {
      const root = createRoot(this[0]);
      this.setReactRoot(root);
      root.render(<MyAppWithInjector />);
    },

    onunmatch() {
      const root = this.getReactRoot();
      if (root) {
        root.unmount();
        this.setReactRoot(null);
      }
    },
  });
});
```

The callback we register with `Injector.ready()` is even more important now - it ensures that we don't attempt to render anything before the transformations have been applied, which would result in fatal errors.

We then make our app `Injector` aware by wrapping it with the `provideInjector` higher-order component.

### Extending an existing GraphQL app

Let's suppose we have a project that extends the `Notes` object in some way. Perhaps we have a `Priority` field whose value alters the UI in some way. Thanks to a module developer who gave use plenty of extension points through `Injector`, this will be pretty easy.

#### Applying the extensions

We'll first need to apply the extension and update our GraphQL scaffolding.

```yml
# app/_config/extensions.yml
App\Model\Note:
  extensions:
    # this extension adds a "Priority" integer field
    - MyOtherApp\Extension\NoteExtension
```

Remember, this example is in a project which is customising the schema from the previous example, so we still have to tell GraphQL where to find our schema modifications.

If you're following along, you could declare a different folder than before within the same project so you can see how the schema definitions merge together into a single schema.

```yml
# app/_config/graphql.yml
SilverStripe\GraphQL\Schema\Schema:
  schemas:
    admin:
      src:
        - app/_graphql
```

```yml
# app/_graphql/models.yml
App\Model\Note:
  fields:
    priority: true
```

#### Creating transforms

Let's first update the `NotesListItem` to contain our new field.

> [!WARNING]
> Note that we're overriding the entire `NotesListItem` component. This is the main reason we broke the original list up into smaller components.

```js
// app/client/src/transformNotesListItem.js
import React from 'react';

const transformNotesListItem = () => ({ note: { content, priority } }) => (
  <li className={`priority-${priority}`}>{content} [PRIORITY: {['Low', 'Medium', 'High'][priority]}]</li>
);

export default transformNotesListItem;
```

Now, let's update the query to fetch our new field.

```js
// app/client/src/transformReadNotes.js
const transformReadNotes = (manager) => {
  manager.addField('priority');
};

export default transformReadNotes;
```

Simple! The transformation passes us a `ApolloGraphQLManager` instance that provides a fluent API for updating a query definition the same way the `FormStateManager` allows us to update Redux form state.

#### Adding fields

In the above example, we added a single field to a query. Here's how that works:

```js
manager.addField(fieldName, fieldPath = 'root');
```

The `fieldPath` argument tells the manager at what level to add the field. In this case, since the `priority` field is going on the root query (`readNotes`), we'll use `root` as the path. But suppose we had a more complex query like this:

```graphql
query readMembers {
    firstName
    surname
    friends {
        email
        company {
            name
        }
    }
}
```

If we wanted to add a field to the nested `company` query on `friends`, we would use a path syntax.

```js
manager.addField('tagline', 'root/friends/company');
```

#### Adding field arguments

Let's suppose we had the following query:

```graphql
query ReadMembers($imageSize: String!) {
    readMembers {
        firstName
        avatar(size: $imageSize)
        company {
            name
        }
    }
}
```

Maybe the `company` type has a `logo`, and we want to apply the `imageSize` parameter as an argument to that field.

```js
manager.addArg('size', 'imageSize', 'root/company/logo');
```

Where `root/company/logo` is the path to the field, `size` is the name of the argument on that field, and `imageSize` is the name of the variable.

#### Applying the transforms

Now, let's apply all these transformations, and we'll use the `after` property to ensure they get applied in the correct sequence.

```js
// app/client/src/boot.js
import Injector from 'lib/Injector';
import transformNotesListItem from './transformNotesListItem';
import transformReadNotes from './transformReadNotes';

Injector.transform(
  'noteslist-query-extension',
  (updater) => {
    updater.component('NotesListItem', transformNotesListItem);
    updater.query('ReadNotes', transformReadNotes);
  },
  { after: 'noteslist-graphql' }
);
```

> [!TIP]
> This transformation could either be transpiled as-is, or if you have other JavaScript to include in this module you might want to export it as a function and call it from some entry point.
> Don't forget to add the transpiled result to the CMS e.g. via the `SilverStripe\Admin\LeftAndMain.extra_requirements_javascript` configuration property.

### Creating extensible mutations

Going back to the original module, let's add an `AddForm` component to our list that lets the user create a new note.

```js
// my-module/client/src/components/AddForm.js
import React, { useRef } from 'react';

const AddForm = ({ onAdd }) => {
  const inputRef = useRef(null);
  return (
    <div>
      <label>New note</label>
      <input type="text" ref={inputRef}/>
      <button onClick={(e) => {
        e.preventDefault();
        onAdd(inputRef && inputRef.value);
      }}
      >Add</button>
    </div>
  );
};

export default AddForm;
```

> [!NOTE]
> Because this isn't a full react tutorial, we've avoided the complexity of ensuring the list gets updated when we add an item to the form. You'll have to refresh the page to see your note after adding it.

And we'll inject that component into our `App` container.

```js
// my-module/client/src/App.js
import React from 'react';
import { inject } from 'lib/Injector';

const App = ({ NotesList, NoteAddForm }) => (
  <div>
    <h3>Notes</h3>
    <NotesList />
    <NoteAddForm />
  </div>
);

export default inject(['NotesList', 'NoteAddForm'])(App);
```

Next, add a mutation template to attach to the form.

```js
// my-module/client/src/state/createNote.js
import { graphqlTemplates } from 'lib/Injector';

const { CREATE } = graphqlTemplates;
const mutation = {
  apolloConfig: {
    props({ mutate }) {
      return {
        onAdd: (content) => {
          mutate({
            variables: {
              input: {
                content,
              }
            }
          });
        }
      };
    }
  },
  templateName: CREATE,
  singularName: 'Note',
  pagination: false,
  fields: [
    'content',
    'id'
  ],
};

export default mutation;
```

It looks like a lot of code, but if you're familiar with Apollo mutations, this is pretty standard. The supplied `mutate()` function gets mapped to a prop - in this case `onAdd`, which the `AddForm` component is configured to invoke. We've also supplied the `singularName` as well as the template `CREATE` for the `createNote` scaffolded mutation.

And make sure we're exposing the mutation in our GraphQL schema:

```yml
# my-module/_graphql/models.yml
App\Model\Note:
  #...
  operations:
    #...
    create: true
```

Lastly, let's just register all this with `Injector`.

```js
// my-module/client/src/boot/registerDependencies.js
import AddForm from '../components/AddForm';
import createNote from '../state/createNote';
// ...

const registerDependencies = () => {
  // ...
  Injector.component.register('NoteAddForm', AddForm);
  Injector.query.register('CreateNote', createNote);

  // ...
  Injector.transform(
    'notesaddform-graphql',
    (updater) => {
      updater.component('NoteAddForm', injectGraphql('CreateNote'));
    }
  );
};

export default registerDependencies;
```

This is exactly the same pattern as we did before with a query, only with different components and GraphQL abstractions this time. Note that even though `CreateNote` is a mutation, it still gets registered under `Injector.query` for simplicity.

### Extending mutations

Now let's switch back to the project where we're customising the Notes application. The developer is going to want to ensure that users can supply a "Priority" value for each note entered. This will involve updating the `AddForm` component.

```js
// app/client/src/transformAddForm.js
import React, { useRef } from 'react';

const transformAddForm = () => ({ onAdd }) => {
  const contentRef = useRef(null);
  const priorityRef = useRef(null);
  return (
    <div>
      <label>Note content</label>
      <input type="text" ref={contentRef}/>
      <label>Priority</label>
      <select ref={priorityRef}>
        <option value="0">Low</option>
        <option value="1">Medium</option>
        <option value="2">High</option>
      </select>
      <button onClick={(e) => {
        e.preventDefault();
        if (contentRef && priorityRef) {
          onAdd(contentRef.value, Number(priorityRef.value));
        }
      }}
      >Add</button>
    </div>
  );
};

export default transformAddForm;
```

We're now passing two arguments to the `onAdd` callback - one for the note content, and another for the priority. We'll need to update the mutation to reflect this.

```js
// app/client/src/transformCreateNote.js
const transformCreateNote = (manager) => {
  manager.addField('priority');
  manager.transformApolloConfig('props', ({ mutate }) => (prevProps) => {
    const onAdd = (content, priority) => {
      mutate({
        variables: {
          input: {
            // Don't forget to keep the content variable in here!
            content,
            priority,
          }
        }
      });
    };

    return {
      ...prevProps,
      onAdd,
    };
  });
};

export default transformCreateNote;
```

All we've done here is overridden the `props` setting in the `CreateNote` apollo config. Recall from the previous section that it maps the `mutate` function to the `onAdd` prop. Since we've changed the signature of that function, we need to override the entire prop.

Now we just need to register these transforms, and we're done!

```js
// app/client/src/boot.js
import transformAddForm from './transformAddForm';
import transformCreateNote from './transformCreateNote';
// ...

Injector.transform(
  'noteslist-query-extension',
  (updater) => {
    // ...
    updater.component('NoteAddForm', transformAddForm);
    updater.query('CreateNote', transformCreateNote);
  },
  { after: ['noteslist-graphql', 'notesaddform-graphql'] }
);
```
