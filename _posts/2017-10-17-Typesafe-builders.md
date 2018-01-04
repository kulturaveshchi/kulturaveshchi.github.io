---
layout: post
title: Typesafe builders
excerpt_separator: <!-- more -->
tags: 
    - Java
    - language ergonomics
---

I've been writing a lot of Java for work recently (after an extended sojourn
in Scala for [my master's project](https://github.com/culturedsys/lucida)),
and it's got me thinking about language ergonomics: the ways in which we adapt
programming languages to make them easier to write (I actually think many
common Java idioms which are supposed to make it easier to write code, don't;
there may be more posts soon-ish on some of these).

One common pattern in Java is the simple builder. I call it a simple builder
to distinguish from the full-blown Builder pattern presented by the Gang of
Four. The full Builder pattern is used when you have a complex construction
process which is shared between a number of different representations. The
example the GoF give is a function that reads a rich-text document and can be
used to build a document model in a number of different formats (TeX, HTML,
GUI widgets).

The simple builders commonly encountered in Java are much simpler: they
usually only construct one type, and the construction process is usually
straightforward, just involving setting a few parameters.<!-- more --> Here's
an example, loosely based on the Amazon Web Services Java SDK:

```java
Client client = 
    Client.builder()
        .withRegion("us-east-1")
        .withCredentials(previouslyLoadedCredentials)
        .build();
```

This example only sets two parameters for the constructed object, but you could
set more (The Amazon Simple Email Service client, for instance has
`withClientConfiguration` and `withEndpointConfiguration` methods, among many
others); indeed, it's this wide range of potential, sometimes optional,
parameters which motivates the use of the simple builder. Traditionally, you
specify parameters for instance creation as arguments to the constructor, but
this can get unwieldy if you have a lot of arguments (it's hard to remember
which position in the argument list means what), and can't easily deal with
some parameters being optional (you might be able to get away with overloading
the constructor, but if you have multiple optional argument of the same type,
there's no obvious way to distinguish between constructors for each type).

So, these simple builders are a workaround for the fact that Java doesn't have
keyword parameters (which allow you to specify what each parameter means by
name, not by position) or default argument values (which allow you write one
constructor which is aware of when optional values are left out). That is, you
can't just write:

```scala
new Client(region = "us-east-1", 
            credentials = previouslyLoadedCredentials);
```

And simple builders function well enough as an alternative, but they do have
one definite disadvantage, at least in the way they are usually implemented:
they're not type safe. One of the reasons we have constructors is to ensure
that objects are not initialised in an incomplete state, because the compiler
ensures that we pass all the required parameters to the constructor. With a
simple builder implemented in the most obvious way, we don't have that
assurance - the compiler will not ensure that we have called the
`withSomeRequiredParameter` method on our builder before calling `build`, an
error which would only be detected at run time.

However, at the cost of a certain amount of boiler-plate, it is possible to 
combine the explicitness of the builder interface with the type safety of
constructors.

What you need is a build function which typechecks when all required 
parameters are provided; that is, we need to be able to represent the presence 
or absence of values at the type level. We can use inheritance for this purpose:

```java
abstract class Parameter<T> { 
    public abstract boolean isPresent();
}

class Absent<T> extends Parameter<T> { 
    @Override
    public boolean isPresent() {
        return false;
    }
}

class Present<T> extends Parameter<T> {
    private T value;

    public Present(T value) {
        this.value = value;
    }

    @Override
    public boolean isPresent() {
        return true;
    }

    public T get() {
        return value;
    }
}
```

Now you need a class to store your, present or absent, parameter values - 
again ensuring that the presence or absence of the parameters is represented 
in the type. You can do this by making your class generic, specifying the 
presence or absence of the required parameters as type parameters. You end up 
with a type declaration like: 

```java
class Parameters<RegionType extends Parameter<String>, 
        CredentialsType extends Parameter<Credentials>, 
        FlagType extends Parameter<Boolean>> {
    private RegionType region;
    private CredentialsType credentials;
    private FlagType flag;

    // Constructor and getters omitted.
}
```

Then, the methods to specify particular parameters return an instanciation of 
the generic type, with the type altered to indicate that the particular 
parameter is now present. So, for instance, the `withCredentials` method is 
specified like so: 

```java
public Parameters<RegionType, Present<Credentials>, FlagType> 
withCredentials(Credentials credentials) {
    return new Parameters<>(region, new Present<>(credentials), flag);
}
```

Note that we pass through `RegionType` and `FlagType`, which could be 
`Present<>` or `Absent<>`, so that if we call `withCredentials` on an object 
that has already had the region set, that will be maintained in the type of 
the returned object.

So, now that we can produce objects which specify in their type whether or not
they have values for particular parameters, we can write a builder function
that only accepts values which have all the required parameters specified. If,
for example, `region` and `credentials` are required parameters, but `flag` is
an optional parameter that defaults to `false`, we could write a builder
function like:

```java
public Client build(Parameters<Present<String>, 
                            Present<Credentials>, 
                            ? extends Parameter<Boolean>> parameters) {
    String region = parameters.getRegion().get();
    Credentials credentials = parameters.getCredentials().get();
    boolean flag = (parameters.getFlag().isPresent()) ? 
                    ((Present<Boolean>) parameters.getFlag()).get() : 
                    false;

    return new Client(region, credentials, flag);
}
```

Note that the `build` method takes the parameters as an argument, rather than
being a method on the class that accumultes parameters, as is usually the case
with builders. This is because we only want this method to work when supplied
when the `Parameters` class is instanciated with `Present<>` types for the
required parameters. There is no way of providing a method only on some
instancations of a generic type, so (as far as I know), there is no way of
making the `build` method a method of the `Parameters` class and maintainging
the type safety we want. So build would probably be a method of the `Client`
class, and you would use it something like:

```java
Client client = 
    Client.build(Client.parameters()
                    .withRegion("us-east-1")
                    .withCredentials(previouslyLoadedCredentials));
```

Anyway, this shows us that the Java type system can give us more type safety
than I initially thought; it also shows us that, if Java just had keyword
parameters, we could save a hell of a lot of boilerplate.

(The [code for this
example](https://gist.github.com/culturedsys/0a9418f0b2bb0ba00f7d10879368a34d)
is available as a gist.)