## Refresh cache after adding .gitignore

1. Add .gitignore to solution

```sh
dotnet new gitignore
```

2. Clear cache, add everything and commit.

```git
git rm -r --cached .
git add .
git commit -m "Reset cache"
```

