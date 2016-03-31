# User Scenarios

* As a developer, ...
  * I want to `npm install some-library` and get its corresponding .d.ts file in my compilation as automatically as possible
  * I want to be able to easily get .d.ts files for libraries which only have global scope effects
* As a package author, ...
  * I want to include my own .d.ts file with my NPM package and have that be recognized automatically by the TypeScript compiler
  * I want to be able to refer to other packages' typing files

-----

# Open Problems in Typing

There are a number of problems with proposed solutions in type acquisition and consumption.
This section serves as a taxonomy and definition of those problems.

## Global Version Conflict
**Problem**: Multiple libraries may claim dependencies on different versions of a global variable
for which only one version may actually exist.

**Example**: Two libraries, Alpha and Beta, both claim a dependency on jQuery.
Alpha depends on jQuery 2.14.
Beta depends on jQuery 1.42.

**Intended fix**: The user needs to resolve the conflict. They can do so using library directives.

## Module Version Conflict
**Problem**: Multiple libraries may claim dependencies on different versions of a module.

**Example**: Same as in Global Version Conflict, but with dependencies on modules.

**Intended fix**: This should simply not be an error.

## Diamond Reference
**Problem**: Multiple libraries may claim dependences on different versions of a module,
but types coming out of semver-compatible definition files should still be structurally
compatible even if they have private members.

**Example**: Same as in Module Version Conflict, but Alpha and Beta each re-export an
instance of a class with a private member. Those re-exports *should* be compatible
as long as their declarations are the same.

**Intended fix**: Unclear. We could choose to use structural typing here, or we could
legitimately decide that these two objects *are* different -- for example, `instanceof`
will not be a reliable runtime indicator of these types.

## Not-in-box Typings
**Problem**: Some libraries (Angular2) include their own typings, but do not include
typings for dependencies.

**Example**: Angular2 ships its .d.ts in-package, but does not provide typings for RxJS.

**Intended fix**: Some mechanism for Angular2 to identify its typings dependencies.

## Language Version
**Problem**: New syntax in TypeScript will not be understood by old versions of the compiler.

**Example**: When we ship `readonly`, anyone who adds this to their .d.ts will break any
prior-versioned consumer who automatically updates their typings.

**Intended fix**: See 'NPM Typings Versioning Strategy'

## UMD Module Definitions
**Problem**: Many definition files on DefinitelyTyped are written in this form:
```ts
interface SomeFooThing {
	// ...
}
declare namespace Foo {
	export function something(): SomeFooThing;
}
declare module "Foo" {
	export = Foo;
}
```

This has several issues:
* We can't use path-based module lookups
* We can't use the path of the file to disambiguate between multiple versions of a library
* Consumers of the module side still get global namespace pollution

**Example**: https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/durandal/durandal.d.ts

**Intended fix**: Built-in language or tooling support for certain kinds of UMD module definitions

## Relative `/// <reference ... />` tags
**Problem**: Many .d.ts files on DefinitelyTyped have relative reference paths
to other definition files. This is good for TSD, etc., but breaks if people
want to bundle those definition files into NPM packages which might have arbitrary
folder structure.

**Example**: https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/durandal/durandal.d.ts

**Intended fix**: Some new declaration form that allows a UMD module to be expressed without
writing out all the types twice.

Tracking at issue #7125

----------------------

# Solution Components

## `node_modules/@types`
### Overview
We will create an NPM user called `@types`. This account will publish one package
per DefinitelyTyped folder. These packages will contain only .d.ts files.

### Peer dependencies
These packages will have dependencies on other NPM `@types/` packages instead of `reference` directives
to peer-folder .d.ts files. `import` declarations will also introduce a dependency to a peer package.

Additionally, any reference directives will be rewritten to library include directives as part of publishing.

### Global definitions
`@types/` packages may also have Library Include Directives (see below).

### Versioning
Version numbers of `@types/` packages is determined as follows:
 * The major and minor versions are read from the .d.ts file (all DefinitelyTyped files start with a version comment)
 * The patch version increments every time we publish
 * A special tagged version `@ts1.6` is created for each legacy TypeScript language version; the most-recent file which
   parses validly in that version of the language is published at this tag   

### Consumption through package.json
Package authors may add a `typing-dependencies` field to their `package.json` which specifies which
other packages should be downloaded. `tsc` will provide a way (`tsc get-types` ?) to promote all
locally-installed packages' `typing-dependencies` to `dev-dependencies` of the root's `package.json`.

## Library Include Directives
### Overview
In addition to `/// <reference path="..." />`, we will also support some new
syntax (TODO: bikeshed this) which behaves similar to C's `#include <filename.h>`. Tracking at #7156

### Resolution
Resolving a library include directive checks in these folders in the following order:
 * `./typings/` (resolved from the root of the compilation unit, e.g. wherever tsconfig.json is / would be)
 * `./node_modules/@types`

The first file found via this algorithm 'wins'.

This search path could be made customizable in `tsconfig.json`.

## Built-in UMD Module Support
### Overview
Without bikeshedding on syntax, we need a way to declare that a global variable has the type
of an external module.

### Structuring
The ideal format would probably look something like this
***underscore.d.ts***
```ts
export function each(...);
// TODO: Bikeshed the syntax
export global var $;
// etc
```

Consumers would then either
```ts
import * as _ from 'underscore';
```
or
```ts
/// <reference library="underscore/underscore.d.ts" />

_.each(...)
```

## Massive Rewrite of DefinitelyTyped

Once all the above pieces are in place, we'll need to start converting definition files
on DefinitelyTyped to the appropriate format.

A recent scan shows that about 30% of files might be able to be converted directly to proper module declarations.
Another 40% are already purely global, which don't require any conversion.
This leaves about 30% which will probably require manual work. It's currently not known what percentage of those
can be converted directly into UMD format.

Part of the NPM publishing tool could be a validation of which modules still required work.

## Typings Dependencies in `package.json`

When an NPM package has dependencies on global definition files, it can provide an entry
in package.json specifying additional type dependencies.

These essentially act like something between a regular dependency and a dev dependency.
The user can run some command (`tsc get-types` ?) that scans the containing project's
`package.json`, finds all dependencies, and adds the typings-dependencies of those
packages to the dev-dependencies of the current project.

