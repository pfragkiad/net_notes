## Refresh cache after adding .gitignore

1. Add .gitignore to solution

```bat
dotnet new gitignore
```

2. Clear cache, add everything and commit.

```bash
git rm -r --cached .
git add .
git commit -m "Reset cache"
```

## Hard reset from remote branch

```bash
git fetch origin
git reset --hard origin/master
```
