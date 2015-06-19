---
layout: post
title: Extensions, Part 2
---

I [suggested two approaches](http://arienmalec.github.io/implementing-fhir/2015/06/13/extensions/) for representing `Extension`s, one explicit, one implicit. I decided on the implicit approach. One is because I haven't yet figured out how to make `Element`s typed by specification; the other is that FHIR has an odd mismatch between `Extension` allowed atomic values and `Primitive` allowed atomic values.

In particular, the basic `Primitive` atomic values include `Time` but this value is not allowed in `Extension` atomic values, and `Extension` allows `Code` from the specialized `Primitive`s, but none of the other specialized values. I'm not sure if that's a bug or intended; it makes it hard for us to define a space for basic `Primitive`s, and reuse that definition in the definition of `Extension`.

Accordingly, I used the following basic approach:

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

Since not every `Primitive` can be an extension value, I did lots of ugly code like this:

```rust
pub fn valid_extension(&self) -> bool {
	match *self {
		Primitive::Time(_) => false,
		Primitive::UInt(_) => false,
		Primitive::Oid(_) => false,
		Primitive::PInt(_) => false,
		Primitive::Id(_) => false,
		_ => true
	}
}
```

and since the names of extension values have special logic, I had to implement a function to address that special casing:

```rust
pub fn extension_name(&self) -> String {
	let s = match *self {
		Primitive::Boolean(v) => "Boolean",
 		Primitive::Int(i) => "Integer",
 		Primitive::Decimal(ref d) => "Decimal",
 		Primitive::String(ref s) => "String",
 		Primitive::Code(ref s) => "Code",
 		Primitive::Uri(ref v) => "Uri",
 		Primitive::Base64(ref s) => "Base64Binary",
 		Primitive::Instant(ref x) => "Instant",
 		Primitive::Date(ref x) => "Date",
 		Primitive::DateTime(ref x) => "DateTime",
 		_  => "Error"
	};
	format!("value{}",s)
}
```

Except for that, most of the code involved creating `Extension`s and very small amounts were involved with serializing to JSON.

There's still more coding to be done, but I have atomic (and probably composite, but I haven't tested yet) `Extension`s wired up without too much fuss, and the rest is a simple matter of testing all the cases.

So we're pretty much done with serialization. Next, I'll publish some more deep thoughts, get path-based searching up and running, and get started with deserialization, for which I'll have to address resource typing and specifications (I have an idea about how to do this, which is why I want to do path based searching first, but who knows if that will pan out).