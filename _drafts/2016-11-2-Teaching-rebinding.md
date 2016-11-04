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
