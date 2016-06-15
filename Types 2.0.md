# Types 2.0

## Background

TypeScript programmers frequently need *definition files* for libraries they're using.
For example, a project using React will need `react.d.ts` to get accurate type information.
Most projects a use dozen or more libraries and need a corresponding number of definition files.

We call the process of getting these files *type acquistion*.
Type acquisition has been cited as a frequent pain point for both new and experienced TypeScript users.
Searching, managing, and downloading definition files can be confusing and cumbersome.

The *Types 2.0* initiatve is targeted at solving this problem.

## Goals for Types 2.0

Our goals for Types 2.0 are to eliminate dependencies on third-party tools,
  better support common JavaScript patterns,
  provide a comprehensive versioning story,
  offer fast and accurate searching,
  and fully support more ways of acquiring types.

### Tool-free Experience

User research shows that a vast majority of JavaScript developers are already familiar with NPM.
Types 2.0 uses NPM as a distribution and versioning mechanism,
  eliminating the need for tools like `tsd` or `typings`.

### Understanding JavaScript Patterns

Almost every popular JavaScript library is available as a *UMD* module,
  meaning it can work with or without a module loader.
Types 2.0 natively understands UMD modules,
  reducing global scope pollution and finding more bugs.

### Comprehensive Versioning

Modern module loaders can simulatenously load multiple versions of the same library.
Types 2.0 can accurately model these situations,
  as well as find potential conflicts in the global scope and resolve them.

### Fast Searching

Types 2.0 understands the universe of available type definitions and can provide instant search,
  allowing developers to quickly find definitions for the libraries they're using.

### More Sources of Types

Types 2.0 can load types from both NPM and first-party definitions.
This allows library authors to easily provide "built-in" type data for their libraries.

# A Quick Guide for Developers

If you already know the name of your library, just run

 > `npm install @types/libraryname` --save-dev

The TypeScript compiler will automatically consume the definition files in the `node_modules/@types` folder,
  so no further action is needed.
Dependencies will also be automatically consumed and installed.

If you're unsure of the name of your library, go to http://aka.ms/types.
This website provides instant search for every available definition file.
You can search for the library name (such as `lodash`),
  the module name (e.g. `react-dom`),
  or a global variable provided by the library (e.g. `$` will find `jquery`).
Submitting the search form will take you to the NPM page for the types package,
  which will provide an installation commandline similar to the one shown above.

# What's Changing on DefinitelyTyped?

* Follow these patterns
* Improved Documentation
* Modified file layout
* Faster test setup
