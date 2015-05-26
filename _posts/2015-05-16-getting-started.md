---
layout: post
title: Getting Started
---

The first issue I ran into getting started with my implementation was figuring out where to start. FHIR has great documentation for developers using an implementation, and that's who documentation should primarily cater to. Alas, however, there seemed to be little documentation for those folks writing a new implementation (hopefully this blog can address some of that need).

From poking around existing implementations, it seemed like most implementations write code that parses the FHIR schemas, and generates types that can be serialized/deserialized to JSON/XML. Now, there's a very old debate between folks who like model driven approaches with generated types that guarantee conformance, and folks who like flexible approaches that accommodate on-the-wire variability. The classic example of the former approach is interoperability based on classes generated from XML Schemas; latter approaches include metaprogramming approaches that mimic field access based on on-the-fly parsing, or declarative (e.g., XPath or CSS selectors), functional or LINQ-based approaches that use selectors and functions to access data. There's a design trade-off here that goes back to [Postel](http://en.wikipedia.org/wiki/Robustness_principle): you can guarantee interoperability by strict contract conformance at the cost of brittleness when you receive something you don't expect, or allow flexibility at the cost of sometimes making interpretative decisions.

My experience in interoperability leans heavily towards the latter approach on read, and the former approach on write: that it, one wants to generate conservatively, and receive liberally. In general, however, with a clear and unambiguous spec, writing correctly is a trivial exercise (generally a simple matter of templating correctly with robust unit and functional tests), and reading robustly is hard. Writing is only a problem when the specification itself is ambiguous or allows for high degrees of optionality, and schemata don't help much there. (Alas, the poorly constrained specification problem is rather endemic in health care).

In any case, I'm not yet able to make any of those decisions because there are certain basic concepts I need to represent. The issue is figuring out what those concepts are.

There's a [whole section](https://hl7-fhir.github.io/datatypes.html) documenting data types, and I initially went down the path of starting with the definition of [primitive types](https://hl7-fhir.github.io/datatypes.html#1.18.0.1). More on that in next post.

I ran myself into a corner implementing primitives, however, because it wasn't clear what the basic structure of FHIR is. I suspect that FHIR was originally developed by creating good simple XML definitions of basic healthcare data structures, and then cross mapping to JSON. That is to say, FHIR was originally a top down exercise based on XML data representation, and then was generalized. This drives some basic representation decisions: an `Element` is, well, an XML `Element`, with `value` being an attribute. Given that definition, we only need to define what kinds of values can be represented as primitives, and sub-elements are given in representation. With XML Schema-based code generation, the top level element translates to a class name, and sub-element names translate to properties or fields.

For someone who wants to go the XML Schema-based code generation route, the implementation approach is something like this:

- Implement the primitive types
- Implement the compound types
- Implement code generation via the the resource definitions or profiles, with built in XML serialization/deserialization
- Implement a custom JSON serialization/deserialization approach (because the JSON serialization format is idiosyncratic, you are going to have to build this from scratch)

That approach probably works well for those for whom XML and XML Schema are a first language and wants a code generation approach, but not so well for someone trying to model FHIR types as a first class concept, who dislikes code generation, and who liked XML when it was first released, but avoided attributes like the plague, and started backing slowly and carefully away from XML when namespaces and schemas were added.


The breakthrough for me was to realize that the basic FHIR element is a key/value pair, where the key is a lowercase ASCII name (or at least all the extant examples are lowercase ASCII), and the value is either a typed atomic primitive, or a bag of elements. (It's slightly more complicated than that, because `Element`s have optional `id` and `extension` attributes).

Based on this breakthrough, the rest of the representation decisions fell into place. More on that in the next post.