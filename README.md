<img src="https://travis-ci.org/Ravenstine/ember-custom-elements.svg?branch=master">

Ember Custom Elements
=====================

The most flexible way to render parts of your Ember application using custom elements!


## Compatibility

* Ember.js v3.6 or above
* Ember CLI v2.13 or above
* Node.js v10 or above

This add-on almost certainly won't work with versions of `ember-source` prior to `3.6.0`.  I will not be trying to get this to work with earlier versions, but I'm open to any pull requests that improve backward compatibility.


## Installation

```
ember install ember-custom-elements
```


## Usage



### Components

All you have to do is use the `customElement` decorator in your component file:

```javascript
import Component from '@glimmer/component';
import { customElement } from 'ember-custom-elements';

@customElement('my-component')
export default MyComponent extends Component {

}
```

Now you can use your component _anywhere_ inside the window that your app was instantiated within by using your custom element:

```handlebars
<my-component></my-component>
```




#### Attributes and Arguments

Attributes instances of your custom element are translated to arguments to your component:

```handlebars
<my-component some-message="hello world"></my-component>
```

To use the attribute in your component template, you would use it like any other argument:

```handlebars
{{!-- my-component.hbs --}}
{{@some-message}}
```

Changes to attributes are observed, and so argument values are updated automatically.




#### Block Content

Block content inside your custom element instances can be treated just like block content within a precompiled template.  If your component contains a `{{yield}}` statement, that's where the block content will end up.

```handlebars
{{!-- my-component.hbs --}}
<span>foo {{yield}} baz</span>
```

```handlebars
<my-component>bar</my-component>
```

When the component is rendered, we get this:

```handlebars
<span>foo bar baz</span>
```

Block content can be dynamic, so if for whatever reason your block content is bound to another component, the content will behave as expected.



### Routes

The `@customElement` decorator can define a custom element that renders an active route, much like the `{{outlet}}` helper does.  In fact, this is achieved by creating an outlet view that renders the main outlet for the route.

Just like with components, you can use it directly on your route class:

```javascript
/* app/routes/posts.js */

import Route from '@ember/routing/route';
import { customElement } from 'ember-custom-elements';

@customElement('test-route')
export default class PostsRoute extends Route {
  model() {
    ...
  }
}
```

In this case, the `<test-route>` element will render your route when it has been entered in your application.




#### Named Outlets

If your route renders to [named outlets](https://api.emberjs.com/ember/release/classes/Route/methods/renderTemplate?anchor=renderTemplate), you can define custom elements for each outlet with the `outletName` option:

```javascript
/* app/routes/posts.js */

import Route from '@ember/routing/route';
import { customElement } from 'ember-custom-elements';

@customElement('test-route')
@customElement('test-route-sidebar', { outletName: 'sidebar' })
export default class PostsRoute extends Route {
  model() {
    ...
  }

  renderTemplate() {
    this.render();
    this.render('posts/sidebar', {
      outlet: 'sidebar'
    });
  }
}
```

In this example, the `<test-route-sidebar>` element exhibits the same behavior as `{{outlet "sidebar"}}` would inside the parent route of the `posts` route.  Notice that the `outletName` option reflects the name of the outlet specified in the call to the `render()` method.




#### Outlet Element

This add-on comes with a primitive custom element called `<ember-outlet>` which can allow you to dynamically render outlets, but with a few differences from the `{{outlet}}` helper due to technical limitations from rendering outside of a route hierarchy.




##### Usage

The outlet element will not be defined by default.  You must do this yourself somewhere in your code:

```javascript
import { EmberOutletElement } from 'ember-custom-elements';

window.customElements.define('ember-outlet', EmberOutletElement);
```

This will allow you to render an outlet like this:

```handlebars
<ember-outlet></ember-outlet>
```

By default, the `<ember-outlet>` will render the main outlet for the `application` route.  This can be useful for rendering an already initialized Ember app within other contexts.

To render another route, you must specify it using the `route=` attribute:

```handlebars
<ember-outlet route="posts.index"></ember-outlet>
```

If your route specifies named routes, you can also specify route names:

```handlebars
<ember-outlet route="posts.index" name="sidebar"></ember-outlet>
<ember-outlet route="posts.index" name="content"></ember-outlet>
```

Since an `<ember-outlet>` can be used outside of an Ember route, the route attribute is required except if you want to render the application route.  You cannot just provide the `name=` attribute and expect it to work.

In the unusual circumstance where you would be loading two or more Ember apps that use the `ember-outlet` element on the same page, you can extend your own custom element off the `ember-outlet` in order to resolve the naming conflict between the two apps.



### Applications

You can use the same `@customElement` decorator on your Ember application.  This will allow an entire Ember app to be instantiated and rendered within a custom element as soon as that element is connected to a DOM.

Presumably, you will only want your Ember app to be instantiated by your custom element, so you should define `autoboot = false;` in when defining your app class, like so:

```javascript
/* app/app.js */

import Application from '@ember/application';
import Resolver from 'ember-resolver';
import loadInitializers from 'ember-load-initializers';
import config from './config/environment';
import { customElement } from 'ember-custom-elements';

@customElement('ember-app')
export default class App extends Application {
  modulePrefix = config.modulePrefix;
  podModulePrefix = config.podModulePrefix;
  Resolver = Resolver;
  autoboot = false;
}

loadInitializers(App, config.modulePrefix);
```

Once your app has been created, every creation of a custom element for it will only create new application instances, meaning that your instance-initializers will run again but your initializers won't perform again.  Custom elements for your app are tied directly to your existing app.



### Options

At present, there are a few options you can pass when creating custom elements:

- **extends**: A string representing the name of a native element your custom element should extend from.  This is the same thing as the `extends` option passed to [window.customElements.define()](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements#High-level_view).
- **useShadowRoot**: By default, custom elements created for your components use a shadow root.  If you set this option to `false`, a shadow root will not be used.
- **observedAttributes**: A whitelist of which element attributes to observe.  This sets the native `observedAttributes` static property on [custom elements](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements).  It's suggested that you only use this option if you know what you are doing, as once the `observedAttributes` are set on a defined custom element, it cannot be changed after the fact(remember that custom elements can be only defined once).  The most common reason to define `observedAttributes` would be for performance reasons, as making calls to JavaScript every time any attribute changes is more expensive than if only some attribute changes should call JavaScript.  All that said, you probably don't need this, as ember-custom-elements observes all attribute changes by default.  Does nothing for custom elements that instantiate Ember apps.
- **customElementClass**: In the extreme edge case that you need to redefine the behavior of the custom element class itself, you can `import { EmberCustomElement } from 'ember-custom-elements';`, extend it into a subclass, and pass that subclass to the `customElementClass` option.  This is definitely an expert tool and, even if you think you need this, you probably don't need it.  This is made available only for the desperate.  The `EmberCustomElement` class should be considered a private entity.
- **camelizeArgs**: Element attributes must be kabob-case, but if `camelizeArgs` is set to true, these attributes will be exposed to your components in camelCase.
- **outletName**: (routes only) The name of an outlet you wish to render for a route.  Defaults to 'main'.  The section on [named outlets][#named-outlets] goes into further detail.




#### Options Example

```javascript
@customElement('my-component', { extends: 'p', useShadowRoot: false })
export default MyComponent extends Component {

}
```


## Notes



### Elements

Once a custom element is defined using `window.customElements.define`, it cannot be redefined.

This add-on works around that issue by changing the definition of the element class that it already defined.  It's necessary in order for application and integration tests to work without encountering errors.  This behavior will only be applied to custom elements defined using this add-on.  If you try to define an application component on a custom element defined outside of this add-on, an error will be thrown.



### Runloop

Because element attributes must be observed, the argument updates to your components occur asynchronously.  Thus, if you are changing your custom element attributes dynamically, your tests will need to use `await settled()`.


## Contributing

See the [Contributing](CONTRIBUTING.md) guide for details.


## License

This project is licensed under the [MIT License](LICENSE.md).
