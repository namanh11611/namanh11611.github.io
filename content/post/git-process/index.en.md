---
title: "A Proper Workflow with Git"
description: "After some time working and moving through a few companies, I realized that each place has a different workflow with Git. This article introduces the Git workflow that I think is proper and is also being used at my current company."
date: 2020-11-10T17:19:00+07:00
slug: git-process
image: git.webp
categories: [Technical]
tags: [Git]
---

# Introduction

After some time working and moving through a few companies, I realized that each place has a different workflow with Git. This article introduces the Git workflow that I think is proper and is also being used at my current company. So, I won't introduce all Git commands, just the ones I think are enough for your daily work.

# Workflow

## First day at work

Simple, right? On your first day, the only command you need is `git clone`. When you want to get the team's source code, just open the terminal and type:

```shell
git clone <url>
```

A small tip: if you want the folder name after cloning to be different from the project name on remote, just add the folder name at the end:

```shell
git clone <url> folder_name
```

## Normal days

### Working alone

The boss assigns a new feature, let's get started. But wait, if you're on another branch, don't forget to checkout the team's main branch (usually called master):

```shell
git checkout master
```

Then pull the latest code:

```shell
git pull
```

To check if your local code is up to date, try using `git log`. I usually use this for a concise view:

```shell
git log --oneline
```

Then, checkout a new branch to start your feature. Adding the `-b` param will create and switch to the new branch:

```shell
git checkout -b feature_branch
```

...

Code... code... code......

...

Done! Now add the files you changed to the stage. Most IDEs support quick add and commit, but if you want to do it manually:

```shell
git add .
```

This adds all changed files to the stage. Then commit:

```shell
git commit -m "Fix all bugs"
```

**Note:** Branch and commit names should be clear, indicating what feature or bug fix they relate to. This depends on your team's rules. Some teams use the task ID as a prefix, others use the purpose, like feature/fixbug/...

Finally, push your code to the repository:

```shell
git push origin feature_branch
```

Now go to the repository and create a merge request for your boss to review. While waiting, grab a coffee.

### Feature with multiple contributors

If your branch has multiple people working on it, and someone else pushed before you, before pushing, pull like this:

```shell
git pull --rebase
```

Your commit will be placed on top of your colleague's in the log.

If you want to fetch code but not merge yet, use:

```shell
git fetch
```

In my opinion, `pull = fetch + merge`.

### Merging code

After review, your boss agrees to merge, but during review you added, changed, or deleted some files. You want to rebase those commits into one, or just edit or delete a commit. Suppose you have 3 commits to combine:

```shell
git rebase -i HEAD~3
```

The terminal gives you options like edit, reword, squash... Change 'pick' to the option you want. Press Ctrl + O to save, then Ctrl + X to exit.

Another issue: someone else pushed to master. You can still merge, but it will create a merge commit. I usually rebase and merge fast forward.

First:

```shell
git fetch
```

Then rebase. You must be on `feature_branch`:

```shell
git rebase origin/master
```

Simply put, rebase gets the latest code from master, then *"rewrites"* your feature branch to put your commits on top.

Finally, force push to your feature branch. Force push applies your local log to the repo branch, regardless of differences:

```shell
git push -f origin feature_branch
```

If you're the only one on the branch, force push is fine. But **be careful when force pushing to a branch with multiple contributors**, as it can cause conflicts for others. Only do this when you're sure your feature is done.

Then, merge code via the merge request on the repo. Task complete!

## Crisis days

### Reset

Sometimes you make a mistake and need to revert code. Git reset has 3 options for you.

Reset commit but keep code in stage, ready to recommit:

```shell
git reset --soft commit_id
```

Reset commit and remove code from stage. You need to use `git add` before recommitting:

```shell
git reset --mixed commit_id
```

Reset commit and delete all code you did:

```shell
git reset --hard commit_id
```

### Stash

You can use this as a lifesaver to temporarily save code before rebasing or checking out another branch that has conflicts. Think of it as a stack-structured scratchpad.

To stash all current changes:

```shell
git stash
```

To apply the last stash:

```shell
git stash pop
```

I mainly use Android Studio now, which has Shelf with similar functionality, so I don't use git stash much anymore.

You can read more [here](https://git-scm.com/docs/git-stash).

# Conclusion

Honestly, the title is just clickbait—there's no such thing as a "proper" workflow. Every company and project is different. If your project is small and speed is a priority, you might skip some steps and push straight to master. If your project is big and strict, you might not allow force pushes to remote. So, what's your company's workflow? Share it with me!

Thanks for reading!
