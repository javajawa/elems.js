<!--
SPDX-FileCopyrightText: 2020 Benedict Harcourt

SPDX-License-Identifier: BSD-2-Clause
-->

# elems.js

![JS Lint](https://github.com/javajawa/elems.js/workflows/JS%20Lint/badge.svg)
![SPDX Lint](https://github.com/javajawa/elems.js/workflows/SPDX%20Lint/badge.svg)
![JS Minification](https://github.com/javajawa/elems.js/workflows/JS%20Minification/badge.svg)

`elems.js` is a lightweight DOM building library for JavaScript, implemented
as a ES6 module.
The core functionality is a function-generator, which creates functions to
generate specific DOM elements, such as `<div>`. Each of these can be called
with any number of parameters, the types of which will be used to determine
how they are interpreted

## Contents

- [elems.js](#elemsjs)
  * [Generating elem.js Functions](#generating-elemjs-functions)
    + [Using `elemGenerator`](#using-elemgenerator)
      - [Example 1: How to use `elemGenerator`](#example-1-how-to-use-elemgenerator)
    + [Using `elemRegister`](#using-elemregister)
      - [Example 2: How to use `elemRegister`](#example-2-how-to-use-elemregister)
  * [Using the `elems.js` generated functions.](#using-the-elemsjs-generated-functions)
    + [Example 3: A Quick List (`elemGenerator`)](#example-3-a-quick-list-elemgenerator)
    + [Example 4: A Quick List (`elemRegister`)](#example-4-a-quick-list-elemregister)
    + [Example 5: Prefix-less SVG Element Generators](#example-5-prefix-less-svg-element-generators)
    + [Example 6: Unpacking Arrays](#example-6-unpacking-arrays)
  * [Using `documentFragment` for fragments](#using-documentfragment-for-fragments)
    + [Exmaple 8: Using `documentFragment` to reduce DOM operations](#exmaple-8-using-documentfragment-to-reduce-dom-operations)
    + [Example 9: Using `elem.js` for Templates](#example-9-using-elemjs-in-custom-element-templates)

## Generating elem.js Functions

Two methods are provided for creating the `elem.js` functions:

 - `elemGenerator` returns a single function which you can store and use.
 - `elemRegister` creates a list of functions and puts them into the global namespace.


### Using `elemGenerator`

`elemGenerator` is a function-generator which outputs functions
for generating DOM elements via a more concise interface.

Calls to `elemGenerator` specify a tag name and, optionally, an XML namespace
and yield a function that will generate DOM elements of that type.

```
elemGenerator( tagName[, XMLnamespace] ) -> function(...)
```

#### Example 1: How to use `elemGenerator`

```js
// Import the function-generator
import { elemGenerator } from './elems.js';

// Generate a function to call
const p = elemGenerator( 'p' );

// Create a paragraph and add it to the page.
document.body.appendChild(
	p( 'Hello I am a new paragraph' )
);
```

### Using `elemRegister`

`elemRegister` is a helper function to register `elemGenerator` functions in
the global (`window`) scope, with an optional prefix. All the returned functions
will have the same XML namespace.

```
elemRegister( globalFunctionPrefix, XMLnamespace, tagName[, tagName...] ) -> void
```

#### Example 2: How to use `elemRegister`

```js
// Import the function-generator
import { elemRegister } from './elems.js';

// Register some functions
elemRegister( '_', null, 'p', 'div' );

document.body.appendChild(
  _div(
    _p( 'Hello I am a new paragraph in a div' )
  )
);
```

## Using the `elems.js` generated functions.

The generated functions takes any number of arguments of different
types, and builds up the element according to them.

*Arrays*:
  Any arrays supplied in the argument list are treated as additional
  arguments. This has a maximum nested of three layers deep.

*Strings*:
  Strings are converted to text nodes in the document, and added to
  the element.

*Nodes/Elements/Attrs*:
  Other DOM Nodes, including elements and attributes, supplied to this
  function will be added to the element via `appendChild`/`setAttribute`
  as appropriate to the type.

*Objects*:
  Objects with iterable keys will be treated as attribute and event maps.

  Values which are functions are treated as event handlers. This uses
  `addEventListener`, so the `on` part of the event name should not be
  a part of the key.

  Other values are are coerced to strings and treated as attributes, with
  a few exceptions:
   - `null` causes no action to happen.
   - a boolean causes that value to be set without a value (true) or,
     the attribute to be removed (false).
   - objects and array cause an exception.

Providing any other type in an argument will result in an exception being thrown.

### Example 3: A Quick List (`elemGenerator`)
[See Demo](examples/simple-list.html)

In this example, we're going to create a `<ul>` with four items, which have
example of text, event handling, styling, and nesting.

```js
import { elemGenerator } from './elems.js';

const ul = elemGenerator('ul')
const li = elemGenerator('li')

document.body.appendChild( ul(
  li( 'Hello' ),
  li( 'Click Me', { click: e => alert('Hi') } ),
  li( 'Don\'t Click Me', { click: e => alert('I said no'), style: 'color: red;' } ),
  li( 'Sub Set', ul (
    li( 'One', { 'data-foo': '1' } ),
    li( 'Two' ),
    li( 'Three' )
  ) )
) );
```

### Example 4: A Quick List (`elemRegister`)
[See Demo](examples/global-list.html)

Here we shall register global function to generate `<ul>`s and `<li>`s.

```js
import { elemRegister } from './elems.js';

elemRegister( '_', null, 'ul', 'li' );

_ul(
  { style: 'color: red' },
  _li( 'Hello' ),
  _li( 'Click Me', { click: e => alert('Hi' ) } )
)
```

### Example 5: Prefix-less SVG Element Generators
[See Demo](examples/svg.html)

```js
import { elemRegister } from './elems.js';

elemRegister( '', 'http://www.w3.org/2000/svg', 'svg', 'path', 'text' );

svg(
  { viewBox: "0 0 200 200" },
  path( { d: 'M10,10h180v180h-180v-180', stroke: 'red' } ),
  text( { x: '100', y: '80' }, 'Hello' )
)
```

### Example 6: Unpacking Arrays
[See Demo](examples/array-map.html)

Arrays passed to `elem.js` generated functions are automatically flattened
and treated as a list of parameters (to maximum depth of 3).
This allows you to avoid intermediate representations of various other params.

Note that we can not just for `items.map(li)`, as this will call `li` with two
parameters, the second one being the numeric index. Numbers are not a valid
parameter type, and will cause an error to be thrown.

```js
import { elemGenerator } from '../elems.js';

const ul = elemGenerator( 'ul' );
const li = elemGenerator( 'li' );

const items = [
	'Item 1',
	'Item 2',
	'Item 3'
];

document.body.appendChild(
	ul( items.map( i => li(i) ) )
);
```

## Using `documentFragment` for fragments

`elems.js` provides a wrapper function to create the elements within a
[Document Fragment](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment),
which can combined with custom elements, templates, or to reduce DOM operations.

This is a stand-alone function which returns a document fragment.
It takes an array of elements as its arguments, which may be the output
of other `elems.js` calls, or existing elements.

### Exmaple 8: Using `documentFragment` to reduce DOM operations
(No demo is supplied for this example)

One use of a fragment is to prepare a list of nodes that you wish to add to
the DOM at one time. DOM operations can be expensive, and so grouping these
may be beneficial.

This example demonstrates a AJAX call which appends to an existing items to
and existing list, using only one append on the actual document.

```js
import { elemGenerator, documentFragment } from '../elems.js';

const li = elemGenerator( 'li' );

fetch( '/data-source' ).then( r => r.json() )
	.then( items => documentFragment( items.map( item => li( item ) ) ) )
	.then( fragment => document.getElementById( 'list' ).appendChild( fragment ) );
```

### Example 9: Using `elem.js` for Templates
(No demo is provided for this example)

A document fragment can be used as a generic pre-built template.
In this example, we define a new custom element which uses a document fragment
to populate its shadow root.

```js
const h1 = elemGenerator( 'h1' );
const slot = elemGenerator( 'slot' );

const template = documentFragment(
	h1( 'the title' )
	slot(),
);

class MyParagraph extends HTMLElement {
	constructor() {
		super();
		this.attachShadow( { mode: 'open' } );
		this.shadowRoot.appendChild( template.cloneNode( true ) );
	}
};

```
