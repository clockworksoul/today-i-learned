# Lastlog

The `lastlog` utility formats and prints the contents of the last login log file, `/var/log/lastlog` (which is a usually a very sparse file), including the login name, port, and last login date and time.

It is similar in functionality to the BSD program `last`, also included in linux distributions; however, `last` parses a different binary database file (`/var/log/wtmp` and `/var/log/btmp`).

## Usage

Lastlog prints its output in column format with login-name, port, and last-login-time of each and every user on the system mentioned in that order. The users are sorted by default according to the order in `/etc/passwd` However, it can also be used to modify the records kept in `/var/log/lastlog`.

```bash
$ lastlog
Username         Port     From             Latest
root                                       **Never logged in**
user             tty3                      Sun Jan 14 16:29:24 +0130 2019
```