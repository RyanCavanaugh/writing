# Preview: Type Acquisition in TypeScript 2.0

Today we'd like to show you a new experience for TypeScript developers,
  and offer you to try it out in our nightly build.

Getting type definitions for libraries is a common task that so far has been handled by third-party tools.
Our goal with this new suite of features is to simplify and streamline the process.
This new experience leverages existing tools in the JavaScript ecosystem,
  offering a single-step commandline process for getting type definitions.

Let's walk through using the example of getting types for [lodash](https://lodash.com/)

### Step 1: `npm install @types/lodash`

On the commandline:

> `npm install @types/lodash`

### Step 2: Ready!

There's no step 2 in this process!
With the type definitions installed in `@types/`, we can use them immediately.

If you're using lodash from the global scope, you can write:
```ts
// consumer.ts
_.map([1, 2, 3], function(n) { return n * 3; });
```

If you're using a module loader to load the script, you can write:
```ts
// consumer.ts
import * as _ from 'lodash';

_.map([1, 2, 3], function(n) { return n * 3; });
```

You can now compile as usual by running:
```
tsc consumer.ts
```

This functionality is available now in the nightly builds of TypeScript (`npm install typescript@next`).

## More

If we install types for a library with dependencies, the dependencies are install automatically:

> `npm install @types/jquery-ui`

```ts
// jQuery and jQuery UI definitions both installed, everything's OK
$('myElem').dialog();
```

