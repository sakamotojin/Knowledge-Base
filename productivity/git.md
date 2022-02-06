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

### Common Commands

#### Getting & Creating Projects

| Command                                                           | Description                                |
| ----------------------------------------------------------------- | ------------------------------------------ |
| `git init`                                                        | Initialize a local Git repository          |
| `git clone ssh://git@github.com/[username]/[repository-name].git` | Create a local copy of a remote repository |

#### Basic Snapshotting

| Command                            | Description                                       |
| ---------------------------------- | ------------------------------------------------- |
| `git status`                       | Check status                                      |
| `git add [file-name.txt]`          | Add a file to the staging area                    |
| `git add -A`                       | Add all new and changed files to the staging area |
| `git commit -m "[commit message]"` | Commit changes                                    |
| `git rm -r [file-name.txt]`        | Remove a file (or folder)                         |

#### Branching & Merging

| Command                                              | Description                                             |
| ---------------------------------------------------- | ------------------------------------------------------- |
| `git branch`                                         | List branches (the asterisk denotes the current branch) |
| `git branch -a`                                      | List all branches (local and remote)                    |
| `git branch [branch name]`                           | Create a new branch                                     |
| `git branch -d [branch name]`                        | Delete a branch                                         |
| `git push origin --delete [branch name]`             | Delete a remote branch                                  |
| `git checkout -b [branch name]`                      | Create a new branch and switch to it                    |
| `git checkout -b [branch name] origin/[branch name]` | Clone a remote branch and switch to it                  |
| `git branch -m [old branch name] [new branch name]`  | Rename a local branch                                   |
| `git checkout [branch name]`                         | Switch to a branch                                      |
| `git checkout -`                                     | Switch to the branch last checked out                   |
| `git checkout -- [file-name.txt]`                    | Discard changes to a file                               |
| `git merge [branch name]`                            | Merge a branch into the active branch                   |
| `git merge [source branch] [target branch]`          | Merge a branch into a target branch                     |
| `git stash`                                          | Stash changes in a dirty working directory              |
| `git stash clear`                                    | Remove all stashed entries                              |

#### Sharing & Updating Projects

| Command                                                                           | Description                                                 |
| --------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| `git push origin [branch name]`                                                   | Push a branch to your remote repository                     |
| `git push -u origin [branch name]`                                                | Push changes to remote repository (and remember the branch) |
| `git push`                                                                        | Push changes to remote repository (remembered branch)       |
| `git push origin --delete [branch name]`                                          | Delete a remote branch                                      |
| `git pull`                                                                        | Update local repository to the newest commit                |
| `git pull origin [branch name]`                                                   | Pull changes from remote repository                         |
| `git remote add origin ssh://git@github.com/[username]/[repository-name].git`     | Add a remote repository                                     |
| `git remote set-url origin ssh://git@github.com/[username]/[repository-name].git` | Set a repository's origin branch to SSH                     |

#### Inspection & Comparison

| Command                                    | Description                    |
| ------------------------------------------ | ------------------------------ |
| `git log`                                  | View changes                   |
| `git log --summary`                        | View changes (detailed)        |
| `git log --oneline`                        | View changes (briefly)         |
| `git diff [source branch] [target branch]` | Preview changes before merging |

#### Miscellaneous&#x20;

| Command                                  | Description |
| ---------------------------------------- | ----------- |
| `git fetch <remote> <rbranch>:<lbranch>` |             |
| `git rebase <remote>/<rbranch>`          |             |
