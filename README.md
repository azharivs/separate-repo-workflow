
# test
```bash
mkdir local_repo
cd local_repo
git init
git remote add origin https://origin_url
git remote add testrepo http://testrepo_url #for development
touch dummy
git add dummy
git commit -m "create dummy and commit to next allow branching"
git branch test #for development
git checkout test
vi .gitignore #add *.* and * to .gitignore which will only be part of this branch (test)
git add .gitignore
git commit -m "added special .gitignore for test branch"
git push -u testrepo test #push the skeleton of the test branch to the testrepo

########### get base code from origin repo

git checkout master #always do work on master branch
git pull origin master #get main code base from origin/master. This is possibly followed by merge into local master
git commit -m "after merge or origin/master into master"

########## pull in work from test (devel) repo and marge into local master (if needed)
git pull testrepo test #possibly followed by merge
git commit -m "merge in testrepo/test into local/master"

########### start developping on local/master .....
#....
commit -m "Final commit on local/master"

########### time to commit changes ONLY to testrepo/test use a modified rebase
export CUR_MASTER_HEAD = cat .git/refs/heads/master #store current master HEAD SHA-1
git checkout master
git rebase test #rebase master onto test so test is updated with all diffs on master since ancestor
git checkout test
git merge master #fast forward test pointer to rebased one (latest)
git checkout $CUR_MASTER_HEAD #restore master head to original value
vi .gitignore #make exception for appropriate files to .gitignore (test) (only those changed during devel and not those pulled from origin but not changed locally)

# what if git pull origin adds/modifies some files not touched by testrepo? 
# we don't want these to go into testrepo

# what if some file f1 is modified via testrepo but is then also modified (or even deleted) via origin?
# both versions of f1 should be merged and rebased onto testrepo upon pulling from origin

# the commit of origin corresponding to each commit of testrepo should be stored in a certain file via githooks


########## ... or branch off local master 

```
