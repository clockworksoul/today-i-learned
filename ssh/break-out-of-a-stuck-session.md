# Break Out of a Stuck SSH Session

_Source: https://smallstep.com/blog/ssh-tricks-and-tips/#exit-stuck-sessions_

SSH sessions can often hang due to network interruptions, a program that gets out of control, or one of those terminal escape sequences that lock keyboard input.

`ssh` includes the escape character `~` by default. The command `~`. closes an open connection and brings you back to the terminal. (You can only enter escape sequences on a new line.) `~?` lists all of the commands you can use during a session. On international keyboards, you may need to press the `~` key twice to send the `~` character.
