---
layout: post
title: Extensions
---

`Extension`s in FHIR provide a space to allow additional values and semantics against defined FHIR resources, without requiring implementors to hack into the design space of the FHIR resources themselves. They play the same role that `x_` header values or, perhaps better, IANA registration of MIME and other types.

Conceptually, `Extension`s are bundles of values (or extensions!) that are scoped by a URI, where the semantics of the values are defined by the `Extension` creator.

When thinking about the low level representational issues, the first thing to note is that an `Extension` is defined as an `Element`, but it has different characteristics from an `Element`. The content of an `Extension` is defined as either:

1. A set of sub-`Extension`s (though the `extension` sub-element of every `Element`
2. A single `valueFoo`s where `Foo` can take a specified set of values constrained by the type indicated by the `Foo` part, where `Foo` is either one of our `Primitive` values or is a constrained `Element` value.

If you want multiple values in your extension, I presume you need to use sub-extensions, and scope each value by the URI that provides its semantics.

Accordingly, I'm not going to represent `Extension` as a type of `Element`, because it shares only the ubiquitous `id` and `extension` values of `Element`, but even the `extension` content is special because it conditioned on the absence of a `valueFoo`.

Here's a tentative definition:

```rust
pub enum ExtensionValue {
	Integer,
	Decimal,
	DateTime,
	Date,
	Instant,
	String,
	Uri,
	Boolean,
	Code,
	Base64Binary,
	Coding,
	CodeableConcept,
	Attachment,
	Identifier,
	Quantity,
	Range,
	Period,
	Ratio,
	HumanName,
	Address,
	ContactPoint,
	Timing,
	Signature,
	Reference,
	Extensions(Vec<Extension>) 
}

pub struct Extension {
	id: Option<String>,
	uri: Url,
	value: ExtensionValue
}
```

There's some more that we need to do here (e.g., map value types to value names, figure out how to re-use our `Primitive` definitions, and get some implementations of the basic composites like `Identifier` and `Quantity` going).

There's, by the way, a much simpler definition of `Extension` that's possible:


```rust
pub enum ExtensionValue {
	Atom(Primitive),
	Composite(Element),
	Extensions(Vec<Extension>) 
}

pub struct Extension {
	id: Option<String>,
	uri: Url,
	value: ExtensionValue
}
```

This helps draw out the underlying structure better, at the cost of being less expressive.