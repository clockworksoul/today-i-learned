# Non-privileged containers based on the scratch image

_Source: [Non-privileged containers based on the scratch image](https://medium.com/@lizrice/non-privileged-containers-based-on-the-scratch-image-a80105d6d341)_

## Multi-stage builds give us the power

Here’s a Dockerfile that gives us what we’re looking for.

```dockerfile
FROM ubuntu:latest
RUN useradd -u 10001 scratchuser

FROM scratch
COPY dosomething /dosomething
COPY --from=0 /etc/passwd /etc/passwd
USER scratchuser

ENTRYPOINT ["/dosomething"]
```

Building from this Dockerfile starts FROM an Ubuntu base image, and creates a new user called _scratchuser_. See that second `FROM` command? That’s the start of the next stage in the multi-stage build, this time working from the scratch base image (which contains nothing).

Into that empty file system we copy a binary called `dosomething` — it’s a trivial Go app that simply sleeps for a while.

```go
package main

import "time"

func main() {
    time.Sleep(500 * time.Second)
}
```

We also copy over the `/etc/passwd` file from the first stage of the build into the new image. This has `scratchuser` in it.

This is all we need to be able to transition to `scratchuser` before starting the dosomething executable.
