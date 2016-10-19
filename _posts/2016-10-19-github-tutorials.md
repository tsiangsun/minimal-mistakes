---
title: "Github tutorials"
layout: single
comments: ture 
tags: github 
categories: programming 
---


## Most useful git commands

Example: local folder and remote on Github  
Sync existing local DataVisual repo to Github.com, first create a new repo online with URL, then at current folder DataVisual  

```bash
$ git init
$ git add *.cpp
$ git commit -m 'Version 1.0'     
$ git commit -a -m "Version 1.1"   # update all files and add message
$ git remote add origin http://github.com/tsiangsun/DataVisual.git
$ git clone https://github.com/tsiangsun/DataVisual.git #first time download existing repo from remote to local
$ git remote -v 
$ git merge origin/master  # merge remote branch with local current branch
$ git push origin master   # upload local master branch to remote origin/master
$ git fetch origin  	   # download remote files that are not in local repo, but don't merge with current branch
$ git pull origin master   # download remote branch history and merge changes automatically
```

## Git Tutorial 
For MaxOSX systems, for other platforms see Git official site [http://git-scm.com](http://git-scm.com)  and useful Github website tutorials available on [https://guides.github.com/](https://guides.github.com/)

### [1]  Install git from http://sourceforge.net/projects/git-osx-installer/  
or if you have macports installed already, type the following in command line  

```bash
$ sudo port install git-core +svn +doc +bash_completion +gitweb
```  
[Git reference](http://gitref.org/index.html) // 
Pro Git book here in [English](http://git-scm.com/book/en/v2) and in [Chinese](http://git-scm.com/book/zh/v1) // [A Visual reference of git](http://ndpsoftware.com/git-cheatsheet.html)

### [2]  Install Github for Mac 

from [https://mac.github.com/](https://mac.github.com/) 

### [3] Version control - getting started

#### (a) Config tooling
Set name and email and enable colorization of command line output:  

```bash
$ git config --global user.name "Xiang Sun"
$ git config --global user.email "tsiangsun@gmail.com"
```
See config info

```bash
$ git config --list   # to see config info for current location
$ git config user.name  # to see user's name
```
Enable coloration of command line output

```bash
$ git config --global color.ui auto
$ git help config
```

#### (b) Create a repository (repo)
Creates a new local repository at current location with the specified name by (also create a .git folder in current location)  

```bash
$ git init [project-name]
$ git init my-frist-repo
```

Add files for git to track (a.k.a. put files to the staged area)

```bash
$ git add [file_name(s)]
$ git add *.c
$ git add README
```

or Download a existing project and its entire version history by

```bash
$ git clone [url] 
$ git clone https://github.com/tsiangsun/Hello-World.git
$ git clone [url] [new_repo_name]
```

Delete local repo:  Delete the .git directory in the root-directory of your repository if you only want to delete the git-related information (branches, versions). 
If you want to delete everything (git-data, code, etc), just delete the whole directory.



#### (c) Show status of current git repo

```bash
$ git status
```

See difference between the current staged and previous version

```bash
$ git diff --staged  or  $ git diff --cached 
```

See difference between the current unstaged and the staged 

```bash
$ git diff
```



#### (d) Submit staged update for a new version - commit

```bash
$ git commit  # commit by revoking vi editor or user defined editor
$ git commit -m 'version 1.0'  # commit repo with message version 1.0
$ git commit -a -m 'version 1.1'  # commit all files without call git add to stage files
```


Exclude tracking:  Text file _.gitignore_ exclude temp files from tracking, for example, in _.gitignore_

```
*.log  
*.[oa]
build/
temp-*
```
To list all ignored files

```bash
$ git ls-files --other --ignored --exclude-standard
```


#### (e) Refactor filenames 

Remove file from the staged area and delete it from the working dir (untrack and delete a file)

```bash
$ git rm [file_name]
$ git rm log/\*.log  # remove any file with .log extension in log/ subfolder
$ git rm --cached [file_name] # only remove tracking, but leave the file in local current folder
```



#### (f) Rename or move a file

```bash
$ git mv file.old file.new
```


#### (g) Review history of commits

```bash
$ git log -p # -p only show difference by patch
$ git log --word-diff # only show diff by word
$ git log --stat  # show diff statistical info
$ git log --follow [file_name]  # list version history of a file, including renames
$ git log -p -3 # show the latest 3 commits
$ git log --since=2.weeks # show the last 2 weeks' commits; --before,--author, --committer
$ git show [commit]  # show metadata and content of changes of specified commit
```


#### (h) Undo changes (You can get any file that has been committed in git)

```bash
$ git commit --amend  # undo the latest commit
$ git reset HEAD [file_name]  # undo a staged file
$ git checkout -- [file_name] # discard changes in working dir, return to unchanged version
```


#### (i) Remote repo 

```bash
$ git remote -v  # list all remote repo
$ git remote add [remote_name] [url]   # name a remote repo
$ git fetch [remote_name]  # fetch remote files that was not in local to local address, but don't combine with current branch
$ git push [remote_name] [branch_name]  # push local data to remote repo
$ git push origin master # is equivalent to
      $ git clone # by default 
$ git remote show [remote_name] # see info of remote repo, like origin
$ git remote rename [old_remote_name] [new_remote_name] # rename remote short name
$ git remote rm [remote_name]  # delete remote repo
```

### [4] Git Branching -- multiple version paths

#### (a) Local branches

```bash
$ git branch [branch_name] 	 	# Create a new branch (add a new path of version history)
$ git checkout [branch_name] 		# Switch to another branch
$ git merge [other_branch_name] 	# Merge current branch with another branch
$ git branch -d [branch_name]  	# Delete a branch
$ git branch # options: -v, --merged, --no-merged  # List all branches, * marks current branch
```

e.g.  Create a new branch called issue53 and switch to this branch 

```bash
$ git checkout -b issue53
$ vim index.html
$ git commit -a -m 'added a new footer [issue 53]'    # submit all changes to resolve issue53
$ git checkout master 
$ git merge issue53   # merge issue53 with current branch (master)
$ git branch -d issue53    # delete branch issue53
```

e.g. if two branches have conflict content in the same location of the same file, auto merge will ask you to choose from unmerged path and commit yourself as follows OR use graphical tool `$ git mergetool`.

```bash
$ git merge issue53
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.

$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")
Unmerged paths:
  (use "git add <file>..." to mark resolution)

        both modified:      index.html

no changes added to commit (use "git add" and/or "git commit -a")

in the file index.html you will see the following part highlighted
<<<<<<< HEAD
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
  please contact us at support@github.com
</div>
>>>>>>> issue53

you can replace this part with the following and save the file
<div id="footer">
please contact us at email.support@github.com
</div>
```

Then add the updated index.html to staged area and commit this latest change

```bash
$ git add index.html
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   index.html
$ git commit 
Merge branch 'issue53'

Conflicts:
  index.html
#
# It looks like you may be committing a merge.
# If this is not correct, please remove the file
#       .git/MERGE_HEAD
# and try again.
```

#### (b) Remote branches  format: [remote_repo]/[branch]

Download all history from remote bookmark repo (recommended)

```bash
$ git fetch [remote_repo]
```

Combine remote branch into current local branch

```bash
$ git merge [remote_repo]/[branch]
```

Uploads local branch commits to remote repo

```bash
$ git push [remote_repo]  [branch]
```

Download remote bookmark history and incorporates changes ~ git fetch + git merge (not highly recommended)

```bash
$ git pull [remote_repo]  [branch]  
```

> When you use pull, Git tries to automatically do your work for you. It is context sensitive, so Git will merge any pulled commits into the branch you are currently working in. pull automatically merges the commits without letting you review them first. If you don’t closely manage your branches, you may run into frequent conflicts.

> When you fetch, Git gathers any commits from the target branch that do not exist in your current branch and stores them in your local repository. However, it does not merge them with your current branch. This is particularly useful if you need to keep your repository up to date, but are working on something that might break if you update your files. To integrate the commits into your master branch, you use merge.

##### 1) Remote branch works like a bookmark, when clone a remote repo, git automatically name the remote repo origin and create an origin/master branch pointer (mark version at the time of cloning) in local machine, and a  master branch pointer for local user,

```bash
$ git clone https://github.com/tsiangsun/Hello-World.git
```
to update origin/master branch pointer, we run 

```bash 
$ git fetch origin 
```

##### 2) Upload your local branch to remote so as to let your collaborator see or change.

```bash
$ git remote add teamone git://git.team1.yourcompany.com    # alias remote repo as teamone
$ git fetch teamone   # create a remote branch called teamone/master
$ git push origin serverfix  # upload local branch serverfix to remote repo origin
```

When your collaborator download remote repo, git only create a static origin/serverfix pointer, but don't allow the collaborator to change anything.

The collaborator can merge the remote branch origin/serverfix to his local repo.

```bash
$ git merge origin/serverfix
```

Or if your collaborator wants develop his own serverfix branch, he can create his own serverfix branch identical to remote branch origin/serverfix.

```bash
$ git checkout -b serverfix origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
Switched to a new branch 'serverfix'
```

##### 3) Remote-tracking branch, is a local branch checked out from remote branch which bonds with each other, like git checkout -b [local_branch] [remote_repo]/[remote_branch], so that  git push and git pull would work from the tracking branch. For git 1.6.2 of newer, one can simplify this command by using

```bash
$ git checkout --track origin/serverfix
Branch serverfix set up to track remote branch serverfix from origin.
Switched to a new branch 'serverfix'
```

##### 4) Delete remote branch. (Notice the following tricky syntax)

```bash
$ git push [remote_repo]  :[branch]
```

e.g. if one wants to delete serverfix branch on remote repo origin

```bash
$ git push origin  :serverfix
To git@github.com:schacon/simplegit.git
 - [deleted]         serverfix
```
