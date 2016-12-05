---
title: Labels and things
---
The Birkbeck Computer Science MSc I'm currently taking includes a required
module on Programming in Java, with the quite sensible purpose of making sure
everyone on the course has a shared facility in a common language (and the MSc
is specifically intended for people coming to CS from other disciplines, so it
makes sense not to assume this shared skill in advance). I have a fair amount
of programming experience, so the aspect of the class about programming in
general hasn't taught me a huge amount (though I have learnt about some of the
peculiarities of Java), but it has been useful to observe what people find
difficult in learning to program, and to think about ways in which a
programming course could ease some of these difficulties.

Once thing that definitely seems to be causing people difficulties is figuring
out what names refer to at different parts of the program; for instance, what
writing `name = "Tim"` means exactly, or what happens when you have variables
with the same name in different functions. I've been thinking about how to 
structure a programming course to make this as clear as possible. 

## Expressions and evaluation

I think it makes sense to start witht the idea of expressions and evaluating
them. You can start with a simply expression like `5 + 5`, in order to
introduce the idea that when a program runs, a lot of what it does is taking
these expressions and evaluating them, that is, working out a result. You can
then go on to consider compound expressions, like `(5 + 5) * 10`, which gets
across the important idea that the program uses the result of evaluating one
expression in working out the result of evaluating another expression. This
might seem obvious, but I think being very explicit about this right at the
beginning would provide a solid foundation for the next few stages.

## Labels and things

The next concept to introduce is variables. Like I think many people of my age
(i.e, people who started learning to program with BASIC), I was introduced to
variables through the metaphor of boxes: a variable is basically a box, oh and
by the way it also has a name. I think this view of variables is too low-level
to be helpful at the early stages of teaching programming (I should note in
passing that I think this may have been something that confused students on
the course I'm currently taking: it introduced the distinction between
primitive types, stored on the stack, and objects, stored on the heap, very
early on, but in Java this is an implementation detail that is almost always
irrelevent); my preference would be to introduce variables as a connection
between a label and a thing. A variable name, like `name` is a label, and it
is attached to a thing, like the word `Tim` or the number `100`. More
specifically, a thing is the result of evaluating an expression; and, neatly,
the result of evaluating a label is just the result you got when you evaluated
the expression in its definition.

I think it would make things clearer to treat variables as immutable at this
point, and perhaps therefore to teach in a language that enforces that (a
concise syntax for introducing an immutable binding would be helpful, like
`val name = "Tim"` as used by, among other languages, Kotlin). The question of
"what happens when a label refers to different things at different points in
the program is a complicated one, and one that should be introduced in its own
right.

(You could also introduce the language's basic IO functions at this point, to
allow students to write interactive programs. A very lightweight IO syntax,
like Python 3's `print()` and `input()` would be nice.)


## Conditionals

I think there are two fundamental operations in how we usually think about
programs, the first of which is conditionals. I don't think these are
particularly challenging for people learning programming (but I might be
wrong). Following on my recommendation above to teach immutable variables
first, I guess it would make sense to teach functional-style conditionals that
return a value, rather than the if statements that you get in the imperative
languages usually used for teaching. I wonder if there's any difference in
ease of understanding between something like:

```scala
val isOddOrEven = if (n % 2 == 0) "Even" else "Odd"
```

As opposed to:

```python
if n % 2 == 0:
    odd_or_even = "Even"
else:
    odd_or_even = "Odd"
```

## Repetition

The second of the two fundamental operations in programming is repetition,
which is more complicated than conditionals because it requires introducing
the idea that the *same* code can do *different* things. Well, you can begin
with a simple unconditional repetition, like the classic logo program to draw
a square:

```logo
REPEAT 4 [
    FORWARD 100
    RIGHT 90
]
```

But the critical point, and I think the harder one for people learning
programming to grasp, is the conditional loop, like, say:

```c
while (number != 1) {
    if (number % 2 == 0) {
        number = number / 2;
    } else {
        number = 3 * number + 1;
    }
}
```

There are two ways of explaining this kind of fully general loop - mutable
variables, as I've used in the example above, or function calls, as in:

```haskell
collatz number = 
    if number == 1 then 
        () 
    else
        if even number then
            collatz (number `div` 2)
        else
            collatz (3 * number + 1)
```            

What both share, and which I think makes them tricky to understand, is this
idea of re-binding variables; that the same symbol (in these examples,
`number`) will refer to different things at different times. I'm tempted to
say there are two hard problems in teaching programming - the idea that labels
refer to things, and the idea that labels refer to different things at
different times. That's why I started this post by proposing to introduce the
distinction betwee labels and things very explicitly early on. But having done
this, it seems to me that there's a significant further hurdle in introducing
the idea that labels *change* what they refer to in predictable but
potentially complex ways. Thus I'd like to introduce the idea of re-binding in
a limited and controlled way before moving to the full generality of
mutability or function arguments.

And it occurs to me that theres an easy to grasp special case of repetition
which might make a good introductory step to an explicit discussion of
re-binding of variable names: repetition over sequential numbers.  

Is it time to revive the BASIC FOR...NEXT loop as a pedagogical tool?

```basic
FOR I% = 1 TO 10
    PRINT I%
NEXT
```

