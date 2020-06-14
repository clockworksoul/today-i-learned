# Ring the audio bell in bash

In many shells, the `\a` character will trigger the audio bell.

Example:

```bash
$ echo -n $'\a'
```

You can also use `tput`, since `/a` doesn't work in some shells:

```bash
$ tput bel
```
