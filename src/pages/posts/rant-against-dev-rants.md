---
title: 'Rant against Dev Rants'
description: 'Seriously\? This is what you complain about\?'
layout: ../../layouts/BlogPost.astro
---


# Rant against Dev Rants

Look, there's a lot of reasons to love your local developer and give them a hug.
They're creators first and foremost and, as many artists would also tell you,
being a creator is just plain hard and you need to be a continual learner.

In addition, their creations have to be adequately secure, performant,
maintainable, blah, blah, blah, blah, blah... All hard things in
addition to just making a cool creation in the first place.

It's astonishing that, given what they have to go through,
they complain about all the wrong things sometimes...

It's something that shouldn't annoy me nor should it be
any of my business to begin with since development isn't
even my main specialty. Here we are anyways... Have fun!

## Rant 1: This Code is Unreadable

This one particularly annoys me. So much. Ahhhhhhhhhhhhh!

I have to read garbage like this for my job
```c
  local_78[0] = &local_48;
  local_e8 = (longlong *)thunk_FUN_14000e450((undefined8 *)(param_1 + 0x10));
  puVar2 = thunk_FUN_14000e4b0(&local_f0,&local_68);
  uVar3 = thunk_FUN_14000e270(puVar2);
  local_b0 = *local_e8;
  iVar1 = (**(code **)(local_b0 + 0x18))(local_e8,&local_60,local_78,uVar3);
  FUN_14000cc30(iVar1);
  local_f4 = 8;
  local_f8 = 0;
  local_e0 = (longlong *)thunk_FUN_14000e470((undefined8 *)(param_1 + 0x18));
  uVar3 = thunk_FUN_14000e810((undefined8 *)(param_1 + 0x28));
  local_a8 = *local_e0;
  (**(code **)(local_a8 + 0x58))(local_e0,uVar3,0,0);
  local_d8 = (longlong *)thunk_FUN_14000e470((undefined8 *)(param_1 + 0x18));
  uVar3 = thunk_FUN_14000e7b0((undefined8 *)(param_1 + 0x30));
  local_a0 = *local_d8;
  (**(code **)(local_a0 + 0x48))(local_d8,uVar3,0,0);
  local_d0 = (longlong *)thunk_FUN_14000e470((undefined8 *)(param_1 + 0x18));
  uVar3 = thunk_FUN_14000e830(&local_f0);
  // You get the point...
```

But this is what really triggers most devs enough for them
to write their next stupid blog post about how programmers
keep getting worse and that they don't make them like they used to?
```c
void foo()
{
    void *bar = malloc(16);
    if (!bar)
        return;
    else
    {
        // Do some stuff...
    }
}
```

And don't lie to me and say they wouldn't be. This is a bad way of
writing it and I did this on purpose to illustrate that even this
stupid way of writing things isn't even as bad as a decompilation
on a good day.

Now, I know that decompilation isn't really the most esoteric
things when it comes to computers. There are esoteric languages
themselves like brainfuck as well as code golf challenges like
this wonderful donut below.

```c
             k;double sin()
         ,cos();main(){float A=
       0,B=0,i,j,z[1760];char b[
     1760];printf("\x1b[2J");for(;;
  ){memset(b,32,1760);memset(z,0,7040)
  ;for(j=0;6.28>j;j+=0.07)for(i=0;6.28
 >i;i+=0.02){float c=sin(i),d=cos(j),e=
 sin(A),f=sin(j),g=cos(A),h=d+2,D=1/(c*
 h*e+f*g+5),l=cos      (i),m=cos(B),n=s\
in(B),t=c*h*g-f*        e;int x=40+30*D*
(l*h*m-t*n),y=            12+15*D*(l*h*n
+t*m),o=x+80*y,          N=8*((f*e-c*d*g
 )*m-c*d*e-f*g-l        *d*n);if(22>y&&
 y>0&&x>0&&80>x&&D>z[o]){z[o]=D;;;b[o]=
 ".,-~:;=!*#$@"[N>0?N:0];}}/*#****!!-*/
  printf("\x1b[H");for(k=0;1761>k;k++)
   putchar(k%80?b[k]:10);A+=0.04;B+=
     0.02;}}/*****####*******!!=;:~
       ~::==!!!**********!!!==::-
         .,~~;;;========;;;:~-.
             ..,--------,*/
```

Even with the decompilation I provided, I really couldn't tell
you what that specific portion of the code does. I even wrote
the program being decompiled and I can't tell you what it does.

However, that's a much harder class of esoteric compared to the
ones devs complain about and y'all know it. Don't try to pull
a fast one on me.

## Rant 2: Everything's a Nail

Developers have a lot of general guidelines to writing code,
especially those with more seniority. This is a good thing
in programming as well as in other fields since it allows
experts to get less bogged down in details that would overwhelm
their less experienced peers.

However, if such an expert is too rigid and is not watching
for ways even well-established patterns can fail on them,
they develop tunnel vision and fail to see other perspectives
that could be helpful.

Hence, they've found their hammer and everything else is a nail.
They'll then rant that anyone who suggests using anything other
than a hammer is unjustified and, once again, write up
yet another stupid blog post about how they don't make programmers
like they used to back then.

There's all kinds of weird voodoo blanket statements programmers adopt
like "You should **usually** adopt X instead of Y", "You should **usually**
be optimizing for Z instead of W", etc.

Or, for those developers less inclined to euphemism,
replace **usually** with **always**

```
For example:

X = smart pointers
Y = raw pointers

Z = clean code
W = performant, dirty code (from all those dirty caches...)

I'm not arguing Y or W is necessarily better than X or Z, but that
I tend to find developers are waaaaaaaay too dogmatic about this
and they don't actually experiment or explore outside their little
bubble enough.
```

## Wait, That's It?

Yeah. That's it. Well, actually, that's it from my takes that
are "hot". I have many other takes that are also pretty common
in the developer world as well that I'd rather not bore you
with for the 65,536-th time.



