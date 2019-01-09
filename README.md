# Welcome to git practice!
This repo is a safe place to practice the basics of git. Try out a rebase without risking any important code! Learn how to sync from an upstream repo! Preactice resolving merge conflicts!
This guide assumes you have a comfortable knowledge of unix shell commands. If you want a refresher, [try here](https://github.com/you-dont-need/You-Dont-Need-GUI). This guide also assumes you have an existing github account and have [added your ssh key to it](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)
This guide is designed as a space to practice git commands, not just teach them. If you are just after a quick reference, I recommend http://rogerdudler.github.io/git-guide/

## What are git and GitHub?
Git is a version-control system. It tracks changes to files and allows people to collaborate on projects. Git is often used for software projects, but can be used for any project, and is especially powerful for projects based on text files. A project in git is called a "repository" (or repo for short). A repo is a bunch of files and directories. Each repo includes a .git directory which contains information on every change made to the repo.

GitHub is an online hosting service for git. While the bulk of your work with git takes place on your own computer, it is often useful to sync your local repo with a "remote" repo (on GitHub), allowing you to work on multiple computers or collaborate with others on a project.
There are plenty of alternatives to GitHub, such as Stash, GitLab, or self-hosting your remote repos.

## Init
To create a new repo, make a directory and use `git init` inside it.
```bash
mkdir myDummyRepo
cd myDummyRepo
git init
```
That's it, real simple. You should see a `.git` directory in there now.
```bash
ls -a # directories starting with . are normally hidden, so use -a to show all
```
We won't be using this new repo, so you can go ahead and delete it.
```bash
rm -rf myDummyRepo
```

## Fork
Fork this repo into your github account by pressing the Fork button on github (top right of the github page for this repo). This will create a fork (copy) of this repo in your github account so you have control of it. Forking is useful when you want to propose changes to an existing repo or use the repo as the basis for your own project.

## Clone
`clone` copies an existing remote repo onto your computer. Let's clone the forked repo from the previous step. We will use ssh when cloning so that we can interact with the remote without having to worry about credentials.
```bash
git clone git@github.com:danrs/xwing.git
```
If you're cloning a repo that you don't own, you will have to clone using https instead of ssh. You will also be unable to push changes to a repo that you don't own.

## Add and Commit
Now that we have a local clone of the repo, let's make a change to it.
```bash
echo "It is a period of civil war" > crawl.txt
git status
```
The `git status` command shows that we have an unstaged change. To record a change in git, we first have to stage that change to the index, and then commit all staged changes.
```bash
git add crawl.txt # stage the change
git status # shows that our change is staged and ready to commit
git commit -m "I made a change" # commit staged changes with a message
```
The staging step seems unnecessary in this case, but it is useful on complex projects where you are changing multiple files. Let's try an example
```bash
echo "Rebel spaceships, striking from a hidden base, have won their first victory against the evil Galactic Empire." >> crawl.txt # modify our first file
echo "I love potatoes" > newFile.txt # create a new file
git status
```
We now have two changes, and we want to commit them separately, which we can do using the staging command. Try running `git status` after each command below to see the changes you are making.
```bash
git add crawl.txt
git commit -m "Started the crawl"
git add newFile.txt
git commit -m "started my food blog"
```

## `HEAD` and the git tree
In git, the `HEAD` is the pointer to the current branch reference. That means it points to the current commit on the current branch. Moving the head is how we move to newer/older commits or move between branches. You can think of the head as a "you are here" marker. Let's take a look at where we are
```bash
git log
```
You can see `HEAD` at the top, but this isn't a particularly clear visualisation of the git tree. If you have a graphical git program you can use it to examine the git tree, but I prefer to stay on the command line.
```bash
git log --graph --decorate --pretty=oneline --abbrev-commit --all # display a much prettier log
```
That's much better! We can see `HEAD` clearly at the top, pointing to the `master` branch. Other branches are visible too, but ignore those for now.

Let's save that command for us to use later. This step is optional, but recommended. Credit to [Conrad parker](http://blog.kfish.org/2010/04/git-lola.html) for the lola command.
```bash
git config --global alias.lola 'log --graph --decorate --pretty=oneline --abbrev-commit --all'
git lola
```

## Branches
As you saw from `git lola`, our git tree has multiple branches. Let's list them all:
```bash
git branch -l # show all local branches (should only be master)
git branch -r # show all remote branches
git branch -a # show all branches
```
Each branch is a pointer to a snapshot of your project. If you are working on multiple different featurs, always use a new branch for each.

### New branch
Let's make a new branch
```bash
git branch myFirstBranch # create a new branch
git lola
```
What has happened? Not a lot. We've created a new branch (snapshot of our code) but that's it. `HEAD` still points to the `master` branch, so any code changes we make will be performed on `master`, not our new branch. To move `HEAD` to a different branch, use the `checkout` command.
```bash
git checkout myFirstBranch
git lola
```
Now, we can see `HEAD` is pointing to our new branch. Often, you will want to go to a branch immediately after creating it. You can do so in a single command:
```bash
git checkout -b "foodBlog" #create a new branch and switch to it immediately
git status
```

If we commit changes to our new branch, we will see it become different to our master branch:
```bash
echo "You have failed me for the last time" >> crawl.txt
git add crawl.txt
git commit -m "Added Vader quote"
git lola
git diff master # compare with master branch
```

## reset
### Resetting to a commit
Now, let's try altering the tree with `reset`. This command moves the branch that `HEAD` points to (and moves `HEAD` along with it). The command is called "reset" because it "resets" or "undoes" changes.

The `reset` command has several modes, so let's start with `--soft`. We will be resetting to the parent of the current commit using the shortcut `HEAD~`. You can also move to a specific commit by specifying the commit hash (visible in `git log` or `git lola`) instead of `HEAD~`.
```bash
git checkout master
echo "clean up" >> mess.txt
git add mess.txt
git commit -m "added mess"
git reset --soft HEAD~
git lola
```
You can see that the master branch has moved to previous commit, and `HEAD` has moved along with it. None of the files have changed, though, and if we run `git status` we see that mess.txt is staged ready to be committed. `git reset --soft HEAD~` has essentially undone our latest commit.

Next up is the default behaviour for reset: `--mixed`. This will still not change any files, but it will unstage any changes as well as undoing commits. Run `git lola` and `git status` after each command below to see the changes you are making.
```bash
git commit -m "commit the changes we just unstaged"
git reset HEAD~
```
After running the default (`--mixed`) `reset`, you can see that `mess.txt` is present, but unstaged. We have undone the commit and unstaged the file.

Finally, the most dangerous option: `reset --hard`. This command will undo commits, unstage changes, AND remove those changes from any files. Your changes will be obliterated forever\* so use with caution!
\*see Reflog
```bash
git add . # stage everything we just unstaged
git commit -m "commit the changes we just unstaged"
git reset --hard HEAD~
```
If you look at your working directory, you can see that `mess.txt` is nowhere to be found, and `git status` is clean.

In summary:
`reset --soft`: Move branch that `HEAD` points to (undo commits)
`reset`: (`--mixed`) Move `HEAD` branch and make the staging index look like the new HEAD location (undo commits and unstage changes)
`reset --hard`: Move `HEAD` branch, make staging index look like HEAD, and make the working directory look like the index (undo commits, unstage changes, and remove all changes. Use with caution).

In the code above, we've been using `reset` to move to commits on the same branch, but there's nothing stopping you from using commits on different branches.

#### Squashing commits
Reset is handy as an undo button, but let's try a more interesting example. Imagine we are a busy developer and we've made a whole pile of useless commit messages.
```bash
git checkout -b butternut
touch pumpkin.txt
git add pumpkin.txt
git commit -m "vegi"
echo "dear diary" >> pumpkin.txt
git add pumpkin.txt
git commit -m "yep"
echo "today I ate a radish" >> pumpkin.txt
git add pumpkin.txt
git commit -m "oops"
echo "I mean, I ate a pumpkin. What a day" >> pumpkin.txt
git add pumpkin.txt
git commit -m "max"
git lola
```
Oh dear - those commit messages are useless. Lets replace them with a single commit message
```bash
git reset HEAD~4
git add pumpkin.txt
git commit -m "Started a log of my food habits"
```
Now return to the master branch for the rest of our work
```bash
git checkout master
```

### Resetting a file
`reset` can also be used on a specific file. If you do this, `HEAD` won't move, but all other `reset` actions will be carried out. Let's try it. Run `git status` as you go to see the changes.
```bash
git checkout master
echo "The empire did nothing wrong" > protest.txt # make a change
git add protest.txt # stage the file
git reset protest.txt # unstage the file
git status
rm protest.txt # clean up our example
```

## checkout
### checking out a commit
Like `reset`, the `checkout` command manipulates the git tree and moves `HEAD` to a specific commit. `git checkout [commit]` is similar to `git reset --hard [commit]`, but `checkout` only moves `HEAD`, not the branch that `HEAD` points to. I like to think of `checkout` as a way to move about the tree, while `reset` is a way to change the tree. Let's try it out.

```bash
git checkout HEAD~
git lola
```
After the checkout, we are now in a "detached head" state - no longer working on a branch. Unlike before when we did `git reset`, we can still see all our previous commits in the git tree, and we can return to our previous branch easily
```bash
git checkout master
git lola
```

`checkout` will also try to merge any uncommitted changes, and stop us if they would be overwritten when checking out the new location.
```bash
echo "stop" > crawl.txt
git status
git checkout HEAD~5
```
git won't let us checkout because we have uncommitted changes

### checking out a file
You can also use `checkout` on a file path. This will not move `HEAD`, but it will unstage any changes to that file in the index and also overwrite that file. `git checkout [branch] file` is equivalent to `git reset --hard [branch] file`, except that the latter command does not exist.

Let's use checkout to remove our unwanted changes to crawl
```bash
git checkout master crawl.txt
```

Reset and checkout are both powerful tools, and with great power comes great complexity. I highly recommend [reading further](https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified) on these two commands.

## Reflog
When I said that `reset --hard` would remove a commit forever, I wasn't quite telling the truth. `git reflog` is available as a last resort option. This command shows a reflog (history) of each time the tip of a branch was updated. If you have a good memory and good commit messages, you can use reflog to recover commits that are no longer in the git tree.

```bash
git reflog # show reflog for HEAD
git reflog show --all # show reflog for all branches

git checkout blizzard
git reset --hard HEAD~ # oops, I just lost a commit that I wanted
git reflog show blizzard # show reflog for blizzard branch amnd search for lost commit
git reset <commit-hash> # put the commit hash in here, no <>
git status # check that the changes are as expected
git reset --hard <commit-hash> # put the commit hash in here
```

## Stash and pop
Sometimes you want to temporarily store some work but not commit it (eg if you want to quickly check out another branch, or if you realised you had started working on the wrong branch. `git stash` will let you do this. `git stash pop` will recover the most recently stashed changes (stashed changes are stored using a stack).
```bash
git checkout master
echo "temporary mcTempFace" >> temp.txt
git add temp.txt
git stash # temporarily store our work because this is the wong branch
git checkout blizzard # move to correct branch
git stash pop # recover stored work
git add .
git commit -m "added something temporary"
```
Never use stash for long term storage, because I guarantee you will forget what you had stored and what branch it was meant to go on. If you want to store something for more than a day and you don't want it on any of your branches, just put it on a new branch in the relevane place.

## Merging
When you want to incorporate code from one branch into another branch, it's time for a `merge`.

### Auto-resolved merge
Most merges are straightforward affairs, and git handles them completely automatically
```bash
git checkout merge-target
git merge simple-feature
git lola
```

### Merge conflicts
#### Manual resolution
Sometimes, a merge will require manual intervention
```bash
git checkout merge-target
git merge complex-feature
git status
```
Bup bow. The merge hit some conflicts, and git isn't sure how to resolve them. `git status` tells us that these conflicts are in `crawl.txt`, so open that file in your favourite editor. You will see conflict markers in your text like so:

```
<<<<<<< HEAD
It is a planet of civil war.
=======
It is a period of PARTIES and maybe war.
>>>>>>> complex-feature
```

The text between `<<<<<<< HEAD` and `=======` is from the `merge-target` branch, while the text between `=======` and `>>>>>>> complex-feature` is from the complex-feature branch. To resolve the conflict, edit the text as you see fit and then delete the conflict markers. When you are finished, save your work and return to the terminal.
```bash
git add crawl.txt
git merge --continue
```

#### Ours vs theirs
Sometimes when you have merge conflicts you will just want to take all the changes from a specific branch whenever there is a conflict.
```bash
git checkout merge-target
git merge complex-feature-2
```

When merging, "ours" means the branch you are merging into, while "theirs" means the feature branch you are merging from. In our case, we are merging into `merge-target` and we are merging from `complex-feature-2`. We have decided that we only want the changes from `complex-feature-2`, aka "theirs", so we can avoid doing a full manual merge (hooray).
```bash
git checkout --theirs funky.txt # take all changes from feature branch
git add funky.txt
git merge --continue
```

## Remotes
A remote is a remote copy of a repository. In most projects, there is a single remote repository which is used as a central source of truth.

### Pull
`pull` syncs your local branch with a remote branch. It works by performing a `fetch` (downloads latest data from the remote) and then `merge`s the fetched data into the current branch. In your case there will be nothing to update, as no changes have been made on the remote.
```bash
git checkout master
git pull origin master
```

### Push
`push` syncs the remote branch with your local branch.
```bash
git checkout master
echo "Porgs" >> crawl.txt
git add crawl.txt
git commit -m "added a cute animal thing"
git push origin master
```

### checkout branch from remote
When you cloned this repository, it was created with a single remote, called `origin`. Let's try that now.

```bash
git fetch # update remote info
git checkout -b origin/check-me-out # create a new local branch to track remote branch
```

### Adding remotes
Since you have forked this repository, you want to keep it up to date with the original repository you forked it from. To do so, add the original repo as a new remote called "upstream:
```bash
git remote add upstream https://github.com/danrs/git-practice.git # add new remote
```

### Sync upstream (fetch and merge)
Now that you've added a new upstream, you can sync upstream branches with local branches
```bash
git checkout master
git fetch upstream
git merge upstream/master
```
Again, this specific example won't do anything unless I've just updated the original repo


## Rebasing
Rebasing moves one or more commits to new base commit. If you want to incorporate changes from a parent branch into your working branch (but you don't want to merge back into the parent branch) then rebase is what you want. Warning: [don't rebase public branches](https://benmarshall.me/git-rebase/)
### Auto-resolved rebase
As with merging, most rebases are trivial.
```bash
git checkout rebel-alliance
git lola
git rebase hoth
git lola
```
Examining the git tree, you can see that the start of the `rebel-alliance` branch has moved to the end of the hoth branch

### Merge conflicts while rebasing
Some rebases will cause merge conflicts
```bash
git checkout yoda
git lola
git rebase hoth
git lola
```
As you did in the merging section, open your favourite editor and resolve any merge conflicts. Use `git status` to see conflicting files. When you are done, add the files and continue rebaseing.
```bash
git add .
git rebase --continue
```

#### ours vs theirs
```bash
git checkout palp
git rebase hoth
```
During a rebase, git checks out the branch you are rebasing onto and replays your branch changes on top of it. Thus, "ours" means the branch you are rebasing onto and "theirs" beans the branch that is being rebased. In this case, we are rebasing palp ("theirs") onto hoth ("ours"). As with merging, we can make use of these terms to avoid a complex merge. Let's say we want to keep changes in the hoth branch whenever there's a conflict

```bash
git checkout --theirs plans.txt
git add plans.txt
git rebase --continue
```

## Cherry-picking
Cherry picking allows you to pick a commit from one branch and apply it to another. Lets say we want to apply commit with the message "PICK ME" from the `cherries` branch onto the `farm` branch.
```bash
git checkout farm
git cherry-pick <commit-hash> # insert the correct hash here
git lola
```
