# A generalized single-instance bash script

_Source: [Przemyslaw Pawelczyk](https://gist.github.com/przemoc/571091)_

An excellent generalized script that ensures that only one instance of the script can run at a time.

It does the following:

* Sets an [exit trap](http://redsymbol.net/articles/bash-exit-traps/) so the lock is always released (via `_prepare_locking`)
* Obtains an [exclusive file lock](https://en.wikipedia.org/wiki/File_locking) on `$LOCKFILE` or immediately fail (via `exlock_now`)
* Executes any additional custom logic

```bash
#!/usr/bin/env bash

set -euo pipefail

# This script will do the following:
#
#  - Set an exit trap so the lock is always released (via _prepare_locking)
#  - Obtain an exclusive lock on $LOCKFILE or immediately fail (via exlock_now)
#  - Execute any additional logic
#
# If this script is run while another has a lock it will exit with status 1.

### HEADER ###
readonly LOCKFILE="/tmp/$(basename $0).lock"  # source has "/var/lock/`basename $0`"
readonly LOCKFD=99

# PRIVATE
_lock()             { flock -$1 $LOCKFD; }
_no_more_locking()  { _lock u; _lock xn && rm -f $LOCKFILE; }
_prepare_locking()  { eval "exec $LOCKFD>\"$LOCKFILE\""; trap _no_more_locking EXIT; }

# ON START
_prepare_locking
 
# PUBLIC
exlock_now()        { _lock xn; }  # obtain an exclusive lock immediately or fail
exlock()            { _lock x; }   # obtain an exclusive lock
shlock()            { _lock s; }   # obtain a shared lock
unlock()            { _lock u; }   # drop a lock

### BEGIN OF SCRIPT ###
 
# Simplest example is avoiding running multiple instances of script.
exlock_now || exit 1
 
# All script logic goes below.
#
# Remember! Lock file is removed when one of the scripts exits and it is
#           the only script holding the lock or lock is not acquired at all.
```