# Current State

## tsd: deprecated

`tsd` has been a successful solution, but is 'deprecated'.
Work in other tools like `typings` is ongoing, but general feedback is that we need a built-in, first-party solution to unify everyone on to.

## DefinitelyTyped

DefinitelyTyped currently has about 1,500 libraries, representing nearly 17,000 commits from 1,700 contributors.
While every file has some minimal data, there is not a precise 1:1 correspondence between an arbitrary JS file
and its corresponding typings file.

With that in mind, we need to ensure anything we do doesn't require huge amounts of manual work updating definition files, as we won't have the time to do this manually.

---------------------------------------------------


# Big Problems in Typing

## Diamond Problem

The 'diamond reference' problem looks like this, where each letter represent a definition file in a folder.
```
     a   a'
     |   |
     b   c
      \ /
       d
       ^
       |
   user code
```
Here, user code referred to module `d`, which in turn referenced modules `b` and `c`.
Those modules, respectively, imported `a` and `a'`.

There are two important variants here:

### `a` === `a'`
In the simplest case, the diamond is actually diamond-shaped, and the compiler simply reports "duplicate identifier" errors.
You could imagine solving this by e.g. uniquefying based on file hash, or other heuristics, but this leads us to the hard variant

### `a` !== `a'`
In the more difficult case, `b` referenced e.g. v2.4 of `a` and `c` referenced v3.0 of `a`.
Now you have two files which are very similar (and still report many duplicate identifier errors), but for which there
is no obvious way to decide which file should be considered authoritative.

### Further Unworkable Problems
In an ideal world, the typings for `a` would only export *modules* and we could distinguish them based on import path.
However, most typings files actually define many types in the global space even if the library itself is a module, so simply distinguishing modules based on path is not sufficient.

## Library Versioning


## Typings Versioning

## Compiler Versioning

## Globals vs Modules


---------------------------------------------------




# Solutions in TSC

## Distribution

## Acquisition

## Versioning

## Querying

---------------------------------------------------


# Changes in DefinitelyTyped

## Versioning folders

## Search metadata

---------------------------------------------------


# Experience Roll-up



---------------------------------------------------
