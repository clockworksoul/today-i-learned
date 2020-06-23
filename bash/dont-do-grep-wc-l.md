# Don't do `grep | wc -l`

_Source: [Useless Use of Cat Award](http://porkmail.org/era/unix/award.html)_

There is actually a whole class of "Useless Use of (something) | grep (something) | (something)" problems but this one usually manifests itself in scripts riddled by useless backticks and pretzel logic.

Anything that looks like:

```bash
something | grep '..*' | wc -l
```

can usually be rewritten like something along the lines of

```bash
something | grep -c .   # Notice that . is better than '..*'
```

or even (if all we want to do is check whether something produced any non-empty output lines)

```bash
something | grep . >/dev/null && ...
```

(or `grep -q` if your grep has that).

## Addendum

`grep -c` can actually solve a large class of problems that `grep | wc -l` can't.

If what interests you is the count for each of a group of files, then the only way to do it with `grep | wc -l` is to put a loop round it. So where I had this:

```bash
grep -c "^~h" [A-Z]*/hmm[39]/newMacros
```

the naive solution using `wc -l` would have been...

```bash
for f in [A-Z]*/hmm[39]/newMacros; do
    # or worse, for f in `ls [A-Z]*/hmm[39]/newMacros` ...
    echo -n "$f:"
    # so that we know which file's results we're looking at
    grep "^~h" "$f" | wc -l
    # gag me with a spoon
done
```

...and notice that we also had to fiddle to get the output in a convenient form.
