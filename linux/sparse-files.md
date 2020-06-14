# Sparse Files

A sparse file is a type of file that attempts to use file system space more efficiently when the file itself is partially empty.

This is done by writing metadata representing the empty blocks to disk instead of the actual "empty" space which makes up the block, using less disk space.

The full block size is written to disk as the actual size only when the block contains "real" (non-empty) data.

The `/var/log/lastlog` is a typical example of a very sparse file.

## Detection
The `-s` option of the `ls` command shows the occupied space in blocks.

```bash
$ ls -lhs
```

## More Reading

* [Stackoverflow: What is a sparse file and why do we need it?](https://stackoverflow.com/questions/43126760/what-is-a-sparse-file-and-why-do-we-need-it)