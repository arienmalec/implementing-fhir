---
layout: post
title: Reflections
---

I've wired up extensions into values and resources, completing the journey. At this point, I've completed construction of, and JSON serialization of, FHIR resources and included components, built from an underlying set of algebraic data types. (I haven't deserialized yet, so there's clearly a lot more to know).

What have I learned along the way:

* Even for a management overhead type with a family, who doesn't code on the daily, in a language that I don't know well, and working on odd hours during stolen nights and weekends, it flat out wasn't hard to build a FHIR implementation from scratch.
* The documentation is designed primarily for FHIR developer users, not implementors. That's generally a great documentation decision, but makes it sometimes confusing for FHIR implementors. At some point, FHIR would benefit from a formal specification
* As I've extensively documented, many of the FHIR representational decisions assume XML as the underlying serialization format. This leads to some odd JSON representations.
* There are some odd corner cases in the FHIR documentation (e.g., treating `Extension`s as types of `Element`s), and a fair amount of inconsistency (most of which I haven't blogged; some examples: the term "element" sometimes means `Element`, sometimes means something with subordinate values; the term "infrastructure" is used inconsistently)

My interim reflections and tentative recommendations to the FHIR team are:

* Consider a formal FHIR specification, that treats FHIR objects as independent data types, with mappings to XML, JSON, etc., with clear definitions of, e.g., element value naming, etc.
* Treat "Extension"s as beta -- to my knowledge, they haven't been extensively tested, and there are a set of alternative approaches (e.g., have a URI scope a set of name, type, value triplets) that may have easier and more natural representations
* Consider removing `id` except from `Resource`s
* Consider limiting `Extension`s to `Resource` objects

To be clear, though, the general experience has been highly positive.