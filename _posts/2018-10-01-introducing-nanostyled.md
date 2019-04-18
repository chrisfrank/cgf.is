---
title: "Introducing nanostyled: CSS-in-JS without CSS-in-JS"
published: true
description: Style React components as if with CSS-in-JS, without any CSS-in-JS
tags: react, css, css-in-js, styled-components, introduction
---

Nanostyled is a tiny library (< 1 Kb unminified) for building styled React components. It tries to combine the flexible, component-based API of CSS-in-JS libraries with the extremely low overhead of plain CSS:

|            | Low overhead | Flexible, component-based API |
|------------|--------------|------------------------------|
| Plain CSS  | ✅           | ❌                           |
| CSS-in-JS  | ❌           | ✅                           |
| nanostyled | ✅           | ✅                           |

Like the CSS-in-JS libraries that inspired it -- 💕 to [styled-components][styled-components] -- nanostyled lets you build UI elements with complex default styles, then tweak those styles throughout your app via props:

```html
<Button>A nice-looking button</Button>
<Button color="blue">A nice-looking button that is blue.</Button>
```

_Unlike_ a CSS-in-JS library, nanostyled doesn't use any CSS-in-JS. Instead, it's designed to accompany a **functional CSS framework** like [Tachyons][tachyons] or [Tailwind][tailwind]. Nanostyled makes functional CSS less verbose, and easier to extract into props-controlled components.

Check out [nanostyled on npm][nanostyled] for installation and usage instructions, or read on for more context.

---

## Functional CSS?

The basic premise of a functional CSS framework is that you can build complex styles by composing tiny CSS utility classes.

A button styled with Tachyons might look like this in markup:

```html
<button class="bg-blue white br2 pa2">Button</button>
```

That's a button with a blue background, white text, rounded corners (`br2`), and some padding on all sides (`pa2`).

> holy hell this is the worst thing I've ever seen
> -Adam Wathan, author of [Tailwind.css][tailwind]

It's true. Functional CSS is ugly, and defies decades-old best practices re: separating content from styling.

On the other hand, styling with functional CSS scales well across large projects, enforces visual consistency, and makes it easy to build new UI elements without writing any new CSS. Adam Wathan, creator of, Tailwind, [defends the approach elegantly here][adam].

Nanostyled makes functional CSS easier to abstract into components, without giving up any of its strengths.

## Why building flexible components with functional CSS in React is hard

To make working with functional CSS less verbose, you can extract long class strings into self-contained React components:

```jsx
const Button = ({ className = '', ...rest }) => (
  <button className={`bg-blue white br3 pa2 fw7 ${className}`} {...rest} />
)
```

The problem, in this case, is that there's no good way to render our `<Button>` with a different background color. Even though it accepts a `className` prop, writing `<Button className="bg-red" />` _won't necessarily render a red button._

Max Stoiber's recent [Twitter poll][poll] is a good illustration of why:

> How well do you know CSS? Given these classes:
> `.red { color: red; }`
> `.blue { color: blue; }`
> Which color would these divs be?
> `<div class="red blue">`
> `<div class="blue red">`

The correct answer, which 57% of respondents got wrong, is that both divs would be blue.

You can't know the answer by looking at just the HTML. You need to look at the CSS, because when two conflicting CSS classes have the same specificity, their order *in the markup* is irrelevant. Which class wins depends on which one is defined last *in the stylesheet*.

So to build a robust `<Button>` with functional CSS, we need to be able to

1. Declare some stock CSS classes that style it
2. Expose a convenient API for *replacing* some of the stock classes with alternatives

This second requirement is key for avoiding counterintuitive class collisions like in Max's poll, and it's the thing nanostyled makes easy.

## Building flexible components with nanostyled and style props

Nanostyled works by mapping **style props** onto class names from your functional CSS framework of choice. 

**Style props** can be named whatever you like, and can each hold any number of CSS classes:

### A nanostyled Button

```jsx
import nanostyled from 'nanostyled';
// This example uses CSS classes from Tachyons
import 'tachyons/css/tachyons.css';

// A Button with three style props:
const Button = nanostyled('button', {
  color: 'white',
  bg: 'bg-blue',
  base: 'fw7 br3 pa2 sans-serif f4 bn input-reset'
});

const App = () => (
  <div>
    <Button>Base Button</Button>
    <Button bg="bg-yellow">Yellow Button</Button>
  </div>
);

/* rendering <App /> produces this markup:
<div>
  <button class="white bg-blue fw7 br3 pa2 sans-serif f4 bn input-reset">Base Button</button>
  <button class="white bg-yellow fw7 br3 pa2 sans-serif f4 bn input-reset">Yellow Button</button>
</div>
*/
```

When a `nanostyled(element)` renders, it consumes its style props and merges them into an HTML class string, as per above.

It's totally to you which style props to use. The `<Button>` above has an API that would make it easy to restyle color or background-color via the `color` and `bg` props, but hard to change other styles without totally rewriting the `base` prop.

### A more flexible nanostyled Button

By using more style props, we can make a more flexible button:

```jsx
import nanostyled from 'nanostyled';
import 'tachyons/css/tachyons.css';

const FlexibleButton = nanostyled('button', {
  color: 'white', // white text
  bg: 'bg-blue', // blue background
  weight: 'fw7', // bold font
  radius: 'br3', // round corners
  padding: 'pa2', // some padding
  typeface: 'sans-serif', // sans-serif font
  fontSize: 'f4', // font size #4 in the Tachyons font scale
  base: 'bn input-reset', // remove border and appearance artifacts
});
```

Rendering a stock `<FlexibleButton />` will produce the same markup as its simpler relative. But it's much easier to render alternate styles:

```jsx
<FlexibleButton
  bg="bg-light-green"
  color="black"
  weight="fw9"
  radius="br4"
>
  Button with a green background, black text, heavier font, and rounder corners
</FlexibleButton>
```

When you need a variation that you didn't plan for in your style props, you can still use the `className` prop:

```jsx
<FlexibleButton className="dim pointer">
  A button that dims on hover and sets the cursor to 'pointer'
</FlexibleButton>
```

## Sharing style props across multiple components

If you're building multi-component UI kits with nanostyled, I recommend sharing at least a few basic style props across all your components. Otherwise it gets hard to remember which components support, say, a `color` prop, and which ones don't.

I usually start here:

```jsx
import React from "react";
import ReactDOM from "react-dom";
import nanostyled from "nanostyled";
import "tachyons/css/tachyons.css";

// The keys in this styleProps will determine which style props
// our nanostyled elements will accept:
const styleProps = {
  bg: null,
  color: null,
  margin: null,
  padding: null,
  font: null,
  css: null
};

/* 
Why choose those keys, in particular? For everything except `css`, 
it's because the elements in the UI kit probably will have some default 
bg, color, margin, padding, or font we'll want to be able to easily override via props.

The `css` prop is an exception. I just like being able to use it instead of `className`.
*/

// Box will support all styleProps, but only use them when we explicitly pass values
const Box = nanostyled("div", styleProps);
/*
<Box>Hi!</Box>
renders <div>Hi!</div>

<Box color="red">Hi!</Box>
renders <div class="red">Hi!</div>
*/

// Button will also support all styleProps, and will use some of them by default
const Button = nanostyled("button", {
  ...styleProps,
  bg: "bg-blue",
  color: "white",
  padding: "pa2",
  font: "fw7",
  // I use a 'base' prop to declare essential component styles that I'm unlikely to override
  base: "input-reset br3 dim pointer bn"
});
/*
<Button>Hi!</Button>
renders
<button class="bg-blue white pa2 dim pointer bn input-reset>Hi!</button>
*/

// Heading uses styleProps, plus some extra props for fine-grained control over typography
const Heading = nanostyled("h1", {
  ...styleProps,
  size: "f1",
  weight: "fw7",
  tracking: "tracked-tight",
  leading: "lh-title"
});

// Putting them all together....
const App = () => (
  <Box padding="pa3" font="sans-serif">
    <Heading>Styling with Nanostyled</Heading>
    <Heading tracking={null} tag="h2" size="f3" weight="fw6">
      A brief overview
    </Heading>
    <Heading tag="h3" weight="fw4" size="f5" tracking={null} css="bt pv3 b--light-gray">
      Here are some buttons:
    </Heading>
    <Button>Base Button</Button>
    <Button css="w-100 mv3" padding="pa3" bg="bg-green">
      Wide Green Padded Button
    </Button>
    <Box css="flex">
      <Button css="w-50" margin="mr2" bg="bg-gold">
        50% Wide, Gold
      </Button>
      <Button css="w-50" margin="ml2" bg="bg-red">
        50% wide, Red
      </Button>
    </Box>
  </Box>
);

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```

This full example is [available on CodeSandbox][codesandbox] for you to experiment with.

Nanostyled is available [on npm][nanostyled], and you can contribute to the library on [GitHub][source].

[nanostyled]: https://www.npmjs.com/package/nanostyled
[source]: https://www.github.com/chrisfrank/nanostyled
[poll]: https://mobile.twitter.com/mxstbr/status/1038073603311448064
[styled-components]: https://www.styled-components.com
[tachyons]: http://tachyons.io/
[tailwind]: https://tailwindcss.com/
[adam]: https://adamwathan.me/css-utility-classes-and-separation-of-concerns/
[codesandbox]: https://codesandbox.io/s/3r8l4rr8p1
[rebass]: https://rebassjs.org
