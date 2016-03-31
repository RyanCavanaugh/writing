
## Problems

`/// <reference path="..." />` comments (hereafter "reference directives") currently serve
multiple purposes:
* Including another definition file containing global declarations
* Including another definition file containing ambient module declarations
* Including another implementation file

The current mechanism has some common shortcomings:
* Relative paths to a common typings folder can be come unwieldy
* There's no mechanism to "override" a reference path
* Reference directives only ever look in one place to find a file.

This has resulted in multiple issue reports of people trying to address this:
* #5893 Multiple source roots - or something like classpath, like python_path, like include_path
* #4615 Support environment variables in reference paths
* #6482 Support a 'declarationPath' option

## New Use Case in Bundled Type Acquisition

An important scenario we need to address is the case where multiple typings
files need to refer to the same *global* set of types or namespaces. If these
libraries refer to multiple versions of the same global types, we have no
existing mechanism for resolving the conflict

## Solution: Library include directive

### Syntax

The proposed syntax (:bike: :house: of course) would look like this
```ts
/// <reference library="jquery/jquery.d.ts" />
```

### Behavior

This syntax means that the compiler should include the *first* file found
when searching the following paths, where relative paths are resolved relative
to the project root (either the location of `tsconfig.json`, or the common
root of all input files):
* `./typings/jquery/jquery.d.ts`
* `./node_modules/jquery/jquery.d.ts`
* `./node_modules/@types/jquery/jquery.d.ts`

This search order provides the following desirable behavior:
* When needed, the user can resolve a version conflict by placing a definitive
global version in the local `typings` folder
* A bundled `.d.ts` file will be found automatically
* A `types` package can fill in for libraries which don't have bundled definitions

We could conceivably allow for configuration of this search order in `tsconfig.json` if
needed for complex scenarios.

### UMD

Additionally, when a UMD module definition is found, its global export declaration (if present)
is added to the global scope.

## All-up World

Along with the rest of the typings acquisition story, we can now have a very coherent way to
explain how file references should be managed in TypeScript programs:

* Modules which are part of your program are always imported using `import`
* Modules which have definition files are always imported using `import`, and those `import`s should resolve to proper modules
* Global definitions should always be located via `/// <reference library = ...` directives
* File ordering for concatenated output is managed with `/// <reference path = ....` directives


