---
layout: post
title: Union types
---

Long time, no updates. I discovered soon after I thought I was complete, that I had missed an important FHIR concept. In the documentation for [resource formats](https://hl7-fhir.github.io/formats.html), there is a mention of "an element which can have one of a several different types". A wild type example is found in the `Extension`s that I struggled with; [the `Procedure` resource](https://hl7-fhir.github.io/procedure.html) contains two examples (`site` and `performed`).

In the computer science type theory literature, these are [tagged union](https://en.wikipedia.org/wiki/Tagged_union) types. This means that the underlying FHIR type model is quite powerful indeed, at the cost of some complexity. Figure out how to implement union types put me into a bit of a spin. My goal was to implement the underlying primitive types in a way that could be reused across primitive atomic and union types. There was accordingly much [shaving of yaks](http://joi.ito.com/weblog/2005/03/05/yak-shaving.html), which led to some interesting work, but zero productivity relative to this project.

Finally, I decided to do this the simplest possible thing, which was reuse my existing union of atomic primitives, and let the programmer use them correctly.

This leads to a pretty simple (but also wrong) definition of a union:

```rust
pub enum Union {
    Atom(Primitive),
    Element(Box<Element>)
}
```

(Ignore the `Box` bit: this is a bit of indirection to avoid a recursive allocation because `Element`s contain `Values` which contain `ValueTypes` which can be `Union` which contain `Element`s),

The definition is wrong because only *some* `Primitive`s and *some* `Element`s are allowed.

Our `Value`s, then, are one of the following:

```rust
pub enum ValueType {
    Atom(Primitive),
    Union(Union),
    List(Vec<Value>),
    Elt(Vec<Element>)
}
```

Just as we saw in [the post on implementing extensions](http://arienmalec.github.io/implementing-fhir/2015/06/18/extensions-2/), there's a bit of gymnastics involved in creating the proper name for serialization, because the name embeds the type information. (Alternative, FHIR could have mapped unions to a name/type/value triple -- likely this was rejected because, although more expressive, it's also harder to see what's going on).

I've got some work to do in working this change all the way through my implementation, and then I'll do a documentation post on the overall FHIR type system.

