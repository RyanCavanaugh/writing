## How To Write UMD Module Definitions

The goal of this guide is to show you how to write a correct definition
  file for a UMD module.
A UMD module is one that can be loaded either through a module loader,
  for example requirejs, or put into the global scope through a script tag.
Many popular libraries are written in this way.

We will also show how to augment the definitions for a UMD module,
  and how to correctly consume the typing file for a UMD module.

For this example, we will write a definition for a fictional
  UMD library 'math-2d', which is available as an module, or
  loads in to the global `Math2d` when used in a script tag.

Here is how someone might use the library:
```ts
// When imported via a module loader
// Note that we can name it arbitrarily
import * as math2d from 'math-2d';

let v = new math2d.Vector(3, 2);
let magnitude = mathd2d.getLength(v);
let p: math2d.Point = v.translate(5, 5);
console.log('x coordinate is now ' + p.x);
```

```ts
// When in included in a script tag
let v = new Math2d.Vector(3, 2);
let magnitude = Mathd2d.getLength(v);
let p: Math2d.Point = v.translate(5, 5);
console.log('x coordinate is now ' + p.x);
```

## Writing the Definition File

There are a few "styles" that are supported.
If you are porting an existing definition file, you may want to choose
  the style that most closely matches the file so as to minimize breaking
  changes for users of that file.


### Style A: *top-level `export`*

This style exports functions, types, and variables from the top-level of the file.
This is the recommended style of definition file.

For our fictional `math-2d` library, we would write this code:
**typings/math-2d/index.d.ts**
```ts
export as namespace Math2d;

export interface Point {
	x: number;
	y: number;
}

export class Vector implements Point {
	x: number;
	y: number;
	constructor(x: number, y: number);

	translate(dx: number, dy: number): Vector;
}

export function getLength(p: Vector): number;
```

### Style B: *`export =`*

This style writes a `module` (or `namespace`) and uses `export = `
  to declare it as the module exported object.
This is not the recommended style, but may be an easier conversion target
  for declaration files written prior to the introduction of native UMD support.

For our fictional `math-2d` library, we would write this code:
**typings/math-2d/index.d.ts**
```ts
// This specifies the name when seen as a global
export as namespace Math2d;

// This namespace is the exported object
export = MathImpl;
declare namespace MathImpl {
    interface Point {
    	x: number;
    	y: number;
    }
    
    class Vector implements Point {
    	x: number;
    	y: number;
    	constructor(x: number, y: number);
    
    	translate(dx: number, dy: number): Vector;
    }
    
    function getLength(p: Vector): number;
}
```


## Augmenting the Definition

Let's now write a definition for a plugin for Math-2d.
The way we do this doesn't depend on how the original definition file was written.

The code for this plugin is simple:

***math-2d-reverse.js***
```ts
import Vector as vec from 'math-2d';
vec.prototype.reverse = function() {
	return new { x: -this.x, y: -this.y };
}
```

To write a definition file for the plugin, we'll need to add a member
  to the `Vector` class.

***math-2d-reverse.d.ts***
```ts
// Pull in the existing types from the library
import * as Math2d from 'math-2d';
// Augment the module. Note that this different from a normal
// top-level ambient module declaration -- it augments it instead.
declare module 'math-2d' {
	// Add a method to the class
	interface Vector {
		reverse(): Math2d.Point;
	}
}
```

This will cause the method to be valid both for module consumers (using `import`) and
  for code using the library as a global.
