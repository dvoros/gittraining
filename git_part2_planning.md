# Git part 2 - branching

## Recap

  - Content addressable object store
  - Storing changes to files without being able to change anything
  - Object types (blob, tree, commit)

## Intro

What are branches?
  - ability do diverge development from main line
  - used to be an expensive thing in older VCS tools
  - nearly instantaneous operations in git
  - a pointer to a commit

## Getting started
```
git init
git status
```

=> On branch Master

master is the name of the default branch

```
echo zebra > animals
git add animals
git commit -m 'init animals'
git log --deco
```

We're gonna use `git log --deco` a lot, create an alias:

```
alias gl="git log --deco"
```

=> HEAD -> master

Lied when I said HEAD is pointing to the last commit (next parent), it's pointing
to a branch first and that's pointing to the commit:

```
cat .git/HEAD
cat .git/refs/heads/master
```

That's our commit hash. See for yourself in the log!

> A branch is nothing more than a reference under `.git/refs/heads`. You can create
branches by creating files there, or using the update-ref  command.

Whenever we moved HEAD (reset, remember?), we were in fact moving the branch!
Let's create another commit and see:

```
echo elephant >> animals
git add animals
git commit -m 'adding elephant'
gl
```

Two commits, all right! Let's move!

```
git reset --hard HEAD~
gl
git reflog
git reset --hard HEAD@{0}
gl
```

We're moving the master branch with HEAD. By the way this `@` syntax is pretty powerful:

```
git rev-parse HEAD@{5.minutes.ago}
```

For more examples and explanation see the "specifying revisions" man page:

```
git help revisions
```

## Creating and switching between branches

Let's say we want to start experimenting with something new without endangering
our maser branch. We're going to create a new branch called experiment:

```
# git update-ref refs/heads/experiment HEAD
git branch experiment
```

Where is it pointing to?

```
gl
```

It's pointing to HEAD, since we didn't specify otherwise.

We can't switch to a new branch with `reset` since that moved the branch too. To
move HEAD only, use the `checkout` command.

```
# git symbolic-ref HEAD refs/heads/experiment
git checkout experiment
```

There's also a shorthand to create a new branch and immediately switch to it:

```
# git checkout -b experiment
```

Now create a new commit:

```
echo phoenix >> animals
git commit -am 'adding phoenix'
gl
```

The `experiment` branch moved, while `master` stayed where it was.

> When's the last point when we could have created/switched to the new branch? (before committing)

We can easily switch back to master with:

```
git checkout master
git status
```

Note that it did not only move HEAD, but also updated files in the index and
working directory!

Now let's move master forward with another commit:

```
echo dog >> animals
git commit -am 'adding dog'
gl
```

We can't see the `experiment` branch in the history, since it's not an ancestor
of master. We can ask for the history of both branches with:

```
gl HEAD experiment
# or
# gl master experiment
```

This is quite misleading though. It looks as if experiment is be the parent
of master. With `--graph` we can print the edges between commits. `--oneline`
can help too:

```
git log --graph --oneline HEAD experiment
```

Let's update our alias with this:

```
alias gl="git log --deco --graph --oneline"
```

## Mering branches

Let's say we need to do an urgent hotfix on top of master. We're going to
create a new branch where master is and commit our fix there.

```
git checkout -b hotfix master
sed -i 's/dog/cat/' animals
git commit -am 'dog -> cat'
gl
```

Now that we're done, we need to merge out branch back into master. Git almost
always changes only the branch that we have checked out, so we need to get
master first.

```
git checkout master
git merge hotfix
```

Note "Fast-forward" in the output. It means that `hotfix` was a successor of `master`
so git was able to simply move `master` to where `hotfix` is pointing.

```
gl
```

Now let's do a little more complicated merge. Let's say we've completed our
experiment and concluded that `phoenix` is in fact an animal. Let's merge out
`experiment` branch into master.

```
git checkout master
git merge experiment
```

Wow, that failed. ): Why is that? Since we've changed the "same part" of the
same file on both branches, Git was not able to tell which version to keep.
Now it's asking for our help to find out whether phoenix goes after dog or cat?
Or maybe before them?

As always `git status` can help us find out what options we have:

```
git status
```

We need to fix our file first. There are fancier tools for resolving merge
conflicts but for this time any text editor should do the trick:

```
vim animals
```

Those lines with the funky characters are the "conflict markers". The lines
between `<<<<<<< HEAD` and `=======` are the ones coming from the file on `HEAD`
while the lines between `=======` and `>>>>>>> experiment` come from the
`experiment` branch.

Resolving the conflict means removing the conflict markers and otherwise editing
the file to our needs on the merger branch. For this time, simply removing the
conflict markers will do.

Once we've resolved conflicts in a file, we should mark it as resolved by adding
it to the index the usual way:

```
git add animals
```

Then conclude the merge by committing:

```
git commit
# or git commit --no-edit
```

Now take a look at the log:

```
gl
```

This is our first commit with multiple parents! How can we refer to those?
`HEAD^` is parent no. 1 and `HEAD^2` is parent no. 2.

> Question: What does HEAD^^ mean? (This is why we've learned `~` first.)
> Question #2: And what is HEAD~ when there are multiple parents?

## Rebasing

Merging is not the only way of integrating changes from one branch into another.
Rebasing is the act of reapplying commits from one branch onto another.

Let's roll back `master` to where we were before merging and rebase changes from
the `experiment` branch to `master`.

```
git checkout master
git reset --hard HEAD^
git checkout experiment
git rebase master
```

Rebase conflicts as before and then conclude the rebase process with `git rebase
--continue`.

```
vim animals
git add animals
git rebase --continue
```

> Question: why is it called --continue and not --finish?

Now take a look at the log and reflog!

```
gl
git reflog
```

Rebase was in fact a three step process:
 - moving to `master`
 - reapplying the commits (only one) from the `experiment` branch
 - moving the `experiment` branch

The middle step of reapplying commits can be done with the `cherry-pick` command,
but we're not going to go into details of that now (see the manual!).

## Merging vs. rebasing

There's no ultimate answer to which to use; it depends.

Rebasing leaves behind a clean and easy to understand line of commits, however
their origin (the fact that they came from an other branch) is lost in the process.

Rebase also creates new commit objects instead of reusing the original commits.
It means that every reference (e.g. tag) pointing to the original commits
will not be part of the main line.

The preferred way of integrating changes from one branch to another (and
especially to `master`) should be clearly defined in the projects workflow.

## Questions

1) What's the difference?

```
git reset --hard other-branch
git checkout other-branch
```

2) What's the difference?

```
HEAD^
HEAD~
```

3) True or false?

- A) rebase(a, b) conflicts => merge(a, b) conflicts
- B) rebase(a, b) conflicts => rebase(b, a) conflicts
- C) merge(a, b) conflicts => merge(b, a) conflicts
- D) merge(a, b) conflicts => rebase(a, b) conflicts
- E) rebase(a, b) don't conflict => merge(a, b) don't conflict
