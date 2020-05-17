# Safer Bash Scripts With `set -euxo pipefail`

_Source: [[1]](https://vaneyckt.io/posts/safer_bash_scripts_with_set_euxo_pipefail/)_

The bash shell comes with several builtin commands for modifying the behavior of the shell itself. We are particularly interested in the `set` builtin, as this command has several options that will help us write safer scripts. I hope to convince you that it’s a really good idea to add `set -euxo pipefail` to the beginning of all your future bash scripts.

## `set -e`

The `-e` option will cause a bash script to exit immediately when a command fails.

This is generally a vast improvement upon the default behavior where the script just ignores the failing command and continues with the next line. This option is also smart enough to not react on failing commands that are part of conditional statements.

You can append a command with `|| true` for those rare cases where you don’t want a failing command to trigger an immediate exit.

### Before

```bash
#!/bin/bash

# 'foo' is a non-existing command
foo
echo "bar"

# output
# ------
# line 4: foo: command not found
# bar
#
# Note how the script didn't exit when the foo command could not be found.
# Instead it continued on and echoed 'bar'.
```

### After

```bash
#!/bin/bash
set -e

# 'foo' is a non-existing command
foo
echo "bar"

# output
# ------
# line 5: foo: command not found
#
# This time around the script exited immediately when the foo command wasn't found.
# Such behavior is much more in line with that of higher-level languages.
```

### Any command returning a non-zero exit code will cause an immediate exit

```bash
#!/bin/bash
set -e

# 'ls' is an existing command, but giving it a nonsensical param will cause
# it to exit with exit code 1
$(ls foobar)
echo "bar"

# output
# ------
# ls: foobar: No such file or directory
#
# I'm putting this in here to illustrate that it's not just non-existing commands
# that will cause an immediate exit.
```

### Preventing an immediate exit

```bash
#!/bin/bash
set -e

foo || true
$(ls foobar) || true
echo "bar"

# output
# ------
# line 4: foo: command not found
# ls: foobar: No such file or directory
# bar
#
# Sometimes we want to ensure that, even when 'set -e' is used, the failure of
# a particular command does not cause an immediate exit. We can use '|| true' for this.
```

### Failing commands in a conditional statement will not cause an immediate exit

```bash
#!/bin/bash
set -e

# we make 'ls' exit with exit code 1 by giving it a nonsensical param
if ls foobar; then
  echo "foo"
else
  echo "bar"
fi

# output
# ------
# ls: foobar: No such file or directory
# bar
#
# Note that 'ls foobar' did not cause an immediate exit despite exiting with
# exit code 1. This is because the command was evaluated as part of a
# conditional statement.
```

That’s all for `set -e`. However, `set -e` by itself is far from enough. We can further improve upon the behavior created by `set -e` by combining it with `set -o pipefail`. Let’s have a look at that next.

## `set -o pipefail`

The bash shell normally only looks at the exit code of the last command of a pipeline. This behavior is not ideal as it causes the `-e` option to only be able to act on the exit code of a pipeline’s last command. This is where `-o pipefail` comes in. This particular option sets the exit code of a pipeline to that of the rightmost command to exit with a non-zero status, or to zero if all commands of the pipeline exit successfully.

### Before

```bash
#!/bin/bash
set -e

# 'foo' is a non-existing command
foo | echo "a"
echo "bar"

# output
# ------
# a
# line 5: foo: command not found
# bar
#
# Note how the non-existing foo command does not cause an immediate exit, as
# it's non-zero exit code is ignored by piping it with '| echo "a"'.
```

### After

```bash
#!/bin/bash
set -eo pipefail

# 'foo' is a non-existing command
foo | echo "a"
echo "bar"

# output
# ------
# a
# line 5: foo: command not found
#
# This time around the non-existing foo command causes an immediate exit, as
# '-o pipefail' will prevent piping from causing non-zero exit codes to be ignored.
```

This section hopefully made it clear that `-o pipefail` provides an important improvement upon just using `-e` by itself. However, as we shall see in the next section, we can still do more to make our scripts behave like higher-level languages.

## `set -u`

This option causes the bash shell to treat unset variables as an error and exit immediately. Unset variables are a common cause of bugs in shell scripts, so having unset variables cause an immediate exit is often highly desirable behavior.

### Before

```bash
#!/bin/bash
set -eo pipefail

echo $a
echo "bar"

# output
# ------
#
# bar
#
# The default behavior will not cause unset variables to trigger an immediate exit.
# In this particular example, echoing the non-existing $a variable will just cause
# an empty line to be printed.
```

### After

```bash
#!/bin/bash
set -euo pipefail

echo "$a"
echo "bar"

# output
# ------
# line 5: a: unbound variable
#
# Notice how 'bar' no longer gets printed. We can clearly see that '-u' did indeed
# cause an immediate exit upon encountering an unset variable.
```

### Dealing with `${a:-b}` variable assignments

Sometimes you’ll want to use a [`${a:-b}` variable assignment](https://unix.stackexchange.com/questions/122845/using-a-b-for-variable-assignment-in-scripts/122878) to ensure a variable is assigned a default value of b when a is either empty or undefined. The `-u` option is smart enough to not cause an immediate exit in such a scenario.

```bash
#!/bin/bash
set -euo pipefail

DEFAULT=5
RESULT=${VAR:-$DEFAULT}
echo "$RESULT"

# output
# ------
# 5
#
# Even though VAR was not defined, the '-u' option realizes there's no need to cause
# an immediate exit in this scenario as a default value has been provided.
```

### Using conditional statements that check if variables are set

Sometimes you want your script to not immediately exit when an unset variable is encountered. A common example is checking for a variable’s existence inside an `if` statement.

```bash
#!/bin/bash
set -euo pipefail

if [ -z "${MY_VAR:-}" ]; then
  echo "MY_VAR was not set"
fi

# output
# ------
# MY_VAR was not set
#
# In this scenario we don't want our program to exit when the unset MY_VAR variable
# is evaluated. We can prevent such an exit by using the same syntax as we did in the
# previous example, but this time around we specify no default value.
```

This section has brought us a lot closer to making our bash shell behave like higher-level languages. While `-euo pipefail` is great for the early detection of all kinds of problems, sometimes it won’t be enough. This is why in the next section we’ll look at an option that will help us figure out those really tricky bugs that you encounter every once in a while.

## `set -x`

The `-x` option causes bash to print each command before executing it.

This can be a great help when trying to debug a bash script failure. Note that arguments get expanded before a command gets printed, which will cause our logs to contain the actual argument values that were present at the time of execution!

```bash
#!/bin/bash
set -euxo pipefail

a=5
echo $a
echo "bar"

# output
# ------
# + a=5
# + echo 5
# 5
# + echo bar
# bar
```

That’s it for the `-x` option. It’s pretty straightforward, but can be a great help for debugging. Next up, we’ll look at an option I had never heard of before that was suggested by a reader of this blog.

## Reader suggestion: `set -E`

Traps are pieces of code that fire when a bash script catches certain signals. Aside from the usual signals (e.g. `SIGINT`, `SIGTERM`, ...), traps can also be used to catch special bash signals like `EXIT`, `DEBUG`, `RETURN`, and `ERR`. However, reader Kevin Gibbs pointed out that using `-e` without `-E` will cause an `ERR` trap to not fire in certain scenarios.

### Before

```bash
#!/bin/bash
set -euo pipefail

trap "echo ERR trap fired!" ERR

myfunc()
{
  # 'foo' is a non-existing command
  foo
}

myfunc
echo "bar"

# output
# ------
# line 9: foo: command not found
#
# Notice that while '-e' did indeed cause an immediate exit upon trying to execute
# the non-existing foo command, it did not case the ERR trap to be fired.
```

### After

```bash
#!/bin/bash
set -Eeuo pipefail

trap "echo ERR trap fired!" ERR

myfunc()
{
  # 'foo' is a non-existing command
  foo
}

myfunc
echo "bar"

# output
# ------
# line 9: foo: command not found
# ERR trap fired!
#
# Not only do we still have an immediate exit, we can also clearly see that the
# ERR trap was actually fired now.
```

The documentation states that `-E` needs to be set if we want the `ERR` trap to be inherited by shell functions, command substitutions, and commands that are executed in a subshell environment. The `ERR` trap is normally not inherited in such cases.

## Conclusion

I hope this post showed you why using set `-euxo pipefail` (or `set -Eeuxo pipefail`) is such a good idea. If you have any other options you want to suggest, then please let me know and I’ll be happy to add them to this list.
