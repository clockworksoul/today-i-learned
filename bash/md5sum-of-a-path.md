# Bash One-Liner to Calculate the md5sum of a Path in Linux

To compute the md5sum of an entire path in Linux, we `tar` the directory in question and pipe the produce `md5sum`.

Note in this example that that we exclude the `.git` directory, which I've found can vary between runs even if there's no change to the repository contents.

```bash
$ tar --exclude='.git' -cf - "${SOME_DIR}" 2> /dev/null | md5sum
851ce98b3e97b42c1b01dd26e23e2efa  -
```
