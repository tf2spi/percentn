---
title: 'Setting Breakpoints on Yourself!'
description: 'How to Set Breakpoints on Yourself for anti-debugging or self-debugging'
pubDate: 2023-11-16
tags: ["debug", "hooks", "linux", "windows" ]
---

# Setting Breakpoints on Yourself!

Setting breakpoints on yourself could be useful because it's
an easy way to dynamically notify a debugger when something
happens. Perhaps it's a condition that shouldn't happen and
you want to break there so that a debugger can catch it.

## Fantastic Breakpoints and How to Set Them.

For Windows, there's ``DebugBreak`` or ``__debugbreak__``.
For Linux, you'd use a compiler intrinsic or inline assembly
via ``gcc``, ``clang``, or whatever other compiler you like.
``__builtin_break`` is specifically defined fro gcc.

If you want to insert a breakpoint when it wasn't set before,
you can write your own memory or even another process's
memory if you have permission.

On Windows, ``WriteProcessMemory`` can be used on yourself.
On Linux, ``/proc/self/mem`` can also be used.

Note that I mention these specifically because they bypass
page protections so even a page that is readonly or even
read/execute can be written to with this, even with W^X
policies like DEP.

After that, write the instruction that does the breakpoint.

```c
// Writing x86 breakpoint on Windows
WriteProcessMemory(GetCurrentProcess(), &fn, "\xcc", 1, NULL);

// Writing x86 breakpoint on Linux
int fd = open("/proc/self/mem", O_RDWR);
lseek64(fd, (off64_t)&fn, SEEK_SET);
write(fd, "\xcc", 1);
```

## Hardware Breakpoint Setting

Hardware breakpoints are especially tricky to set on yourself.

First off, not all threads have the same hardware breakpoints.
This is actually a cool feature because you can break or watch
the data access on a single thread, whereas software breakpoints
are forced to break for all threads. However, it makes it harder
to broadcast this for all threads.

Note that threads do not inherit hardware breakpoints so you
often have to reserve a hardware breakpoint for thread creation
if you really want a true global hardware breakpoint.

For Windows, ``GetThreadContext`` and ``SetThreadContext``
allow you to set hardware breakpoints, even on ARM!

```c
CONTEXT ctx;
HANDLE hThread = GetCurrentThread();
ctx.ContextFlags = CONTEXT_DEBUG_REGISTERS;
GetThreadContext(hThread, &ctx);
// ...
ctx.Dr0 = NewDr0;
ctx.Dr1 = NewDr1;
// ...
ctx.ContextFlags = CONTEXT_DEBUG_REGISTERS;
SetThreadContext(hThread, &ctx);
```

For Linux, it's a massive pain and very architecture-specific,
but what ends up ultimately happening is that you use ``ptrace``.
However, ``ptrace`` can't be used by a thread in the same
process group, so you have to create a new process.

This is very inconvenient but you can use the ``clone``
syscall with ``CLONE_VM`` flag without the ``CLONE_THREAD``
flag to create a new process which shares the same address space
(We love how cursed Linux is...).

In addition, distros have been using ``yama`` recently,
so you need to use ``prctl(PR_SET_PTRACER, pid)`` on
the child you create in order to allow it to debug you. 


```c
int debugger(void *arg)
{
    int pid = (long)arg;
    // You should error check ptrace calls.
    ptrace(PTRACE_ATTACH, mypid);
    // POKEUSER is specific to x86 Linux
    // ARM, for instance, uses PTRACE_SETHBPREGS
    // ...
    ptrace(PTRACE_POKEUSER, mypid, offsetof(struct user, u_debugreg) + 0, NewDr0);
    ptrace(PTRACE_POKEUSER, mypid, offsetof(struct user, u_debugreg) + sizeof(long), NewDr1);
    // ...
    ptrace(PTRACE_DETACH, mypid);
    return 0;
}

void debugee()
{
    int mypid = getpid();
    size_t stacksize = 1 << 16;
    uint8_t *stack = malloc(stacksize);
    int child = clone(debugger, stack + stacksize, (void *)(long)mypid, CLONE_VM | SIGCHLD);
    if (child != -1) {
        // RACE CONDITION!
        // prctl could execute after 'debugger' thread is done.
        // You should synchronize with 'debugger'
        prctl(PR_SET_PTRACER, child);

        // You should error check waitpid
        waitpid(child, NULL, 0);
    }
}
```

## How to Hook your Own Breakpoints

This is already established in [Bag of Tricks 1](./bag-of-tricks).

That being said, you still have to be careful about
debugging details on each architecture, since the debugger won't help you there.
