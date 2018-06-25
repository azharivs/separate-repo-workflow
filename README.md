
# test
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
vi .gitignore #add *.* and * to .gitignore which will only be part of this branch (dev)
git add -f .gitignore
git commit -m "added special .gitignore for dev branch"
git push -u devrepo dev #push the skeleton of the dev branch to the devrepo

########### get base code from origin repo

git checkout master #always do work on master branch
git pull origin master #get main code base from origin/master. This is possibly followed by merge into local master
git commit -m "after merge or origin/master into master"

########## pull in work from dev (devel) repo and marge into local master (if needed)
git pull devrepo dev #possibly followed by merge
git commit -m "merge in devrepo/dev into local/master"

########### start developping on local/master .....
#....
commit -m "Final commit on local/master"

########### time to commit changes ONLY to devrepo/dev use a modified rebase
export CUR_MASTER_HEAD = cat .git/refs/heads/master #store current master HEAD SHA-1
git checkout master
git rebase dev #rebase master onto dev so dev is updated with all diffs on master since ancestor
git checkout dev
git merge master #fast forward dev pointer to rebased one (latest)
git checkout $CUR_MASTER_HEAD #restore master head to original value
vi .gitignore #make exception for appropriate files to .gitignore (dev) (only those changed during devel and not those pulled from origin but not changed locally)

# what if git pull origin adds/modifies some files not touched by devrepo? 
# we don't want these to go into devrepo

# what if some file f1 is modified via devrepo but is then also modified (or even deleted) via origin?
# both versions of f1 should be merged and rebased onto devrepo upon pulling from origin

# the commit of origin corresponding to each commit of devrepo should be stored in a certain file via githooks


########## ... or branch off local master 

```
