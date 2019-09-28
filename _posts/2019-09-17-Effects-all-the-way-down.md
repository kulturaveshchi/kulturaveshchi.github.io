---
layout: post
title: Effects all the way down
tags: scala, types
excerpt_separator: <!-- more -->
---

I was recently writing some code using Scala's [fs2](https://fs2.io/) streaming library, where I wanted
to produce a stream of effects, starting with a given effect and then generating
each subsequent effect from  the value produced by the preceding effect. Unless I missed something,
fs2 doesn't seem to have this facility built in; there's `iterateEval`, which
produces a stream of effects, but this requires a seed value which is itself
*not* produced by an effect. It turns out to be fairly straight-forward to write
the function I wanted; I'm posting it here in the hope it will be useful for anyone
facing the same problem I did (as always, that 'someone' could well be future me).

```scala
def iterateEvalFromEffect[F[_], A](start: F[A])(f: A => F[A])
        : Stream[F, A] =
    Stream.eval(start)
            .flatMap(s => Stream.emit(s) ++ 
                    iterateEvalFromEffect(f(s))(f))
```

This also turned out to be an interesting example of how types can help guide development.<!-- more -->
I began by looking at the source for `iterateEval`, which is:

```scala
def iterateEval[F[_], A](start: A)(f: A => F[A]): Stream[F, A] = 
    emit(start) ++ eval(f(start)).flatMap(iterateEval(_)(f))
```

My initial thought was that I could just change the `start` parameter be an effect (of 
type `F[A]`), and replace the first call to `emit` with one to `eval`. But it 
became obvious when I looked at the recursive call that that wouldn't work, because
the function passed to `flatMap` has to take an `A`, not an `F[A]`. So I started
thinking about how I would write the function so that I would have access to
an `A` when I needed one to emit, and an `F[A]` when I needed one to eval. Once I
started thinking about that, it didn't take long to come up with the function I
needed. Of course, I could have undertaken the same thought process without types,
but I think the type system giving me clear labels for the different sorts of 
thing I was dealing with, and forcing me to think about what kind of thing I
needed at each point in the algorithm, added some really helpful clarity to the
development process.