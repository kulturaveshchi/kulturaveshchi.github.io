---
title: Rough guide to Intel's Spark CRF library
layout: post
---

Spark includes a machine-learning library implementing a number of useful
statistical techniques, but one it does not include is Conditional Random
Fields, which are a popular choice for classifying tokens in a sequence (I
think they're probably best known as a technique for part-of-speech tagging,
but I'm interested in using them to classify paragraphs in documents based on
their semantic role). There is a Spark-based CRF library, though, which is
part of [Intel's IMLLIB](https://github.com/Intel-bigdata/imllib-spark).
Unfortunately, it's not very well documented, so I've spent the past couple of
days figuring out how to use it, which I thought I'd document here in case
it's of any use to anyone else (even if that other person is just me in a few
weeks time). <!-- more -->

The library has two primary entry points: `CRF.train(templates, sequences)`, a
method on the CRF object which takes an array of features templates and an
`RDD` of `Sequence`s (which must have labels already attached), returning a
`CRFModel`; and the `predict(sequences)` method on this `CRFModel` instance,
which takes an `RDD` of `Sequences` and produces another `RDD` of `Sequences`,
this time with predicted labels attached to each token. The main issue I had
in figuring out how to use the library was working out what format it expected
for sequences, tokens and, especially, feature templates.

## Sequences of Tokens ##

CRF models are used to classify tokens within sequences, so it makes sense
that the primary data types used to interface with the CRF library are
`Sequence` and `Token`. `Sequence` is mostly just a case class that wraps an
array of `Token`s (it also includes some more complicated functionality to
deal with multiple different classifications, for instance if you want to use
your model to get the five most probably classifications, rather than just the
top one, but I haven't used that functionality). A `Token` is an array of
`Strings`, each string being one attribute of the token, along with another
`String` which is the label assigned to that token; if no label is assigned,
the label field should be set to `null`. 

## Feature templates ##

The feature templates are a bit more complicated, in part because of the (to
my simple mind, anyway) slightly more complicated nature of CRF features. In a CRF model, a 
feature isn't just an attribute of a token - rather, a feature can depend on both attributes or 
classifications of other tokens in the sequence. So, for example, 'was the previous paragraph 
bold' could be a feature, as could 'what was the previous paragraph classified as', 
or 'what was the previous paragraph classified as, and is this paragraph bold'. 
The first of these is called a unigram feature, because the classification
of a token does not depend on the classification of any other token, while the latter two are 
bigram features, because they specify a relationship between the classification of two tokens. 
Both of these bigram features only refer to the classification of the previous token: general CRF
models allow bigram features to refer to the classification of any other token in the sequence, 
but IMLLIB's CRF engine only allows bigram features that relate adjacent tokens (I believe such a
model is called a 'linear chain CRF').

So, tokens are sequences of attributes, while features are specified in terms of an attribute and
the relative position of the token that has that attribute (and, optionally, the classification 
of the previous token). These are specified using feature templates, for which IMLLIB inherits 
[CRF++](https://taku910.github.io/crfpp/)'s rather obscure syntax. You supply the feature templates as an array of strings, where 
each string is one template, having the following format:

  - the letter `U` (for a unigram feature) or `B` (for a bigram feature)
  - an arbitrary string, which is the label for this feature
  - a colon, `:`
  - the string, `%x`
  - an open square bracket, `[`
  - the relative position of the *token* to consider for this feature, e.g. `-1`
  - a comma, `,`
  - the index into the array of attributes in the `Token` instance, e.g. `3`
  - a closing square bracket, `]`
  - (there's an additional optional format that allows you to refer to more than one attribute in
    a feature, but I'm ignoring that here )
  
IMLLIB parses these tokens in a very simple way, making much use of hard-coded indexes and with 
very little validation, so it's important to make sure that you are providing strings that 
exactly match this format (with, for example, no whitespace around the brackets or between the 
two numbers). For the features specified above, the feature templates would be something like 
(assuming 'is bold' is attribute 10 in our attributes array):
  
  - was the previous paragraph bold: `UprevBold:%x[-1,10]`
  - what was the previous paragraph's classification: `B` (note here that there is no colon or 
    bracketed part, because the feature only depends on classifications, not attributes)
  - what was the previous paragraph's classification, and is this paragraph bold: 
    `BthisBold:%x[0,10]`
    
## Generating feature templates ##

Rather than using this rather obscure syntax directly, I've been generating my feature templates 
and attribute arrays from code. I have a very simple model, where every attribute is only used in
one feature, so it's easy to map a sequence of feature specifications both to a sequence of 
strings representing these attributes. I represent tokens as a case class, like:
 
```scala
case class Paragraph(
  isBold: Boolean,
  fontSize: Int
)
```

and feature templates as a sequence, where each template knows how to extract the relevant 
attribute:

```scala
trait FeatureTemplate[A] {
  // Extract the relevant attribute from the token model
  def extractor: A => Any
  
  // Specifies the relative position of the token to consult
  def relative: Int
}
case class Unigram[A](extractor: A => Any, relative: Int = 0)
case class Bigram[A](extractor: A => Any, relative: Int = 0)

object Features {
  val template = Seq(
    Bigram(_.isBold),
    Unigram(_.fontSize)
  )
}
```

Because, in my model, each attribute is used in just one feature, I can generate attribute arrays
where the index of an attribute corresponds to the index of the feature in the sequence of 
feature templates, like so:

```scala
def tokensAsStrings[A](templates: Seq[FeatureTemplate[A]], 
                        tokens: Seq[A]):
    Seq[Array[String]] = {
  tokens.map { token =>
    templates.map { template =>
        template.extractor(token).toString
    }.toArray
  }
}
```

and then generate feature templates which use indexes corresponding to these attribute arrays 
(this example uses a `tag` method on the FeatureTemplate trait which supplies `U` for unigram 
features and `B` for bigrams):

```scala
def templatesAsStrings[A](templates: Seq[FeatureTemplate[A]]): 
    Array[String] = {
  templates.zipWithIndex.map {
    case (template, index) =>
      s"${template.tag}$index:%x[${template.relative},$index]"
  }.toArray
}
```  

This generates a list of attributes from the list of feature templates, and produces an attribute
array which ensures that the index of an attribute in this array is the same as the index of the 
feature template in the list of feature templates. If any attribute were used in more than one 
feature template, this wouldn't work - you would need a separate list of attributes and feature 
templates, and some way of relating the two. I think this could be automatically generated with 
some kind of metaprogramming, but I haven't looked into that. For my needs, this simple method 
suffices.