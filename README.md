# elems.js

`elems.js` is a lightweight DOM building library for JavaScript, implemented
as a ES6 module.
The core functionality is a function-generator, which creates functions to
generate specific DOM elements, such as `<div>`. Each of these can be called
with any number of parameters, the types of which will be used to determine
how they are interpreted

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

Arrays:
  Any arrays supplied in the argument list are treated as additional
  arguments. This has a maximum nested of three layers deep.

Strings:
  Strings are converted to text nodes in the document, and added to
  the element.

Nodes/Elements/Attrs:
  Other DOM Nodes, including elements and attributes, supplied to this
  function will be added to the element via `appendChild`/`setAttribute`
  as appropriate to the type.

Objects:
  Objects with iterable keys will be treated as attribute and event maps.
  Values which are strings are treated as attributes, values that are
  functions are treated as event handlers.

Providing any other type in an argument will result in an exception being thrown.

### Example 3 - A Quick List (`elemGenerator`)

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

### Example 4 - A Quick List (`elemRegister`)

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

```js
import { elemRegister } from './elems.js';

elemRegister( '', 'http://www.w3.org/2000/svg', 'svg', 'path', 'text' );

svg(
  { viewBox: "0 0 200 200" },
  path( { d: 'M10,10h180v180h-180v-180', stroke: 'red' } ),
  text( { x: '100', y: '80' }, 'Hello' )
)
```

