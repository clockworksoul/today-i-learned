# `socat`

_Sources: [`socat`](https://medium.com/@copyconstruct/socat-29453e9fc8a6) by Cindy Sridharan; also [[1]](https://blog.travismclarke.com/post/socat-tutorial/), [[2]](https://linux.die.net/man/1/socat)_

`socat` (**SO**cket **CAT**) is a network utility similar to `netcat` that allows data transfer between two addresses by establishing two bidirectional byte streams and transferring data between them.

An address, in this context, can include:

* A network socket
* Any file descriptor
* Unix domain datagram or stream socket
* TCP and UDP (over both IPv4 and IPv6)
* SOCKS 4/4a over IPv4/IPv6, SCTP, PTY, datagram and stream sockets
* Named and unnamed pipes
* Raw IP sockets
* OpenSSL
* Any arbitrary network device (on Linux)

## Usage

The simplest `socat` invocation would be:

```bash
socat [options] <address> <address>
```

A more concrete example would be:

```bash
socat -d -d - TCP4:www.example.com:80
```

where `-d -d` would be the command options, `-` would be the first address and `TCP4:www.example.com:80` would be the second address.

![A typical socat invocation](https://miro.medium.com/max/1092/1*tH0A5UWSVZYjA7TFbjKxaQ.png)

This might seem like a lot to take in (and the examples in [the man page](http://www.dest-unreach.org/socat/doc/socat.html#ADDRESS_TCP_LISTEN) are, if anything, even more inscrutable), so let’s break each component down a bit more.

Let’s first start with the address, since the address is the cornerstone aspect of `socat`.

## Addresses

In order to understand `socat` it’s important to understand what addresses are and how they work.

The address is something that the user provides via the command line. Invoking `socat` without any addresses results in something like:

```bash
~: socat
2020/05/25 12:38:20 socat[36392] E exactly 2 addresses required (there are 0); use option "-h" for help
```

An address comprises of three components:

* The address _type_, followed by a `:`
* Zero or more required address _parameters_ separated by `:`
* Zero or more address _options_ separated by `,`

![The anatomy of an address](https://miro.medium.com/max/1400/1*i4QHuX3Cp-AGPSMZ0orBMA.png)

## Type

The _type_ is used to specify the kind of address we need.

Popular options are `TCP4`, `CREATE`, `EXEC`, `GOPEN`, `STDIN`, `STDOUT`, `PIPE`, `PTY`, `UDP4` etc, where the names are pretty self-explanatory.

However, in the example we saw in the previous section, a `socat` command was represented as

```bash
socat -d -d - TCP4:www.example.com:80
```

where `-` was said to be one of the two addresses. This doesn’t look like a fully formed address that adheres to the aforementioned convention.

This is because certain address types have aliases. `-` is one such alias used to represent `STDIO`. Another alias is `TCP` which stands for TCPv4. The manpage of `socat` lists all other aliases.

## Parameters

Immediately after the type comes zero or more required address parameters separated by `:`.

The number of address _parameters_ depends on the address _type_.

The address _type_ `TCP4` requires a _server specification_ and a _port specification_ (number or service name). A valid address with _type_ `TCP4` established with port `80` of host `www.example.com` would be `TCP:www.example.com:80`.

![The address type TCP4 requires a server specification and a port specification](https://miro.medium.com/max/1092/1*2uqxvYSneFCZeHvsHUId0Q.png)

Another example of an address would be `UDP_RECVFROM:9125` which creates a UDP socket on port 9125, receives one packet from an unspecified peer and may send one or more answer packets to that peer.

The address _type_ (like `TCP` or `UDP_RECVFROM`) is sometimes optional. Address specifications starting with a number are assumed to be of type `FD` (raw file descriptor) addresses. Similarly, if a `/` is found before the first `:` or `,` then the address type is assumed to be `GOPEN` (generic file open).

![The address type is sometimes optional](https://miro.medium.com/max/1092/1*XVvp04sgHDPNBeFFzoxMlQ.png)

## Address Options

Address parameters can be further enhanced with _options_, which govern how the opening of the address is done or what the properties of the resulting byte streams will be.

Options are specified after address parameters and they are separated from the last address parameter by a `,` (the `,` indicates when the address parameters end and when the options begin).

Options can be specified either directly or with an `option_name=value` pair.

Extending the previous example, we can specify the option `retry=5` on the address to specify the number of times the connection to `www.example.com` will be retried.

```
TCP:www.example.com:80,retry=5
```

Similarly, the following address allows one to set up a TCP listening socket and fork a child process to handle all incoming client connections.

```
TCP4-LISTEN:www.example.com:80,fork
```

Each _option_ belongs to one _option group_. Every address type has a set of option groups, and only options belonging to the option groups for the given address type are permitted. It is not, for example, possible to apply options reserved for sockets to regular files.

~[Address options have a many-to-1 mapping with option groups.](https://miro.medium.com/max/1400/1*c903Tb2lJCiNlhuJNhGxtA.png)

For example, the `creat` option belongs to the `OPEN` option group. The `creat` option can only be used with those address types (`GOPEN`, `OPEN` and `PIPE`) that have `OPEN` as a part of their option group set.

The `OPEN` option group allows for the setting of flags with the `open()` system call. Using `creat` as an option on an address of type `OPEN` sets the `O_CREAT` flag when `open()` is invoked.

## Unidirectional and Bidirectional Streams

Now that we have a slightly better understanding of what addresses are, let’s see how data is transferred between the two addresses.

For the first stream, the first address acts as the data source and the second address is the data sink. For the second byte stream, the second address is the data source and the first address is the data sink.

Invoking `socat` with `-u` ensures that the first address can only be used for _reading_ and the second address can only be used for _writing_.

In the following example,

```bash
socat -u STDIN STDOUT
```

the first address (`STDIN`) is only used for _reading_ data and the second address (`STDOUT`) is only used for _writing_ data. This can be verified by looking at what `socat` prints out when it is invoked:

```bash
$ socat -u STDIN STDOUT
2018/10/14 14:18:15 socat[22736] N using stdin for reading
2018/10/14 14:18:15 socat[22736] N using stdout for writing
...
```

![Invoking socat with -u establishes two unidirectional byte streams](https://miro.medium.com/max/1400/1*b9TEYP5Pre2lC_R6FkBb6Q.png)

Whereas invoking `socat STDIN STDOUT` opens both the addresses for reading and writing.

```bash
$ socat STDIN STDOUT
2018/10/14 14:19:48 socat[22739] N using stdin for reading and writing
2018/10/14 14:19:48 socat[22739] N using stdout for reading and writing
```

![By default socat establishes two bidirectional byte streams](https://miro.medium.com/max/1400/1*IzP1AE9ejvNh4FE1PnSc2Q.png)

## Single Address Specification and Dual Addresses

An address of the sort we’ve seen until now that conforms to the aforementioned format is known as a `single address specification`:

```bash
socat -d -d - TCP4:www.example.com:80
```

Constructing a `socat` command with _**two single address specifications**_ ends up establishing _two unidirectional byte streams_ between the two addresses.

![Two Single Address Specifications](https://miro.medium.com/max/1092/1*E0YsnW2Xf01OMrFkSb8w0g.png)

<!-- ![Foo](https://miro.medium.com/max/1400/1*zZTTjRPx-a0mxHB_c4OoeQ.png) -->

In this case, the first address can write data to the second address and the second address would write data back to the first address. However, it’s possible that we might not want the second address to write data back to the first address (`STDIO`, in this case), but instead we might want it to write the data to a different address (a file, for example) instead.

Two _single addresses specifications_ can be combined with `!!` to form a _dual type address_ for one byte stream. Here, the first address is used by `socat` for reading data, and the second address for writing data.

![Foo](https://miro.medium.com/max/1092/1*QQBLra9ro4OSobfXtz9AWg.png)

```bash
socat -d -d   READLINE\!\!OPEN:file.txt,creat,trunc   SYSTEM:'read stdin; echo $stdin'
```

In this example, the first address is a _dual address specification_ (`READLINE!!OPEN:file.txt,creat,trunc`), where data is being read from a source (`READLINE`) and written to a different sink (`OPEN:file.txt,creat,trunc`). 

Here, `socat` reads the input from `READLINE`, transfers it to the second address (`SYSTEM` — which forks a child process, executes the shell command specified).

The second address returns data back to a different sink of the first address — in this example it’s `OPEN:file.txt,creat,trunc`.

![Now I'm starting to get confused](https://miro.medium.com/max/1400/1*aHB60Om_hpC037HRelExpg.png)

This was a fairly trivial example where only one of the two addresses was a dual address specification. It’s possible that both of the addresses might be dual address specifications.

## Address Options Versus `socat` Options

It’s important to understand that the options that apply to an address, which shaping how the address behaves, are different from invoking `socat` itself with specific options, which govern how the `socat` tool itself behaves.

We’ve already seen previously that adding the option `-u` to `socat` ensures that the first address is only opened for reading and the second address for writing.

Another useful option to know about is the `-d` option, which can be used with `socat` to print warning messages, in addition to fatal and error messages.

Invoking socat with `-d -d` will print the fatal, error, warning, _and_ notice messages. I usually have `socat` aliased to `socat -d -d` in my dotfiles.

Similarly, if one invokes `socat -h` or `socat -hh`, you will be presented with a wall of information that can be a little overwhelming and not all of which is required for getting started.

## Cheatsheet

_Source: [Socat Cheatsheet](https://blog.travismclarke.com/post/socat-tutorial/) by Travis Clarke_

There are also a _ton_ of examples on the [`socat` man page](http://www.dest-unreach.org/socat/doc/socat.html#EXAMPLES).

### Server/Client communication

#### Create an echo server

##### Server

```bash
socat -v tcp-l:8080,fork exec:"/bin/cat"
```

##### Client

```bash
socat TCP:localhost:8080 -
```

#### Create a simple client/server TCP connection

##### Server

```bash
socat TCP-LISTEN:8800,reuseaddr,pf=ip4,fork -
```

##### Client

```bash
socat TCP:localhost:8800 -
```

#### Create a client/server TCP connection over TLS/SSL

##### Server

```bash
socat OPENSSL-LISTEN:4443,reuseaddr,pf=ip4,fork,cert=$HOME/tmp/server.pem,cafile=$HOME/tmp/client.pem -
```

##### Client

```bash
socat OPENSSL:localhost:4443,verify=0,cert=$HOME/tmp/client.pem,cafile=$HOME/tmp/server.pem -
```

### UNIX domain socket communication

#### Create a UNIX domain socket for IPC (inter-process communication) with a **single (bi-directional)** endpoint

##### Server

```bash
socat UNIX-LISTEN:/usr/local/var/run/test/test.sock -
```

##### Client

```bash
socat UNIX-CONNECT:/usr/local/var/run/test/test.sock -
```

#### Create a UNIX domain socket for IPC (inter-process communication) with **multiple (bi-directional)** endpoints

##### Server

```bash
socat UNIX-LISTEN:/usr/local/var/run/test/test.sock,fork -
```

##### Client

```bash
socat UNIX-CONNECT:/usr/local/var/run/test/test.sock -
```

#### Create UNIX domain sockets for IPC (inter-process communication) and save the communication output to file

##### Server

```bash
socat UNIX-LISTEN:/usr/local/var/run/test/test.sock,fork - >> $HOME/tmp/socketoutput.txt
```

##### Client

```bash
socat UNIX-CONNECT:/usr/local/var/run/test/test.sock -
```

### Command execution

#### Execute shell commands on a remote server (i.e. basic ssh client)

##### Server

```bash
socat TCP-LISTEN:1234 EXEC:/bin/bash
```

##### Client

```bash
socat TCP:localhost:1234 -
```

### Tunneling

#### Create an encrypted tunnel between a local computer and a remote machine to relay services created through an SSH protocol connection

##### Server

```bash
socat TCP-LISTEN:54321,reuseaddr,fork TCP:remote.server.com:22
```

##### Client

```bash
ssh root@localhost -p 54321
```

#### Create a virtual point-to-point IP link through a TUN network device

Mac OS X: requires the [TunTap](http://tuntaposx.sourceforge.net/) kernel extensions.

##### Server

```bash
socat /dev/tun0 -
```

##### Client

```bash
ifconfig tun0 10.0.0.2 10.0.0.3
ping 10.0.0.3
```
