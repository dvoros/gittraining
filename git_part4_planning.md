# Git 4 - Tips and Tricks

This time we're trying to cover useful stuff that we couldn't fit
into the previous sessions.

## git stash

Quickly saving current messy state to be able to move HEAD:

```
git stash
```

> Question: How many files did this create under .git/objects?

List stashed stuff with:

```
git stash list
git stash 
```

And than reapply with:

```
git stash pop
# or
git stash apply
```

> Question: Has anyone ever applied stuff twice from stash? (:

## Interactive and partial commands

### Rebase

Interactive rebase lets you execute specific actions for every commit during
rebasing:
```
p, pick = use commit
r, reword = use commit, but edit the commit message
e, edit = use commit, but stop for amending
s, squash = use commit, but meld into previous commit
f, fixup = like "squash", but discard this commit's log message
d, drop = remove commit
```

> Question: which option have I removed from the list and why?

We've seen how we can edit previous commit message/content with `git commit --amend`.
Interactive rebase on the same commit is a great way of rewriting the history.
Use it to squash/split your commits to look more organized!

```
git rebase -i @{u}
```

!!! first command reword => squashed commit

### Add

```
git add -i
```

Let's you stage and unstage files using an interactive shell. The most useful
part of adding partial files is also available with:

```
git add -p 
```

## git worktree

Need the same project in multiple working directories? Cloning multiple times
could work, but you're duplicating most of the stuff under .git! Worktree
let's you manage multiple working trees, each with it's own branch checked out.

```
git worktree add ../otherwd otherbranch
```

## .gitignore files

Keep some files under the working directory from getting into the repository.

You can have .gitignore files in subdirectories of your project.

To exclude something without having to edit (and eventually commit) the
.gitignore files in a project, you can use a global .gitignore file for all
your projects. You should really consider adding those to the committed .gitignore
files though...

```
git config --global core.excludesfile
```

For a list of common rules see https://github.com/github/gitignore

## git clean

Removing stuff from the working directory that's not under version control.
Use the `-n` option to get a list of removal candidates without actually
removing them!

With `-x` you can delete files that are otherwise ignored (with .gitignore).
Be careful not to delete stuff your IDE is using though! It's probaby a good
idea to configure your IDE to keep it's meta files in a separate directory.

```
git clean -dfxn
```

## Bisecting

Useful to find out which commit have broken something. Start by pointing out
a good and a bad commit:

```
git bisect bad HEAD
git bisect good HEAD~~~
```

Git will do a binary search between the two endpoints by resetting HEAD to
specific commits and asking you to classify them as good/bad with:

```
git bisect good
# or
git bisect bad
```

You can further automate the process by providing a command that decides whether
if current state is good or bad:

```
#!/bin/bash

! cat file | grep BUG_IN_CODE
exit $?
```

Save that to `/tmp/bisecter.sh`, set executable and run with:

```
git bisect run /tmp/bisecter.sh
```

Exit from bisecting with

```
git bisect reset
```

## Extract folder into new repo

(Based on https://help.github.com/articles/splitting-a-subfolder-out-into-a-new-repository/)

A part of your project has matured into a project on its own? You can move that
to a new repository with keeping its history!

Create new branch and remove every commit from there that's not touching the
subdirectory:

```
git checkout -b subbranch
git filter-branch --prune-empty --subdirectory-filter subdir/ subbranch
```

Push this to another repo:

```
git remote add otherrepo ...
git push -f otherrepo HEAD:master
```

## Use browser plugin to view patches in the browser

For chrome: https://chrome.google.com/webstore/detail/git-patch-viewer/hkoggakcdopbgnaeeidcmopfekipkleg
For firefox: https://addons.mozilla.org/en-US/firefox/addon/ffdiff

## Hooks

Check out the samples under `.git/hooks`. My personal favorite is pre-push to
avoid pushing commits you didn't mean to:

```
# Check for WIP commit
commit=`git rev-list -n 1 -E --grep "^WIP|REVERT ME|DON'?T PUSH" "$range"`
if [ -n "$commit" ]
then
        echo >&2 "Found WIP commit in $local_ref, not pushing"
        exit 1
fi
```

!!! git global hook

## Dangers of resolving conflicts with `checkout --ours/--theirs`

Will get the _whole_ file from one of the sources, not only for the conflicting
parts!

## Using github as multiple users

Edit your `~/.ssh/config` to add:

```
Host github-OTHERUSER.com
        HostName github.com
        User git
        IdentityFile ~/.ssh/id_rsa_github_otheruser
```

Then add remote with:

```
git remote add otheruser git@github-OTHERUSER.com:otheruser/whatever.git
```

## Sharing Github links pointing to lines of a file

Always use "static" link, that points to a specific commit and not a branch!

*Don't do this:*

https://github.com/apache/hive/blob/master/ql/src/java/org/apache/hadoop/hive/ql/metadata/Hive.java#L5284

*Do this instead:*

https://github.com/apache/hive/blob/5e2a530cc857c36dc97de6f5bec4003c569d00bd/ql/src/java/org/apache/hadoop/hive/ql/metadata/Hive.java#L5284

Kucsi: Press `y` on Github! Wow!

## Diff options

View character diffs instead of whole lines:

```
--color-words=.
```

Generate diff without `a/` and `b/` prefixes before file names:

```
--no-prefix
```

## Some useful aliases

https://github.com/dvoros/dotfiles/blob/master/.shrc



## Others

Other stuff we didn't have time to discuss:

- Advanced merging with renameLimit, rename-threshold and rerere
- Handling tags from multiple remotes
https://stackoverflow.com/questions/22108391/git-checkout-a-remote-tag-when-two-remotes-have-the-same-tag-name
Edit .git/config with another `fetch=...` line
- annotated tags
- git hash VS sha1
- git submodule
- sparse checkout? clone only a part of the repo -> https://git-scm.com/docs/git-read-tree#_sparse_checkout
- git-crypt
- git lfs
- rebase stragety + options
- change user for a single commmit: `git -c user... commit ...`?
