---
layout: post
title: Resources
---

I've decided to tackle `Resource`s before tackling `id` and `extension` because there are common features between `Element`s and `Resource`s. This is somewhat confusing: The `Resource` type is [described as having no base type](https://hl7-fhir.github.io/resource.html), but like the `Element`, has a name and a set of values (defined by the `Resource` base content, and the respective `Resource` definition and profile). The `Resource` has an `id` and [the `DomainResource` also has a set of extensions](https://hl7-fhir.github.io/resource.html). In addition, each `Resource` has a set of sub-elements, some of which are incorporated from the `Resource` and `DomainResource` model, and some of which are defined by the individual resource domain models and profiles.

In accordance with my basic philosophy of "do the simplest thing that could possibly work", the easiest definition for `Resource` is the following:

```rust
pub struct Resource {
	pub name: String,
	pub elts: Vec<Element>
}
```

Unlike the `Element` case, we can handle `id` and `extension` as regular elements (although `Extension`s are a bit strange, as we'll see when we get to them). We'll see why when we get to them.

Given this definition, serializing is pretty trivial:

```rust
impl ToJson for Resource {
	fn to_json(&self) -> Json {
		let mut o: BTreeMap<String,Json> = BTreeMap::new();
		o.insert("resourceType".to_string(),Json::String(self.name.clone()));
		for e in self.elts.iter() {
			o.insert(e.name.clone(), e.value.to_json());
		}
		Json::Object(o)
	}
}
```

There's a `resourceType` name value pair which holds the name of the `Resource`, and then we just serialize each sub-element according to [the serialization rules we previously defined](http://arienmalec.github.io/implementing-fhir/2015/05/25/json-serialize/).

And that's kinda it.

Next up, `id` and `extension` at the `Element` level, and we'll have fully functional JSON serialization.