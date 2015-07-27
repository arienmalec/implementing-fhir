---
layout: post
title: FHIR data types
---

We are now in a better position to define somewhat formally define the underlying FHIR data types. That is to say, not the "on the wire" representational types, but the types of FHIR objects themselves (as, for instance, they might be represented in FHIR implementation).

## Elements

Let's start with the primitive FHIR item, the `Element` (note: please do not confuse the FHIR `Element` with an `XML` element). As we've noted, an `Element` is a name/value pair.

### Element Names

An `Element` `name` is a non-empty sequence of characters. `name`s must not start with the '_' character. Certain `name`s are reserved, including 'id', and 'extension'.

What values are allowed for a `name`? Based on the `name`s of existing documented FHIR `Element`s, we might surmise that `name`s are limited to ASCII characters ('a'-'z','A'-'Z') with upper case characters used as word separators (e.g., "birthDate"), but this is not documented. Based on XML encoding semantics, we can reasonably surmise that `name`s must at least conform to [XML `Name` production rules](http://www.w3.org/TR/REC-xml/#NT-Name). This is a wide range of possible name restrictions.

### Element Values

An `Element` may contain either a single value or list of values all of the same type.

A FHIR value is one of the following:

1. Atomic primitive
2. A set (that is, unique by name) of sub-elements
3. A union value (set of possible type values)

Values are additionally associated with an optional single `id` and an optional bag (that is, allows recurring values by `extension` URI and ordering is not significant) of `extension`s. (That is, the "value" is a 3-tuple, with a value slot, an optional `id` slot, and an `extension` bag slot).

This definition is deliberately recursive: FHIR elements (and resources) define a directed acyclic graph, terminating in leaf nodes which are atomic primitives.

### Value Types

FHIR values are typed. The value type constrains the domain of possible values and value cardinality that can provided for the element value. For example, an `Element` named `foo` may be value type constrained as an atomic primitive `boolean` value, and instances named `foo` must therefore either be `true` or `false`.

#### Primitive type definitions

Atomic primitive value instances are constrained to the domain value of their primitive value type. Primitive value domains are as [documented in the current FHIR specification](https://hl7-fhir.github.io/datatypes.html#1.19.0.1).

#### Composite type definitions

Sub-element typed instances are constrained to a composite value type definition. A composite type definition defines the possible names and cardinality of sub-elements and the type definitions of those sub-elements (recursively until all sub-elements terminate in atomic primitives).

For example a composite type of `Foo` constrains a set of sub-elements `bar`, `baz` and `quux`, which are further constrained to types (which may also be composite typed).

#### Union type definitions

Values may be typed to a tagged union type. A tagged union defines a set of types which the value can possibly occupy. Each value instance must be typed to a single one of the defined union types. Tagged union types may mix primitive and composite types. For example, a union value type of `Bar` may include `boolean` and the `Foo` composite type defined above. FHIR union types are tagged unions; which is to say that instances must be constrained to a single identified type.

Elements with Union value types can not be of multiple cardinality (this is currently undocumented, but confirmed with the FHIR maintainers).

The FHIR documentation calls union types variously choice types or polymorphic types.

#### Cardinality and Optionality

FHIR value definitions include cardinality and optionality constraints. Cardinality defines if values may repeat (that is, is defined as a list of values), or if only a singular value is allowed; optionality defines if an value may be missing. The combination of optionality and cardinality defines four constraints, represented \[0..1\], \[1..1\], \[0..\*\], \[1..\*\], where the first `0` or `1` define optional or required, and the second `1` or `*` define singular versus repeating.

FHIR values are never serialized empty; optional elements are simply elided from serialization representation. FHIR implementation should follow these semantics. If an implementation allows `null` values it MUST treat `null`s as equivalent to "missing" or "not provided". (It would be better, however for an implementation to not allow missing values to be accessed, and provide, e.g., `is_present` or `is_missing` or equivalent methods to check for the presence of element values).

## Differences between this and the official FHIR specification

Note that this definition of `Element` values differs considerably from the base FHIR specification. That documentation more closely ties `Element`s to XML elements (and therefore assumes the definition of names and some of the underlying structure).

The benefit of defining `Element`s and their values the way we are here is that, by aligning `id` and `extension` with values, and by specifying that sub-elements must be a set (unique by name), we more closely align the FHIR data types to those of most implementation programming languages (which do not, for instance, allow multiple object instance variables of the same name, in the way that XML allows multiple element tags of the same name at the same level).