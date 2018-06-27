# Initialization Phase
If you want to setup the main and dev repositories from scratch:

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
However, in general these will be craeted beforehand. The only requirement is that the dev repo only have a dev branch and that both be introduced to the local work tree as remotes via:
```bash
git remote add origin https://origin_url
git remote add devrepo http://devrepo_url #for development
git checkout -b dev devrepo/dev #only for the first time when no branch dev is setup in local working tree
```
# Development Workflow:

* Pull in development code from `devrepo` into `master` local branch (and merge)
* Pull in base code from `origin` repo into `master` local branch (and merge)
* Develop on `master` branch of local work tree
* Rebase/Cherry pick commits made onto a `dev` branch of local tree
* Push work from a `dev` branch to `devrepo`


**Step1. Pull in  code from devrepo:**

Pull in work from the `dev` (devel) repo updating all dev branches and then marge into local `master` to start development on it.

only for the first time when no branch dev is setup in local working tree
```bash
git checkout -b dev devrepo/dev 
```
all other times: just check out local dev and pull from devrepo
```bash
git checkout dev 
git pull devrepo dev #possibly followed by merge
```
Local `dev` branches all setup and updated. Note that there may well be multiple branches off dev
each belonging to a different development topic. Now merge `dev` into local `master`, with merge policy that 
resolves conflicts by taking the `dev` copy of the files over the `origin/master` copy. The reason is that the development work
has higher priority.

```bash
git checkout master
git merge -X theirs dev
git commit -m "merge in dev into master"
```

**Step2. Pull in  code from origin:**
If desired, we can update base code (master branch) from origin. Resolve merge conflicts manually. We should also record the merge SHA, in case a merge from origin into master has resulted in changing of a file on dev. This is because we later have to cherry pick the diff for this merge onto dev as well to reflect that change.

```bash
git checkout master #always do work on master branch
git pull origin master #get main code base from origin/master. This is possibly followed by merge into local master
git commit -m "after merge of origin/master into master"
```

Next we check if a merge from origin into master has resulted in changing of a file on dev. This will be used later in Step 4:
The first line stores the the merge parent which belongs to origin/master. Note that `origin/master` should match the remote branch we are pulling in and should be set differently if the name of the remote and branch is different. This could potentially be used in cherry picking (see Step 4). The second line stores the commit SHA-1 for the resulting merge as a reference point for later rebase/cherry-picks of `local/master` onto `local/dev` (see Step 4) (can use client side git hooks for this). 

```bash
cat .git/refs/remotes/origin/master > .git/info/MERGE_SIDE #store the origin/master side of merge (parent) 
cat .git/refs/heads/master > .git/info/LAST_ORIGIN_PULL #store merge SHA
git diff $(cat .git/info/LAST_ORIGIN_PULL) | grep -e "---" -e "+++" | cut -d'/' -f2-1000 > ./git/info/$LAST_ORIGIN_PULL.change #obtain list of files changed by this merge
git checkout dev #or checkout any other branch on dev 
find . -path ./.git -prune -o -print | cut -d'/' -f2-1000 > .git/info/ALL_DEV_FILES.tmp #list of all files on dev
grep -F -x -f .git/info/DEV_FILES.tmp ./git/info/$LAST_ORIGIN_PULL.change > .git/info/DEV_FILES.change #find those changed by merge which are also part of dev
#next parts to be run only if changes are made (grep returns some matches)
git log | grep Merge: | cut -d' ' -f2 | grep $(head -c 7 .git/info/MERGE_SIDE) #use -m1
git log | grep Merge: | cut -d' ' -f3 | grep $(head -c 7 .git/info/MERGE_SIDE) #use -m2
cat .git/info/LAST_ORIGIN_PULL > ./git/info/CHERRY_PICK_COMMITS 
```
You may use https://github.com/azharivs/separate-repo-workflow/blob/test/post-pull-origin as a full script performing all this and many more. To use this as a githook you may do the following:
clone this repo into your .git/hooks/
add symlinks post-commit and post-merge that point to this:
```bash
cd .git/hooks
git clone https://github.com/azharivs/separate-repo-workflow.git
ln -s separate-repo-workflow/post-pull-origin post-merge
ln -s separate-repo-workflow/post-pull-origin post-commit
```


**Step 3. Start developping on local/master:**
Commit on master (or any one of its branches) as you work. With each commit also add the commit SHA to the list of cherry pick commits:
```bash
git add files changed but not staged in this dev session #add to staging
git commit -m "commit on local/master"
cat .git/HEAD >> ./git/info/CHERRY_PICK_COMMITS #not needed if using provided hook
```
The last line is also included in the above provided git hooks so you don't need that if installing the hooks.
Then when finished with development do that last commit:

```bash
git add files changed but not staged in this dev session #add to staging
git commit -m "Final commit on local/master"
cat .git/HEAD >> ./git/info/CHERRY_PICK_COMMITS #not needed if using provided hook
```

**Step 4. Cherry Pick onto dev:**
Time to commit changes ONLY to `devrepo/dev` using cherry picking. This is also doable in one stage using rebasing. However, we are going to do it via repeated cherry picks that will be run for each commit starting from the latest git merge of origin into master. Take commits after the last merge into master C1,C2,C3,.... In case a merge from origin into master has resulted in changing of a file on dev then need to take this merge commit as well: M (client side git hooks can be used to store these in a file)

```bash
git checkout master 
git log 
git checkout dev #move to dev for cherry-picking onto it
git cherry-pick [-m(1|2) M] C1 C2 C3 ... #also need to determine which parent of the merge? -m1 or -m2? TODO?
```

Resolve conflicts: if any modified file is not in dev then:

```bash
git add newly added files
git cherry-pick --continue
git push devrepo dev #push changes back upstream (server side git hooks ONLY ALLOW PUSH FROM dev to dev)
```


**TODO:** NEVER PUSH MASTER TO `devrepo` (use server side git hooks for this) Only allowed to push from `dev` branches to `dev` branches on `devrepo`

**TODO:** NEVER ALLOW BRANCHING OFF master; ONLY BRANCH OFF dev for different topics of development

**TODO:** NEVER allow push while on local/master because this will take all the history on master including base repo files into `devrepo`! (server side git hooks)

**TODO:** NEVER allow commits while on `local/dev` branch (client side git hooks). `local/dev` branch only updated with cherry-picking or rebasing using automatic scripts (client side git hooks)


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


