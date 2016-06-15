# What's Next for DefinitelyTyped

To better support our new vision (TODO: link) of type acquisition,
  the TypeScript team intends to invest heavily in improving DefinitelyTyped.
We've identified five major areas of improvement and
  we'd like to share with you our vision in each of these areas.

## Better Infrastructure

Today, it takes nearly an hour to run every test in the DefinitelyTyped repo.
This is because the test runner starts a separate compiler instance for each file.
Now that the TypeScript compiler API is stabilized,
  we can use it to much more quickly validate the entire repository.
This also eliminates the need for complex change detection logic,
  and allows us to more easily validate downstream breaks in dependent files.

Starting with the 2.0 branch, the test runner will use the compiler API.
This results in a massive improvement to overall runtime,
  with a full test run now taking less than 10 minutes.

Additionally, we'll be using tsconfig files in each folder to store compilation settings.
This means that authors can validate their changes simply by running `tsc`.

## Comprehensive Documentation

We've seen consistent feedback that the TypeScript documenation for writing definition files
  is incomplete and confusing.
Many files today contain common errors or confusions resulting from the lack of clear documentation.

To remedy this, we've started a separate documentation project targeted specifically
  at definition file authors.
This project contains templates,
  detailed example-driven guides,
  deep dives on writing advanced files,
  and specific guidance in areas of common confusion.

We'll continue to update this documentation portal as the language evolves and as we receive more feedback.

## Improved Static Verification

We will write new TSLint (TODO: link) rules specifically targeted at definition file authors.
These rules will catch common errors and provide specific recommendations for fixes.

As with the documentation, we'll improve and augment these rules as we identify new problem areas.

## Faster Turnaround

Today, DefinitelyTyped is staffed by a small collection of dedicated volunteers.
It's time for the TypeScript team to dedicate more of its resources to assist this critical effort.
To that end, we'll be assigning TypeScript engineers to the specific task
  of reviewing and merging pull requests.

Our goal is to provide same-day turnaround during business hours for minor changes,
  and 2 business day turnaround for more complex changes.

Additionally, we'll be plowing through the large backlog of unmerged pull requests.

Finally, during the transitional phase,
  the TypeScript team will perform weekly merges from the `master` branch to the `2.0` branch.
This will continue until 6 weeks after the 2.0 TypeScript release.

## Clearer Ownership

Today, we see a spectrum of ownership of definition files.
Some definitions are clearly led by a small group of core contributors.
Other definitions are built ad-hoc by many people pitching in small changes.
And some definitions fall somewhere between these two extremes.
Some NPM packages also provide their own definition files,
  eliminating the need for a definition on DefinitelyTyped.

We feel that the DefinitelyTyped repo is the appropriate place for ad-hoc definitions.
Having a central repo for these typically small definitions lowers the barrier to entry,
  and allows a broader set of people to sign off and merge changes.
It's important that we not have large upfront costs for making small definition files.

However, DefinitelyTyped should not be a megarepo.
For definition files that have a core set of strong and active contributors,
  or an appropriate organization to support them,
  hosting development in a separate repo is appropriate and necessary.
We'll support people "taking ownership" of a definition by having redirection
  from DefinitelyTyped to their own repo, and perform periodic audits of these
  repos to ensure development is being appropriately stewarded.

Additionally, for packages that ship their own definition files as part of distribution,
  we don't want duplicative effort on DefinitelyTyped to take place.
We will support package authors claiming "first party" definition status of a particular definition.
However, if a package author stops updating their definition file appropriately,
  we may need to revert this redirection back to the community.
Those will be handled on a per-case basis.

To clarify this, we want to establish some terminology:
 * A *community* definition is one hosted in the main DefinitelyTyped repo
 * An *organization* definition is hosted in an external repo
 * A *first-party* definition comes from the underlying NPM package itself

