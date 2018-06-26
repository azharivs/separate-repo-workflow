
```bash
mkdir local_repo
cd local_repo
git init
git remote add origin https://origin_url
git remote add devrepo http://devrepo_url #for development
touch dummy
git add dummy
git commit -m "create dummy and commit to next allow branching"
git branch dev #for development
git checkout dev
#vi .gitignore #add *.* and * to .gitignore which will only be part of this branch (dev)
#git add -f .gitignore
#git commit -m "added special .gitignore for dev branch"
git push -u devrepo dev #push the skeleton of the dev branch to the devrepo
```

get base code from origin repo

```bash
git checkout master #always do work on master branch
git pull origin master #get main code base from origin/master. This is possibly followed by merge into local master
git commit -m "after merge or origin/master into master"
```

**TODO:** `.git/info/last_origin_pull` stores the commit SHA-1 for this merge as a reference point for later rebase/cherry-picks of `local/master` onto `local/dev` (use client side git hooks for this)

**TODO:** NEVER PUSH MASTER TO `devrepo` (use server side git hooks for this) Only allowed to push from `dev` branch to `dev` branch on `devrepo`

Optional: pull in work from `dev` (devel) repo and marge into local `master`

```bash
git checkout -b dev devrepo/dev #only for the first time when no branch dev is setup in local working tree
git checkout dev #all other times: just checkout local dev and pull from devrepo
git pull devrepo dev #all other times: possibly followed by merge
```

local `dev` branch setup and updated now merge into local `master`

```bash
git checkout master
git merge -X theirs dev
git commit -m "merge in dev into master"
```

start developping on local/master .....

```bash
git commit -m "Final commit on local/master"
git add files changed but not staged in this dev session #add to staging
```

**TODO:** NEVER allow push while on local/master because this will take all the history on master including base repo files into `devrepo`! (server side git hooks)

**TODO:** NEVER allow commits while on `local/dev` branch (client side git hooks). `local/dev` branch only updated with cherry-picking or rebasing using automatic scripts (client side git hooks)

time to commit changes ONLY to `devrepo/dev` use cherry picking

```bash
git checkout master 
git log #take commits after the last merge into master C1,C2,C3,.... In case a merge from origin into master has resulted in changing of a file staged on dev then need to take this merge commit as well: M (client side git hooks can be used to store these in a file)
git checkout dev #move to dev for cherry-picking onto it
git cherry-pick [-m(1|2) M] C1 C2 C3 ... #also need to determine which parent of the merge? -m1 or -m2? TODO?
```

resolve conflicts: if any modified file is not in dev then:

```bash
git add newly added files
git cherry-pick --continue
git push devrepo dev #push changes back upstream (server side git hooks ONLY ALLOW PUSH FROM dev to dev)
```

**TODO:** eliminate `.gitignore` no need for it because we are controlling what is kept on the `dev` branch locally and the `master` branch is only a private one locally.

```bash
# time to commit changes ONLY to devrepo/dev use a modified rebase
# export CUR_MASTER_HEAD = cat .git/refs/heads/master #store current master HEAD SHA-1
# git checkout master
# git rebase dev #rebase master onto dev so dev is updated with all diffs on master since ancestor
# git checkout dev
# git merge master #fast forward dev pointer to rebased one (latest)
# git checkout $CUR_MASTER_HEAD #restore master head to original value
# vi .gitignore #make exception for appropriate files to .gitignore (dev) (only those changed during devel and not those pulled from origin but not changed locally)
```

What if `git pull origin` adds/modifies some files not touched by `devrepo`? we don't want these to go into `devrepo`
TODO: No problem as long as we record the commit hash of the merging `origin/master` into master and start cherry picking AFTER that (tested and works. Use client side git hooks to take care of)

What if some file `f2` is modified via `devrepo` but is then also modified (or even deleted) via `origin`?
both versions of `f2` should be merged and rebased onto `dev` upon pulling from `origin`

**option1:** keep origin version and discard devrepo's version: in this case devrepo has to be updated so that `f2` is unstaged from it

**option2:** keep devrepo's version: no changes required to anything

**option3:** modify origin's version to a newer one: this case will be an option 1 followed by the general approach of development so we should be fine. However, in this case it is best not to go with option 1 because then we have to delete it from devrepo and then add it again. So it is better to open up the text editor and start resolving conflicts! Then cherry-pick onto dev. In this case the merge itself has to be included in cherry picking and the parent used for getting the diff as well (-m1 or -m2) he parent used should be the one pointing to the change on the origin 

```bash
git diff pull_from_origin_merge_commit | grep -e "---" -e "+++" | cut -d'/' -f2-1000
```

**TODO:** the commit of origin corresponding to each commit of devrepo should be stored in a certain file via git hooks. This tells the developper what commit should be pulled from origin and marged onto master locally with devrepo/dev 


