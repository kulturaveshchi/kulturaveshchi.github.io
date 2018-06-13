---
layout: post
title: On not avoiding null pointer exceptions
excerpt_separator: <!-- more -->
tags: 
    - Java
    - Type systems
---

A common piece of Java advice that floats around encourages programmers to use
"Yoda conditions" - that is, to structure comparisons so that the subject of
comparison is on the right hand side of the expression, while the thing it is
being compared against is on the left. Instead of

```java
name.equals("Yoda");
```

we are advised to write:

```java
"Yoda".equals(name);
```

This is bad advice; but I think the reason *why* it is bad advice illustrates
some interesting features of how (and how not) to write reliable software.<!-- more -->

To understand why Yoda conditions are bad, we need to understand why they are
advocated in the first place. The claim is that they help us avoid null pointer
exceptions. This isn't false, in that, if `name` is `null`,
`name.equals("Yoda")` will produce a null pointer exception, while
`"Yoda".equals(name)` will not. It doesn't follow from this fact, though that
we should adopt Yoda conditions, and the reason is that "avoiding null pointer
exceptions" is not a goal that we should be pursuing, and thinking in terms of
avoiding null pointer exceptions tends to lead to thinking about the role of
null in our software in ways which lead to less reliable software.

The fundamental misunderstanding that leads people to advocate Yoda conditions
is the idea that null pointer exceptions are a problem, and so our software
will be more reliable if we eliminate null pointer exceptions. But null pointer
exceptions are not a problem: they are a symptom. Null pointer exceptions are a
sign that the actual data flow of the program doesn't match the specified (or
assumed) data flow, so that a null pointer got to a point in the program where
it makes no logical sense.

Whether it makes sense for a variable to be null or not is a fundamental
feature of a program's logic. Where null is a logicaly permissible value, it
should be considered explicitly; where null is not a logically permissible
value, a null pointer exception in the case of a null value is *good*. If a
null value value arises in such a cases, the program's invariants have been
violated, which means you can't rely on its behaviour. In this case, the best
thing to do is for the program to explicitly fail and to rely on error recovery
at a higher level.

Yoda conditions fail in both cases - they are not explicit about handling the
null case, which makes them inappropriate if nulls are expected, and they do
not produce an error in the presence of nulls, so they are inappropriate if
nulls are not expected. Yoda conditions suggest you can avoid null pointer
exceptions without thinking, just by always coding to a particular template.
This is the opposite of what we need to produce reliable; we should be paying
*more* attention to the invariants at different parts of our programs.

Luckily, we can outsource some of this work to our tools. `Optional` types
explicitly mark those cases where the absence of a value makes sense, and make
handling both the present and absent case more straightforward (`map` handles
the case where a value is present, while `orElse` and similar methods handle
the case where a value is absent). `@Nullable` and `@Nonnull` annotations allow
IDEs and static analysis tools to keep track for us of where nulls are and
aren't permissable. These tools can help us to address the real problem of
understanding the logical structure of our code, rather than encouraging us to
stop thinking about reliability and just hide the symptoms.