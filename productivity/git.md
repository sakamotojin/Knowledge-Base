# Git

Git is a Distributed Version Control System commonly used in software industry . Version control is a system that records changes to a file or set of files over time so that you can recall specific versions later. Common Terminologies associated with git are&#x20;

* Pull  - fetch changes from somewhere and reflect in working directory&#x20;
* Push - send your changes to remote branch
* Rebase - pull changes from some other branch and add your changes on top , a common version(commit) is req for this to happen

![](<../.gitbook/assets/Screenshot 2022-02-06 at 1.05.48 AM.png>)

* Merge - add all changes in your working directory&#x20;

![](<../.gitbook/assets/Screenshot 2022-02-06 at 1.05.32 AM.png>)

* Local Branch - your local version of code
* Remote Branch - repo not in local , generally common a central repo where all devs push there changes
* Remotes - list of remote repos/branches
* origin - name given to remote repo

General Guideline While Raising A Pull Request&#x20;

* Make sure your branch is up to date with the master branch
* Run git rebase -i master.
* You should see a list of commits, each commit starting with the word "pick".
* Make sure the first commit says "pick" and change the rest from "pick" to "squash". -- This will squash each commit into the previous commit, which will continue until every commit is squashed into the first commit.
* Save and close the editor.
* It will give you the opportunity to change the commit message.
* Save and close the editor again.
* Then you have to force push the final, squashed commit: git push --force-with-lease atiq Squashing commits can be a tricky process but once you figure it out, it's really helpful and keeps our repo concise and clean.

#### Common Commands

`git init`

`git fetch <remote> <rbranch>:<lbranch>`

`git rebase <remote>/<rbranch>`

`git push <remote> <branch>`

`git branch`

`git log`

`git remote add origin git@github.com/test.git`

`git add .`

`git status`

`git commit -m <message>`

`git diff <branch>`

`git diff branchA...branchB`

`git branch`

`git checkout branch`

`git checkout -b newBranch`

`git stash`

`git push -u origin localBranch:remoteBranchToBeCreated`&#x20;



