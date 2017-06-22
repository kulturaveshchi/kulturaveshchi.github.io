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

You use this pattern where you have a complex construction process that can be
built up from simpler parts, and you have multiple implementations of those
parts. The example the GOF use is creating different representations of text
from an RTF. The class that controls the construction process (in this case,
parsing the RTF) is called the 'director'; the classes that implement the
elements of this construction process are called 'builders'. 

```scala
trait TextBuilder {
    def addParagraph(): Unit
    def addCharacter(char: Char): Unit
    def changeFont(newFont: Font): Unit
}

class RTFReader {
    def parseRTF(rtf: String, builder: TextBuilder): Unit = {
        // Imagine this parses the rtf string and calls
        // methods on the builder for each relevant part
        // of the RTF.
    }

}

class PlainTextBuilder extends TextBuilder {
    private sb: StringBuilder = new StringBuilder

    override def addParagraph(): Unit = sb.append("\n\n")
    override def addCharacter(char: Char): Unit = sb.append(char)
    override def changeFont(newFont: Font): Unit = ()

    def getString: String = sb.toString
}

class TextWidgetBuilder extends TextBuilder {
    // Using some imaginary windowing system's text widget
    private widget: WindowSystemTextWidget = new WindowSystemTextWidget

    override def addParagraph(): Unit = widget.newParagraph()
    override def addCharacter(char: Char): Unit = widget.append(char)
    override def changeFont(newFont: Font): Unit = widget.setFont(font)

    def getWidget: WindowSystemTextWidget = widget
}
```

## Bridge and builder: similarities and differences

So in the builder pattern, you have some functionality (complex construction),
implemented in terms of an API which itself may have a number of
implementations. Described that way, the builder pattern sounds a lot like the
bridge pattern. There are differences, though, in the exposed functionality.
The bridge pattern suggests that the functionality is fairly rich - in the
example, the functionality exposed abstractly by `Window` and the other traits
is sufficient to build a whole GUI. In the builder pattern, the functionality
is much simpler - the director class, `RTFReader` only has one method. What's
more important, though, is what's missing from this abstracted functionality:
the method to get the constructed object only exists in the concrete builder
implementations, not in the director class or the abstract `TextBuilder`
class.

This is intentional. The GOF write:

> Because the client usually configures the director with the proper concrete
> builder, the client is in a position to know which concrete subclass of
> Builder is in use and can handle its products accordingly.

The difference here is one of point-of-view. The bridge pattern approaches the
question from the point of view of a client of an abstraction, who wants to be
able to use it without caring about the underlying implementation. Hence the
process of creating the concrete implementation is encapsulated behind an
interface that deals only in abstract classes.

The builder patter, on the other hand, approaches the question from the
perspesctive of the implementer of a complex algorithm, and it is this
algorithm which is encapsulated in the concrete director class.

And this is why thinking about design patterns in terms of language features
tends to lead us astray. You sometimes hear design patterns dismissed with
variants on the argument that you don't need design patterns if you have
higher-order functions (or macros, or some other higher-level programming
technique), but that confuses the implementation with the motivation. This
mistake is sometimes reinforced by how design patterns are taught. There's
often an emphasis on implementing the GoF patterns in highly contrived
situations, with no discussion of what makes the pattern appropriate for that
situation; this is often because the pattern *isn't* appropriate for the
contrived situation (indeed, this post was in part inspired by my frustration
at [this sequence of
videos](http://www.newthinktank.com/videos/design-patterns-tutorial/), which
strike me as a textbook example of how not to teach design patters).

The implementation of design patterns isn't particularly important. More
important is that they provide perspectives from which to see problems, and
thereby to see similarities, and also differences between problems. This
ability to approach programming at a level of abstraction above that
explicitly represented in code, and with the flexibility and imagination that
can be spurred by analogical thinking, is important no matter what programming
language or paradigm you're using (there is an interesting debate to be had
about the relative value of the analogical thinking encouraged by design
patterns, and the formal thinking encouraged by mathematical approaches to
programming, but that will have to be another post).
