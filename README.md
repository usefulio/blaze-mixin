# Blaze Mixin

Since around meteor 1.0 there's been pretty powerful support for building reusable components with Blaze. This package makes it easy to use that foundation to build flexible & reusable components.

**Work In Progress - This package is nothing but a readme right now.**

## API

### Component Constructor
Create a new component using the constructor.

- `new Component(name, options)` Create a new component
    + `name` The name of your component, this name is used to distinguish your component from other components, and also to namespace component helpers and varaibles if you pass the namespace option.
    + `options` Options for your component, currently we only accept the namespace option
    + `options.namespace` Pass true (the default) to ensure that your component instance is namespaced which greatly reduces possible name conflicts. If true, the component instance will be accessible as `Template.instance()[component.name]` and component helpers will be accessible as `{{componentName.helperName}}`

```js
Form = new Component('form');
```

### Component Class
In instance of the Component Class represents a single reusable component, you can use it to create instances of your component, and it exposes methods which you can use to extend the component.

- `component.mixin(templateOrComponent)` Extend a template or component with your component's behavior.

```js
Form.mixin(Template.myForm);
```

- `component._extendInstance(templateInstance)` Extend a template instance with your component's behavior (this method is called by component.mixin).

- `component.constructor(options)` Create an instance of the component, you can replace this constructor or extend it's prototype.

*Note: We recommend attaching functions using `component.helpers` or `component.methods` instead of extending the constructor prototype.*

*Question: should we provide a default constructor for each component, or should we leave the constructor empty, and if there is no constructor, use a plain object?*

```js
Form.constructor.prototype.getValue = function (name) {
    return this.doc.get()[name];
};
```

- `component.onCreated(fn)` Register a function to be passed to `template.onCreated`

```js
Form.onCreated(function () {
   // Template.instance() === this 
});
```

- `component.onRendered(fn)` Register a function to be passed to `template.onRendered`

```js
Form.onRendered(function () {
   // Template.instance() === this 
});
```

- `component.helpers(helpers)` Register helpers that should be available on the template instance & in extended templates as blaze helpers

```js
Form.helpers({
    doc: function () {
        // this is the component instance, not the template instance
        return this.doc.get();
    }
});
```

- `component.methods(methods)` Register methods that should be available on the template instance but not as blaze helpers. If you register a method & a helper with the same name, the helper will be called from your template code, and the method will be available on your component instance.

```js
Form.methods({
    set: function (name, val) {
        //...
    }
});
```

- `component.instance()` Get the current instance of your component. Basically this is a shortcut for `Template.instance()[componentName]` but we travel up the template parents chain if necessary.

```js
Template.myForm.helpers({
    isValid: function () {
        var form = Form.instance();
        return form && form.isValid();
    }
});
```

- `component.attachGlobalHelper()` Attach a global helper for this component which will make component helpers available in child templates.

**Note: We recommend that component authors do not call `attachGlobalHelper`, instead only call `attachGlobalHelper` when you write a template which depends on it.**

```js
Form.attachGlobalHelper();
Form.mixin(Template.myForm);
```

```html
<template name="myForm">
    {{> nameField}}
</template>
<template name="nameField">
    <!--
    Works because form is a global helper which returns the nearest form
    component instance
    -->
    {{form.doc.name}}
</template>
```

### Notes
- Component instance helpers & methods should be bound to the current instance.
- We should implement namespaced components first, and later implement un-namespaced components

