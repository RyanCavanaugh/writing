## Caveats

There are subtle differences in how CommonJS, RequireJS, SystemJS, and ES2015 (AKA ES6) module loaders
  expose the contents of modules.

To understand the most important distinctions, we'll start with an explanation of what an ES2015 module is.

### ES2015 modules

The definition of an ES2015 module is deceptively simple:
An ES2015 module consists of a set of properties.
These properties are created through the `export` keyword.

### `import` and `export` in ES2015

#### `export`

ES2015 supports a variety of `export` forms:
```ts
let a = 10;
// Export the `a` variable under the name `a`
export { a };
// Export `a` under the name `b`
export { a as b };

// Export the variable `c` under the name `c`
export var c;

// Export the class `Foo` under the name `Foo`
export class Foo { }

// Export the function `func` under the name `func`
export function func() { }
```

#### `export default`
Additionally, most of these forms can accept the `default` modifier:
```ts
export default function() { }
```

The semantics of the `default` keyword are very simple:
A `default` export creates an export with the name `"default"`.

This bears repeating: the only effect in ES2015 of a `default` export
  is to create an export with the *name* "default".

### `import`

ES2015 also supports a variety of `import` forms:
```ts
// Create a local, `a`, from the `a` export of someMod
import { a } from 'someMod';

// Import the entire module object of `someMod` as `obj`
import * as obj from 'someMod';

// Create a local, `b`, from the `c` export of someMod
import { c as b } from 'someMod';

// Create locals `Foo` and `func` from the corresponding exports of someMod
import { Foo, func } from 'someMod'

// Create a local 'def' from the 'default' export of someMod
import def from 'someMod';
```

It's important to call out a distinction between these two lines:
```ts
import * as foo from 'someMod';
import bar from 'someMod'
```
These look like they might do the same thing, but they don't.

The `foo` local contains the *module object* itself. The `bar` local contains the `default` export of that object.

In other words, `foo.default === bar` (or equivalently, `foo["default"] === bar`).

Again, the default export is *just a name*.
The `import x from 'y'` form is syntactic sugar for retrieving the export named `default`.

### Exporting a top-level call signature

If you read the above text carefully, you'll notice there's no mechanism by which this would be legal:
```ts
import * as fn from 'someMod';
fn(); // In ES2015, this cannot succeed
```

This is because the module object, `fn`, is never based on a function.
It is only a collection of properties.
However, this is a common pattern in CommonJS and other legacy module systems.
How is this handled today?

### Legacy Module Systems and Callable Modules

In CommonJS, for example, you might see this code:
```ts
var express = require('express');
var x = express();
```
While this is legal in CommonJS and other module loaders, the `express` module is not
  compliant with ES2015 semantics.
Only properties may be exported from an ES2015 module, not call signatures.

This presents a problem for module loaders because correctly emulating
  the ES2015 behavior would break many CommonJS modules.

This is solved through the *synthetic default export* behavior.

### Synthetic `default` exports



