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
weeks time).

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

