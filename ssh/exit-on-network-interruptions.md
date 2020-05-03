# Exit SSH Automatically on Network Interruptions

_Source: https://smallstep.com/blog/ssh-tricks-and-tips/#exit-stuck-sessions_

SSH sessions can often hang due to network interruptions, a program that gets out of control, or one of those terminal escape sequences that lock keyboard input.

To configure your SSH to automatically exit stuck sessions, add the following to your `.ssh/config`:

```
ServerAliveInterval 5
ServerAliveCountMax 1
```

`ssh` will check the connection by sending an echo to the remote host every `ServerAliveInterval` seconds. If more than `ServerAliveCountMax` echoes are sent without a response, `ssh` will timeout and exit.
