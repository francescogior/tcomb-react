% tcomb-react

![tcomb logo](http://gcanti.github.io/resources/tcomb/logo.png)

This library allows you to add type checking to React components. You can check all props, including the children. You can even check the quality and quantity of nested components.

# Playground

If you want to see this library in action, try the playground [here](https://gcanti.github.io/resources/tcomb-react-bootstrap/playground/playground.html)

# Contents

- [How to add type checking while prototyping](#prototyping)
- [How to write safe components](#safe-components)
- [Advanced usage](#advanced-usage)

# Prototyping

To add a throwaway type checking while prototyping, add some raw asserts to your `render` methods.

```js
assertEqual(props, type, [opts])
```
- `props` component props
- `type` a `struct` or a `subtype` of a `struct`
- `opts` see [Options](#options)

If you pass a wrog prop to the component **the debugger kicks in** so you can inspect the stack and quickly find out what's wrong, then it throws an error with a descriptive message.

## Example

Let's build a simple React component with a fancy spec, the component must have:

- only a `href` prop and it must be a string starting with `#`
- only one child and it must be a string

```js
var t = require('tcomb-react');
var Str = t.Str;          // the string type
var subtype = t.subtype;  // build a subtype
var struct = t.struct;    // build a struct (i.e. a class)

// a predicate is a function with signature (x) -> boolean
var predicate = function (s) { return s.substring(0, 1) === '#'; };

// the `href` spec
var Href = subtype(Str, predicate);

// the props spec
var AnchorProps = struct({
  href: Href,
  children: Str
});

var Anchor = React.createClass({
  render: function () {
    // add this assert and you are done
    t.react.assertEqual(this.props, AnchorProps);
    return (
      <a href={this.props.href}>{this.props.children}</a>
    );
  }
});
```

here some results

```js
// OK
React.renderComponent(
  <Anchor href="#section">content</Anchor>
, mountNode);

// KO, href is missing
React.renderComponent(
  <Anchor>content</Anchor>
, mountNode);

// KO, content is missing
React.renderComponent(
  <Anchor href="#section"></Anchor>
, mountNode);

// KO, href is wrong
React.renderComponent(
  <Anchor href="http://mydomain.com">content</Anchor>
, mountNode);

// KO, content should be a string not a span
React.renderComponent(
  <Anchor href="#section"><span>content</span></Anchor>
, mountNode);
```

## Options

### opts.strict

If set to `true`, disallows unspecified properties (default `false`).

```js
t.react.assertEqual(this.props, Props, {strict: true});

...

// KO, forbidden `title` attribute
React.renderComponent(
  <Anchor href="#section" title="mytytle">content</Anchor>
, mountNode);
```

# Safe components

To handle third party components or if you want a more fine-grained control you can `bind` your component to a type. 
The function `bind` returns a proxy factory with the same interface of the original factory but with asserts included.
In production you can choose to switch from the proxy factory to the original one.

```js
bind(factory, type, opts)
```

- `factory` a React component factory
- `type` a `struct` or a `subtype` of a `struct`
- `opts` see [Options](#options)

## Workflow

### 1. Define your component as usual

```js
// unsafe-component.js
var Anchor = React.createClass({
  render: function () {
    return (
      <a href={this.props.href}>{this.props.children}</a>
    );
  }
});

module.exports = Anchor;
```

### 2. Define the type of the props and `bind`

```js
// safe-component.js
var AnchorProps = struct({
  href: Href,
  children: Str
});

var Anchor = require('unsafe-component.js');

module.exports = t.react.bind(Anchor, AnchorProps);
```

### 3. Safely use the proxy factory

```js
// app.js

var Anchor = require('safe-component.js')

// KO, href is missing
React.renderComponent(
  <Anchor>content</Anchor>
, mountNode);
```

You can find extensive examples of the `bind` method in the [tcomb-react-bootstrap](https://github.com/gcanti/tcomb-react-bootstrap) project.

# Advanced usage

This library exports a bunch of common React types:

## Tags

For each `React.DOM` factory, there is a type in the `DOM` namespace.
Say you want modify the first example to accept only a `span` child:

```js
var Props = struct({
  href: Href,
  children: t.react.DOM.Span
});

// OK
React.renderComponent(
  <Anchor href="#section"><span>content</span></Anchor>
, mountNode);

// KO
React.renderComponent(
  <Anchor href="#section">content</Anchor>
, mountNode);
```

## Misc

- `Component`: represents a component, i.e. an instance of a React factory
- `Key`: represents the `key` attribute
- `Mountable`: represents a mounting node
- `Ref`: represents the `ref` attribute
- `Renderable`: represents everything renderable in a component
