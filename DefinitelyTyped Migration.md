# What's Next for DefinitelyTyped

With the upcoming release of TypeScript 2.0 and its improved story for getting type definitions,
  it's time for us to also bring some improvements to DefinitelyTyped.



# Background

DefinitelyTyped currently hosts 1,622 type definitions.
These were created from around 18,000 commits from 1,800 contributors over the course of 7,000 Pull Requests.

# Problems Today

We identify four prominent patterns of type definition files on DefinitelyTyped:

* *"declare module"*, where the sole top-level declaration in a file is `declare module "someName" { ... }`. Approximately 30% of files.
* *"global"*, where only global declarations exist (e.g. no ambient external module declarations, no top-level `export` / `import` declarations). Approximately 40% of files.
* *"mixed"*, containing both global declarations and an ambient external module declaration (`declare module "someName" { ... }`). Approximately 25% of files.
* *"multiple module"*, where there are multiple declarations of the form `declare module "someName" { ... }`. Approximately 5% of files.

All of these forms have at least one declaration that ends up in the global namespace.
This is a problem when multiple copies of these files are referenced in the same compilation; "Duplicate identifier" errors occur
  and users are confused about how to fix it.

# The New World

Two major new features in TypeScript will drastically simplify .d.ts authoring and distribution.

The first, UMD module definitions (https://github.com/Microsoft/TypeScript/issues/7125) will make
  the current UMD module declaration pattern obsolete.
This will reduce global pollution and force declaration file authors to be more modular in their type layouts.

The second, library reference directives (https://github.com/Microsoft/TypeScript/issues/7156) will make
  it more predictable for users to reference typings files without having to memorize relative paths or filenames.

Unfortunately, both of these are new syntax that current versions of TypeScript won't understand.
We will need to provide a transition path.

# The Path Forward

Going forward, we expect to see only two kinds of files:

* *"proper modules"*, which contain top-level `import` and `export` declarations. These are currently nonexistant on DefinitelyTyped.
* *"global declarations"*, which do not contain `declare module "someName" {` or top-level `export` declarations.

Files written in the *declare module* format can be mechanically converted into *proper modules*.

Files written in the *global* format can be left as-is, though they should be reviewed to ensure they don't pollute the global scope more than needed.

*Mixed* files break down into two cases.
When the file only declares one global, it can be rewritten to a UMD module without other impact.
When the file declares more than one global, it will likely need to be substantially refactored to only expose one global.
This will represent a breaking change to consumers of the .d.ts file.
In the event this change is too broad, we may need to break the file out into separate global and module definition files.

Files written as *multiple module* need to be addressed on a case-by-case basis.
They may be better suited to be left as-is (e.g. node.d.ts), or broken into separate definition folders.

# Transition Plan

Plan for transition:

 * Create a new branch, `modern`
 * Auto-convert as many files as possible to the new format (current guess is that we can do this for 60% of folders)
 * Manually convert any popular libraries that weren't auto-converted
 * Start publishing compliant typings to NPM
 * Create a work item listing all remaining unpublished noncompliant typings

# Typings Sources

With the addition of TypeScript support for getting typings from NPM packages,
  we see three sources of type data going forward:

* **Bundled** typings are .d.ts files that are part of the NPM package of the library itself
* **Community** typings are hosted on DefinitelyTyped and republished to the `@types/` namespace
* **Third-party** typings are hosted DefinitelyTyped repos, but are still published to `@types/`

# UMD and Library Directives


