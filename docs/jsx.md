---
id: jsx
title: JSX
---

Reason comes with the [JSX](https://reasonml.github.io/guide/language/jsx) syntax! ReasonReact transforms it from an agnostic function call into a ReasonReact-specific call through a macro. To take advantage of ReasonReact JSX, put `{"reason": {"react-jsx": 2}` in your [`bsconfig.json`](http://bucklescript.github.io/bucklescript/Manual.html#_bucklescript_build_system_code_bsb_code) (schema [here](http://bucklescript.github.io/bucklescript/docson/#build-schema.json)).

**Note that due to current syntax constraints, you need to put spaces around the JSX children**: `<div> foo </div>`.

### Uncapitalized

```reason
<div foo={bar}> {child1} {child2} </div>
```

transforms into

```reason
ReactDOMRe.createElement("div", ~props=ReactDOMRe.props(~foo=bar, ()), [|child1, child2|]);
```

which compiles to the JS code:

```js
React.createElement('div', {foo: bar}, child1, child2)
```

Prop-less `<div />` transforms to:

```reason
ReactDOMRe.createElement("div", [||]);
```

Which compiles to

```js
React.createElement('div', undefined)
```

**Note that `ReactDOMRe.createElement` is intended for internal use by the JSX transform**.

### Capitalized

```reason
<MyReasonComponent key={a} ref={b} foo={bar} baz={qux}> {child1} {child2} </MyReasonComponent>
```

transforms to

```reason
ReasonReact.element(
  ~key=a,
  ~ref=b,
  MyReasonComponent.make(~foo=bar, ~baz=qux, [|child1, child2|])
);
```

Prop-less `<MyReasonComponent />` transforms to:

```reason
ReasonReact.element(MyReasonComponent.make([||]));
```

The `make` above is exactly the `make` function you've seen in the previous section.

**Note how `ref` and `key` have been lifted out of the JSX call into the `ReasonReact.element` call**. `ref` and `key` are reserved in ReasonReact, just like in ReactJS. **Don't** use them as props in your component!

### Children

ReasonReact children are **fully typed**, and you can pass any data structure to it (as long as the receiver component permits it). When you write:

```reason
<MyForm> <div /> <div /> </MyForm>
```

You're effectively passing the array `[| <div />, <div /> |]` to `MyForm`'s children. But this also means that the following wouldn't work:

```reason
let theChildren = [| <div />, <div /> |];
<MyForm> theChildren </MyForm>
```

Because this actually translates to:

```reason
let theChildren = [| <div />, <div /> |];
ReasonReact.element(
  MyReasonComponent.make([|theChildren|])
);
```

Which wraps the already wrapped `theChildren` in another layer of array. To solve this issue, Reason has a special [children spread syntax](https://reasonml.github.io/guide/language/jsx#children-spread):

```reason
let theChildren = [| <div />, <div /> |];
<MyForm> ...theChildren </MyForm>
```

This simply passes `theChildren` without array wrapping. It becomes:

```reason
let theChildren = [| <div />, <div /> |];
ReasonReact.element(
  MyReasonComponent.make(theChildren)
);
```

For more creative way of leveraging Reason's type system, data structures and performance to use `children` to its full potential, see the [Children section](children.md)!
