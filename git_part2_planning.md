# Git part 2 - branching and remotes

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
git branch experiment
```

Where is it pointing to?

```
gl
```

It's pointing to HEAD, since we didn't specify otherwise (as usual).

We can't switch to a new branch with `reset` since that moved the branch too. To
move HEAD only, use the `checkout` command.

```
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
```




## Questions

1) What's the difference?

```
git reset --hard other-branch
git checkout other-branch
```
