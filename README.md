# Add the submodule

At this point, the API in the submodule was at version 1.

```sh
$ git submodule add --branch master https://github.com/GROSSWEBER/git-submodule-sub.git lib/api
Cloning into '/scratch/git-submodule-super/lib/api'...
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 3 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
Checking connectivity... done.

$ git status
On branch master

Initial commit

Changes to be committed:

        new file:   .gitmodules
        new file:   lib/api

Untracked files:

        run.sh

$ ./run.sh
submodule version 1
```

Now you may commit as usual.

# Update the submodule by fetching new commits in the submodule's `master`branch

```sh
$ git submodule update --remote
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 3 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
From https://github.com/GROSSWEBER/git-submodule-sub
   8be9e95..d6eb0c1  master     -> origin/master
Submodule path 'lib/api': checked out 'd6eb0c1d30f6ceab5359ae36b334109cc4e146c3'

$ ./run.sh
submodule version 2
```

# Make changes in `lib/api` to e.g. fix a bug

```
# edit lib/api/api.sh

$ ./run.sh
submodule version 2 with fix

$ git status
On branch master
Changes not staged for commit:

        modified:   lib/api (modified content)

$ cd lib/api

$ git commit -am "fix a bug in version"
[detached HEAD ebaf90a] fix a bug in version
 1 file changed, 1 insertion(+), 1 deletion(-)

$ cd ../..

$ git status
On branch master
Changes not staged for commit:

        modified:   lib/api (new commits)
```

At this point the submodule points to a commit only available on the local machine. **By committing and pushing the new commit in lib/api you would essentially make the submodule pointer invalid for everybody else.** Make sure you push your submodule first before pushing the superproject:

```sh
$ cd lib/api

$ git push git@github.com:GROSSWEBER/git-submodule-sub.git HEAD:master
...
To git@github.com:GROSSWEBER/git-submodule-sub.git
   d6eb0c1..ebaf90a  HEAD -> master

$ cd ../..
```

If that happens very often, you might want to create a new remote with `git remote add <name> <url>`.

Back in the superproject, the situation is still the same: The superproject references a new commit (i.e. the submodule pointer changed) that needs to be committed and pushed.

```sh
$ git status
On branch master
Changes not staged for commit:

        modified:   lib/api (new commits)

$ ./run.sh
submodule version 2 with fix

$ git commit -am "Update submodule with fix"
[master 2495be1] Update submodule with fix
 1 file changed, 1 insertion(+), 1 deletion(-)

$ git push origin master
...
To git@github.com:GROSSWEBER/git-submodule-super.git
   85d569e..2495be1  master -> master
```

# Changing the HEAD revision in the superproject doesn't update the submodule

When you change HEAD by e.g. `git reset` or `git checkout`, submodules aren't updated automatically.

```sh
$ ./run.sh
submodule version 2 with fix

$ git reset --hard bbc3168
HEAD is now at bbc3168 Pointing to API version 1

$ ./run.sh
submodule version 2 with fix

$ git status
On branch master
Your branch is behind 'origin/master' by 2 commits, and can be fast-forwarded.
  (use "git pull" to update your local branch)
Changes not staged for commit:

        modified:   lib/api (new commits)
```

The submodule is still pointing at version 2 with the fix, this is why `git status` displays `(new commits)`!

Make sure we have the API consistent with the `HEAD` revision:

```sh
$ git submodule update
Submodule path 'lib/api': checked out '8be9e950346c56f9757392524d6b2e5e45b035d1'

$ ./run.sh
submodule version 1
```

# Clone a repository including all submodules

`git clone` will not download submodules:

```sh
$ git clone https://github.com/GROSSWEBER/git-submodule-super.git clone-non-recursive

$ cd clone-non-recursive

$ ./run.sh
./run.sh: line 3: ./lib/api/api.sh: No such file or directory
```

You need to fetch the submodules yourself:
```sh
$ git submodule init
Submodule 'lib/api' (https://github.com/GROSSWEBER/git-submodule-sub.git) registered for path 'lib/api'

$ ./run.sh
./run.sh: line 3: ./lib/api/api.sh: No such file or directory

# Still not working!

$ git submodule update
Cloning into '/scratch/clone-non-recursive/lib/api'...
...
Submodule path 'lib/api': checked out 'ebaf90a7b75e31df8b30cc6f96d6c1fb470cc478'

$ ./run.sh
submodule version 2 with fix
```

Alternatively, `git clone` the repository including all submodules:

```sh
$ git clone --recursive https://github.com/GROSSWEBER/git-submodule-super.git clone-recursive
Cloning into 'clone-recursive'...
...
Submodule 'lib/api' (https://github.com/GROSSWEBER/git-submodule-sub.git) registered for path 'lib/api'
Cloning into '/scratch/clone-recursive/lib/api'...
...
Submodule path 'lib/api': checked out 'ebaf90a7b75e31df8b30cc6f96d6c1fb470cc478'

$ cd clone-recursive

$ ./run.sh
submodule version 2 with fix
```







