---
layout: post
title: Bridging patterns and programming 
---

It's hard to escape the feeling, reading the Gang of Four *Design Patterns*
book, that there are slightly too many design patterns. Many of the patterns
seem to have similarities and homologies, leading to the suspicion that with
just a little more abstraction some of the patterns could be combined. After
thinking about this for a while, though, I'm not sure this objection is right;
at any rate, considering why it might not be helpful to abstract away
differences between patterns has clarified some stuff for me about the point
of design patterns.

What leads to the suspicion of there being a few too many patterns are the
many often-remarked on similarities between different patterns (including some
that are remarked on in the GOF book itself). What got me thinking was a
similarity that hadn't occurred to me before (and which Google suggests
isn't commonly discussed), that between the bridge and builder patters.

## Bridge

The bridge pattern is used, according to the GOF, to

> decouple an abstraction from its implementation so that the two can vary
> independently.

That is, you have a hierarchy of classes which can be implemented in terms of
some set of operations, and that set of operations can itself have multiple
implementations. The example they use is a hierarchy of widgets in a GUI, each
of which could itself be implemented by many different window systems.
Something like

```scala
trait Window {
    def setPosition(x: Int, y: Int): Unit
    def draw(): Unit
    def onClick(handler: Int => Unit): Unit    
}

trait Button extends Window {
    def setCaption(caption: String): Unit
}

trait List extends Window {
    def setEntries(entries: List[String])
}
```

Rather than having a separate implementation of each interface for each window
system, you provide an interface for basic window system operations, and you
implement the widgets in terms of this interface, e.g.

```scala
trait WindowOperations {
    def drawRect(left: Int, top: Int, right: Int, bottom: Int): Unit
    def drawText(x: Int, y: Int, text: String): Unit
    def registerHandler[T](event: Event, handler: T => Unit): Unit
}

class WindowImpl(ops: WindowOperations) extends Window {
    var x: Int
    var y: Int

    override def setPosition(x: Int, y: Int): Unit = {
        this.x = x
        this.y = y
    }

    override def draw(): Unit {
        ops.drawRect(x, y, x + width, y + height)
    }

    override def onClick(handler: Int => Unit): Unit {
        ops.registerHandler(Click, handler)
    }
}
```

Then you could provide as many implementations as you need of the
`WindowOperations` interface: `MSWindowOperations` for Windows,
`XWindowOperations` for X, etc.

## Builder

The builder pattern, again according to the GOF, is used to

> separate the construction of a complex object from its representation so
> that the same construction process can create different representations.

