---
layout: post
title: JSON Serialization
---

Now that we have a working implementation of `Element`s and `Primitive`s, let's working on serializing `Elements` to JSON. The [documentation for the JSON format](https://hl7-fhir.github.io/json.html) needs some revision, but the basics are clear.

Given our `Element` definition, we'll implement a string representation to JSON. This will be pretty easy, because it's mostly what we already implemented, and now that `Primitive` values know how to represent themselves, it will mostly work. We'll get to why "mostly" in just a bit.

Atomic `Primitive` values have [straightforward representations](https://hl7-fhir.github.io/json.html#1.17.2.2), although note the differences in which values are quoted and not quoted. In general, the convention is:

```json
"elt_name1": bare_value,
"elt_name2": "quoted value"
```

Where bare value types are those that map to JSON base types (boolean and numerics). Oddly, despite the care to which the FHIR spec devotes to making decimal types exact precision, decimal values map JSON numbers.

This causes a problem for us, because native Rust JSON serialization doesn't allow me to override numeric representation. We also originally chose to represent `decimal` with a string alternative, but that's not going to work, because strings and numeric values are represented differently in JSON. I could write my own JSON serialization framework, but that doesn't let me take advantage of the efficiency built into the native Rust framework.

I'm going to change my `decimal` representation to error on parse if the underlying value isn't numeric, and use the built in JSON framework. That will cost in terms of preserving precision -- the only way around that, as far as I can see, is for FHIR to change the `decimal` JSON representation to `String`. I expect that will be a problem for most native XML Schema decoders as well. I'll also use the native JSON serializer for now.

The implementation of `primitive` serialization is pretty straightforward then:

```rust
impl ToJson for Primitive {
	fn to_json(&self) -> Json {
		match *self {
			Primitive::Boolean(v) => Json::Boolean(v),
	 		Primitive::Int(i) => Json::I64(i as i64),
	 		Primitive::UInt(i) => Json::U64(i as u64),
	 		Primitive::PInt(i) => Json::U64(i as u64),
	 		Primitive::Decimal(ref d) => Json::F64(d.val),
	 		Primitive::String(ref s) => Json::String(s.clone()),
	 		Primitive::Id(ref v) => Json::String(v.to_string()),
	 		Primitive::Uri(ref v) => Json::String(v.to_string()),
	 		Primitive::Oid(ref v) => Json::String(v.to_string()),
	 		Primitive::Base64(ref v) => Json::String(v.to_string()),
	 		Primitive::Instant(ref v) => Json::String(v.to_string()),
	 		Primitive::Date(ref v) => Json::String(v.to_string()),
	 		Primitive::DateTime(ref v) => Json::String(v.to_string()),
	 		Primitive::Time(ref v) => Json::String(v.to_string()),
	 	}
	}
}
```

JSON `Object`s (and Rust's implementations of them) are just maps from names to values, so the `Element` implementation of serialization is nearly as simple (once we know where the object comes from):

```rust
ElementType::Atom(ref v) => {
	object.insert(self.name.clone(),v.to_json());
}
```

We are leaving the `Elt` case alone until we figure out what to do with multiple cardinality. As I noted before, we need to consider the defined cardinality of each element. Earlier I noted that Atomic primitives are relatively simple. FHIR values can have cardinality > 1, however. This maps to a JSON `array` of values, all of which are the same time. That is, if `foo` is `boolean` valued and multiple cardinality, the JSON representation of `foo` is going to be:

```json
"foo": [true, false, /* ... */ true],
```

This means our previous definition of `Element`s was wrong:

> As we previously discovered, the `value` of an `Element` is either a primitive atomic value, or a sub-elements bag of `Element`.

What we really should have said is:

The `value` of an `Element` is one of:

* A primitive atomic value
* A list of `Element` values, all of the same type
* A set of sub-`Element`s (it's a set this time, not a bag, because we've moved the repeating elements to the list type)

(It's funny that FHIR serialization is defined in terms of XML, because this definition is a perfect map to JSON. It's still incomplete, however, because we haven't deal with `id`s and `extension`s).

Given that definition, again the definition of `ElementType` and `to_json` are reasonably straightforward:

```rust
pub enum ElementType {
	Atom(Primitive),
	List(Vec<ElementType>),
	Elt(Vec<Element>)
}

impl ToJson for ElementType {
	fn to_json(&self) -> Json {
		match *self {
			ElementType::Atom(ref v) => v.to_json(),
			ElementType::List(ref v) => {
				Json::Array(v.iter().map(|e| e.to_json()).collect::<Vec<Json>>())
			},
			ElementType::Elt(ref v) => {
				let mut o: BTreeMap<String,Json> = BTreeMap::new();
				for e in v.iter() {
					o.insert(e.name.clone(), e.value.to_json());
				}
				Json::Object(o)
			}
		}
	}
}
```

The definition for `ElementType` maps nicely to our definition; in the JSON serialization, we simply map those definitions to their implementations:

* For atomic primitive values, we generate the JSON for the atom (using the code defined above for `primitive` value types)
* For lists, we construct a JSON array with our values serialized to JSON
* For subelements, we construct a JSON Object, with key value pairs, where each key is the name of the subselement, and the value is the (recursive) JSONized value

Given all that, we define for the moment `Element`'s implementation of `to_json` as follows:

```rust
impl ToJson for Element {
	// Will only be valid JSON if self.value is an element list.
	fn to_json(&self) -> Json {
		self.value.to_json()
	}
}
```

I'd normally put the contents of the top level `Element` in an anonymous object, and indeed, that's what FHIR does, but with some special rules for `Resource`s, which are the normal top level serialized FHIR type.

My TODO list right now looks like this:

* implement `id` and `extension` and serialize them
* implement `Resource` and special serialization rules
* lots of refactoring and convenience functions and moar unit tests
* figure out what to do for deserialization
* XML :-(

I might write up some interim lessons learned along the way.