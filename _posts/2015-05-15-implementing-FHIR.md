---
layout: post
title: Implementing FHIR
---

One of the best ways to understand a specification is to implement it, and one of the best ways to improve a specification is to implement from scratch and provide comment on any area of ambiguity.

I've been a huge champion of [FHIR](https://hl7-fhir.github.io) from the [early days](https://twitter.com/amalec/status/201750234861285376), and engineering teams I've worked with have implemented FHIR in multiple contexts. But I've never implemented from scratch, following only the spec.

At the same time, I've been playing with a new systems programming language, [Rust](http://www.rust-lang.org), and one of the best ways to master a new language is to write a non-trivial program in it. Generally, you want to implement a new spec in a language you know, and learn a language programming in a domain you know, but what the hell.

Rust is an interesting language to write an implementation in. It is a very strongly typed language with a highly expressive type system, implementing [Algebraic Data Types](http://en.wikipedia.org/wiki/Algebraic_data_type), that are immutable by default. This implementation space forces a set of design decisions that are interesting from the perspective of looking at a specification. If I were to write an implementation in Ruby, for instance, I'd be more likely to go top down, and focus primarily on developer ergonomics and flexibility; in Rust, I'm inclined to implement bottom up and get the low level primitives formally designed. That leads to more focus on some areas of the spec that are not often looked at.

Some caveats: I'm not a master programmer, have generally only the odd hour nights and weekends, and have no guarantees of being able to finish the project. I doubt this will even be a workable reference implementation. I *will* learn something, though, and hope you will as well. If I'm really lucky, I'll contribute some useful insights back to the FHIR specification.

Code will be available in [a Github repository](https://github.com/arienmalec/fhir-rust).

In the first installment, I'm going to start at the very beginning.