# Using core.excludesFile to define a global git ignore

_Source: The [`gitignore` documentation](https://git-scm.com/docs/gitignore)_

The `core.excludesFile` git configuration variable can be used to define an arbitrary file location (`~/.gitignore` in this example) that contains patterns for git to always ignore.

Example usage:

```
$ git config --global core.excludesFile ~/.gitignore
```
