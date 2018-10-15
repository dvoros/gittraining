# Git part 3 - remotes

- create new local repo and quickly go thru previous material
- clone that repo
- create tmp branch in original
- fetch in clone

```
git checkout -b mytmp
git reset --hard origin/tmp
echo phoenix >> animals
git commit -am 'adding phoenix'
git push origin HEAD:tmp
```

It would be easier if wouldn't have to always specify tmp:

```
git branch --set-upstream-to origin/tmp
```

We could have done this when checking out the branch:

```
git checkout -b tmp --track origin/tmp
git branch -D tmp                     # did this delete the remote branch??
git checkout tmp
```

- how to delete remote branch

```
git push origin :tmp 
```
