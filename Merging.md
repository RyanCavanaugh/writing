# Performing an incremental merge

Make sure you're on the 2.0 branch

> git checkout 2.0

Make a new branch

> git checkout -b my_merge

Run this command to make sure renames are correctly detected

> git config merge.renamelimit 5000

Run this command

> git merge upstream/master~QUACK -s recursive -X rename-threshold=40% -X ignore-space-change

where QUACK is 100 fewer than the number of commits you have to merge (assuming you want to do 100 commits at once).
Start with a number like 800 and go down by 100 until you pull in a nonzero number of commits.

If you get too many commits at once, run

> git reset --hard my_merge

Fix everything up, commit, then reduce QUACK by 50 or 100 or so, and repeat until QUACK is 0.

# Handling bad merges

Sometimes git doesn't do a good job doing a three-way merge in renames.
You'll see a message like this.

> CONFLICT (modify/delete): request/request.d.ts deleted in HEAD and modified in upstream/master~650. Version upstream/master~650 of request/request.d.ts left in tree.

Beyond Compare 4 seems to do this better.
I use a script that runs from the DefinitelyTyped root and takes one argument - the folder name.

> "C:\Program Files\Beyond Compare 4\BComp.exe" %1\%1.d.ts %1\index.d.ts C:\github\DefinitelyTyped-master\%1\%1.d.ts

This script needs the following:
 * Current directory is the root of the DefinitelyTyped repo
 * You're in the state where the file was renamed (so you have `foo\foo.d.ts` in `master` and `foo\index.d.ts` in the 2.0 branch)
 * You have the repo `C:\github\DefinitelyTyped-master` checked out to the **newest common ancestor** of `2.0` and `master`
   * Use `git ?????? master` to find this commit

When the Beyond Compare window opens, have it write the merged file to the "right" file.

# Faster Cleanup

Another useful script is:

> call pushd %1 && tsc && tsfmt -r index.d.ts && git add index.d.ts && git rm %1.d.ts
> popd

Again, this script runs from the DefinitelyTyped root and accepts a folder name as an argument.
When run, this will:
 * Change into the folder
 * Run `tsc` to confirm changes. If this succeeds:
   * Formats the file (`npm install -g tsfmt` first)
   * Stages the changes to `index.d.ts`
   * Removes any file named under the old convention
 * Changes back to the DT root

Note that if there are test failures, you'll need to fix them.
