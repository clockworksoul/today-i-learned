# Creating a Linux Service with systemd

_Sources: [[1]](https://medium.com/@benmorel/creating-a-linux-service-with-systemd-611b5c8b91d6), [[2]](https://www.freedesktop.org/software/systemd/man/systemd.unit.html)_

# The Program

For the purposes of this example, we'll start a toy echo server on port 80 using `socat`.

Let’s start it:

```bash
$ socat -v tcp-l:8080,fork exec:"/bin/cat"
```

And test it in another terminal:

```
$ nc 127.0.0.1 8080

Hello world!
Hello world!
```

Cool, it works. Now we want this script to run at all times, be restarted in case of a failure (unexpected exit), and even survive server restarts. That’s where systemd comes into play.

## Turning It Into a Service

Let’s create a file called `/etc/systemd/system/echo.service`:

```
[Unit]
Description=Echo service
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=ubuntu
ExecStart=/usr/bin/env socat -v tcp-l:8080,fork exec:"/bin/cat"

[Install]
WantedBy=multi-user.target
```

Of course you'll want to set your actual username after `User=`.

That’s it.

## Starting The Service

We can now start the service:

```bash
$ systemctl start echo
```

And automatically get it to start on boot:

```bash
$ systemctl enable echo
```

## Going further

Now that your service (hopefully) works, it may be important to dive a bit deeper into the configuration options, and ensure that it will always work as you expect it to.

### Starting in the right order

You may have wondered what the `After=` directive did. It simply means that your service must be started after the network is ready. For example, if your program expects MySQL to be up and running you could add:

```
After=mysqld.service
```

### Restarting on exit

By default, systemd does not restart your service if the program exits for whatever reason. This is usually not what you want for a service that must be always available, so we’re instructing it to always restart on exit:

```
Restart=always
```

You could also use `on-failure` to only restart if the exit status is not `0`.

By default, systemd attempts a restart after 100ms. You can specify the number of seconds to wait before attempting a restart, using:

```
RestartSec=1
```

### Avoiding the Start Limit Trap

By default, when you configure `Restart=always` as we did, systemd gives up restarting your service if it fails to start more than 5 times within a 10 seconds interval. Forever.

There are two `[Unit]` configuration options responsible for this:

```
StartLimitBurst=5
StartLimitIntervalSec=10
```

The `RestartSec` directive also has an impact on the outcome: if you set it to restart after 3 seconds, then you can never reach 5 failed retries within 10 seconds.

The simple fix that always works is to set `StartLimitIntervalSec=0`. This way, systemd will attempt to restart your service forever.
It’s a good idea to set `RestartSec` to at least 1 second though, to avoid putting too much stress on your server when things start going wrong.

As an alternative, you can leave the default settings, and ask systemd to restart your server if the start limit is reached, using `StartLimitAction=reboot`.

## Is that really it?

That’s all it takes to create a Linux service with systemd: writing a small configuration file that references your long-running program.

Systemd has been the default init system in RHEL/CentOS, Fedora, Ubuntu, Debian and others for several years now, so chances are that your server is ready to host your homebrew services.
