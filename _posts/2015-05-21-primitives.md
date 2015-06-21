---
layout: post
title: Primitive Types
---

We said in the [last post](http://arienmalec.github.io/implementing-fhir/2015/05/17/elements/) that `Element`s are either `Atom`s with `Primitive` values or are `Elt`s with a vector of sub-`Element`s. Now it's time to delve into what those `Primitive` values are.

It's worth noting that, rather than defining a set of FHIR types, and mapping those types to XML and JSON, many of the primitives are based on XML Schema, with some cross mapping to JSON. Again, we see XML Schema bleed into FHIR at the margins.

There are a list of basic types, with convenient mappings to the underlying data types of most programming languages:

- `integer` (32 bit)
- `unsignedInt` (unsigned 32 bit int)
- `boolean`
- `string`

There's a highly specific `decimal` type, that requires exact precision. For example, a lab test may be measured to 2 decimal places, and a value might be sent as `0.10`; representing as a native floating point value would cause that value to be read as `0.1`. There are a number of different ways of representing this. The easiest, if you have access to an efficient arbitrary precision math library, is to use that, although that's probably overkill (since you probably shouldn't be doing exact precision math with a reference implementation).

A simple way of handling this is to tag a floating point value with its precision:

```rust
struct Dec {
	raw: String,
	val: Option<f64>,
	precision: Option<usize> 
}
```

Note that I'm storing the raw string value; that enables me to represent the odd textual value being sent in a numeric field; I can either print the value with specified precision, or default to printing the string value:

```rust
impl fmt::Display for Dec {
	fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
		match self.val {
			Some(v) =>  write!(f,"{:.*}",self.precision.unwrap(),v),
			None => f.write_str(&(self.raw))
		}
	}
}
```

This raises a more general question about what to do with values that don't fit the value constraints. I've seen enough headaches with supposed numeric values to be paranoid with this one; is there a more general pattern to consider here? This will come back up when I implement deserialization.

Decimal representation requires extensive unit testing to get the precision right. I tested at a number of interesting boundary conditions: `1.`, `1.0`, `1.0000`, `0.10` (note that this value is one of those without an exact binary representation), `0.0000` as well as invalid strings. It would be useful to have a more standard set of conformance tests here.

Next up are a variety of time representations. Some of these also have representations with precision:

- `instant` (a system time with time zone)
- `datetime` (a date with precision ranging from year to exact time with timezone)
- `date` (a date with precision ranging from year to year/month/day)
- `time` (a time with precision ranging from hour to hh:mm:ss)

All of these values could be represented by appropriate subset profiles of ISO 8601, and `instant` and the full `datetime` conform to RFC 3339 The `date` type is intended for remembered dates (the spec calls these dates used in human communication). For example: "When were you last immunized?" "Uh, 2003, I think."

I'm not sure why the `datetime` type exists. It can allow any value between "2003" and "May 1st, 2003, at 4:35:34 PM Pacific".  It's possible that it's intended for values that could either be human remembered, or human recorded (whereas `date` is only for human remembered?). For example, date of immunization could either be recorded by clinical staff on administration, or be recorded from human memory.

I implemented this type with a `struct` that can accommodate either partial date parts or a full `datetime`.

```rust
pub struct VarDate {
	y: Option<i32>,
	m: Option<u32>,
	d: Option<u32>,
	dt: Option<DateTime<FixedOffset>> // also need to handle zulu time
}
```

An alternative representation is to store a full `datetime` value with a precision tag, and use that precision tag to mask display appropriately. That representation can handle time as well (by masking all but the time parts), but requires additional complexity on parse (you need to pad input values with fake `datetime` parts to construct the full value).

For all these types, we need to parse the XML Schema defined seven element datetime format. Representational issues abound here: it's nice to deal with library date handling routines. This is trivial if you have XML Schema parsing available, but kind of a pain if you don't. You probably also want to convert to native datetime values (but need to be aware that you can't always do so).

Again, due to the precision issues, all of these need extensive unit tests.

Then there are a set of utility primitives that are specializations of more basic primitives, or special purpose primitives:

- `positiveInt` (unsigned 32 bit, != 0)
- `uri` (which are both URLs and URNs)
- `base64Binary` (Base64 encoded byte data)
- `code` (Strings used as codes without leading/trailing whitespace)
- `oid` (URI representation of OID - i.e., a URN whose namespace is "oid")
- `id` (constrained strings used as resource identifiers or XML ids)

These are all represented either by a framework type (e.g., URI), or a specialization of a base type. Like `Decimal` and the date/time values, the interesting programming work here is not representation and efficient serialization/deserialization, but validation. For example, `uri`, `oid`, `base64Binary`, `id`, and `code` are all trivially represented as strings; this issue is whether that string conforms to the constraints for the type.

As a whole, FHIR primitives occupy an intermediate role between type serialization specifications, like JSON, ProtoBuf, and XML Schema, and the underlying model types. The underlying design issue is when to place the constraint on the primitive type, and when to place the constraint at the level of the model. There's a bit of type over proliferation here, for my tastes. (Do we need, e.g., an `oid` type as a specialization of the `uri`? HL7 has an odd fascination with OIDs, but I'd prefer to let `oid`s be quiet subtypes of `uri`, and let `uri`s do the hard labor, as they do in other domains. Likewise, I'm not sure that positive integers needs to be a primitive type concern, as that constraint makes it harder to map to JSON and programming language representations.)

I've published [an interim representation of Primitives](https://github.com/arienmalec/fhir-rust/tree/master/src/primitive) on the GitHub repository. As I noted above, I've chosen to represent the Primitive types with all the constraints required, but when we implement deserialization, it's possible that we'll need a different approach to validation that accommodates validation failures. Again, in interoperability, with a good spec and good will, writing correctly is easy but reading robustly is hard.

Next, we'll look at the JSON representation, which will teach us that we have some missing bits to understand to properly serialize JSON given our `Element` representation.