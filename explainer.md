## Explainer

### Authors

- Benjamin Coe ([@bcoe](https://github.com/bcoe))
- Robert Kieffer ([@broofa](https://github.com/broofa))
- Christoph Tavan ([@ctavan](https://github.com/ctavan))

### Participate

* [issue tracker](https://github.com/WICG/uuid/issues).
* [contributing guide](https://github.com/WICG/uuid/blob/gh-pages/CONTRIBUTING.md)

### Introduction

We propose adding the `randomUUID()` method to the `crypto` interface. This method
provides an API for generating RFC 4122 identifiers. Initially, the only
version of UUID supported will be the [version 4 "Algorithm for Creating a UUID from Truly Random or Pseudo-Random Numbers"](https://tools.ietf.org/html/rfc4122#section-4.4).

#### Motivation

##### UUID generation is an extremely common software requirement

The [`uuid` module](https://www.npmjs.com/package/uuid) on npm currently receives some
[235,000,000 monthly downloads](https://npm-stat.com/charts.html?package=uuid) and is relied on by over 8,200,000 repositories (as of April 2021).

The ubiquitous nature of the `uuid` module demonstrates that UUID generation is a common
requirement for JavaScript software applications, making the functionality a good candidate for standardization.

##### Developers "re-inventing the wheel" is potentially harmful

Developers who have not been exposed to RFC 4122 might naturally opt to invent their own approaches to UUID generation, potentially using `Math.random()` (as explained in [TIFU by using `Math.random()`][tifu], using non-Cryptographically-Secure-Pseudo-Random-Number-Generator, such
as `Math.random()`, may lead to collisions).

Standardizing a UUID method, which dictates that a CSPRNG must be used, helps protect
developers from security pitfalls.

### Goals

Provide a straightforward and secure mechanism for generating UUIDs that 
addresses developers' common needs (_identifiers for generated pages, identifiers for sessions, identifiers for generated HTML elements, database keys, etc._ ), while
minimizing the amount of domain knowledge a developer requires to discover and
use the feature.

### Proposed API (addition to crypto interface)

```js
const uuid = crypto.randomUUID(); // "52e6953d-edbe-4953-be2e-65ed3836b2f0"
```

#### Considered Alternative (exposing method globally)

```js
const uuid = randomUUID(); // "42e6953d-edbe-4953-be2e-65ed3836b2f0"
```

The primary reason that we are advocating that `randomUUID()` be added to
the `crypto` interface, rather than `global`, is the important requirement
that UUIDs be generated using a CSPRNG. Inclusion in the `crypto` namespace
helps emphasize this requirement.

### Detailed design discussion

#### Why expose only the v4 version at initially?

While there is utility in other UUID versions, we are advocating starting with a minimal API
surface that supports a large percentage of users _(the string representation of version 4 UUIDs)._

If research and/or user feedback later indicates that additional functionality, such as versions 1, 3, and 5 UUIDs, would add value, this proposal does not preclude these additions.

#### Why call the method `randomUUID()` and not something like `uuidV4()`?

As pointed out [in the disucssion](https://github.com/tc39/proposal-uuid/issues/3#issuecomment-544173041) `v4`
UUIDs have the maximum amount of entropy possible for a valid UUID as defined in [IETF RFC
4122][rfc-4122].

UUIDs defined in [IETF RFC 4122][rfc-4122] are 128 bit numbers that follow a specific byte layout. All of them contain a "version" field comprising 4 bits and a "variant" field comprising 2 bits, meaning that 6 out of 128 bits are reserved for meta information.

Since `v4` UUIDs are defined to have all remaining 122 bits set to random values, there cannot be another UUID version that would contain more randomness.

While any name involving `v4` requires a rather deep understanding of the intricate meaning of the term "version" in the context of the UUID spec, the term `randomUUID()` appears to be much more descriptive for `v4` UUIDs.

### How do other languages/libraries deal with UUIDs?

Some other languages/libraries use the term "random" to describe version 4 UUIDs as well
([go](https://godoc.org/github.com/google/uuid#NewRandom),
[Java](<https://docs.oracle.com/javase/10/docs/api/java/util/UUID.html#randomUUID()>),
[C++ Boost](https://www.boost.org/doc/libs/1_71_0/boost/uuid/random_generator.hpp)).

Apart from that, UUID adoption across other languages/libraries seems to be rather inconsistent:

- [deno](https://github.com/denoland/deno/tree/master/std/uuid) added UUID to their standard
  library, leaving out `v3`. The code for UUID creation is essentially copied from the
  [`uuid` npm module](https://github.com/uuidjs/uuid), hence method naming follows the `vX` scheme.
- [Java](https://docs.oracle.com/javase/10/docs/api/java/util/UUID.html) provides methods for
  generating
  `v3`([`UUID.nameUUIDFromBytes()`](<https://docs.oracle.com/javase/10/docs/api/java/util/UUID.html#nameUUIDFromBytes(byte%5B%5D)>))
  and `v4`
  ([`UUID.randomUUID()`](<https://docs.oracle.com/javase/10/docs/api/java/util/UUID.html#randomUUID()>))
  UUIDs but not `v1` or `v5`. It would be interesting to investigate further as to why these
  algorithms were chosen, given that on the one hand time-based UUIDs (`v1`) appear to have much
  broader use than name-based (`v3`/`v5`) UUIDs and that on the other hand for name-based UUIDs the
  [RFC already recommends `v5` over `v3`](https://tools.ietf.org/html/rfc4122#section-4.3).
- [C++ Boost](https://www.boost.org/doc/libs/1_71_0/libs/uuid/doc/uuid.html#boost/uuid/name_generator.hpp)
  defaults to `v5` over `v3` for name-based UUIDs but in its implementation anticipates that `v5`
  (which uses SHA-1 for hashing) will be followed up by a newer name-based UUID version which will
  use a different hashing algorithm ("In anticipation of a new RFC for uuid arriving…").
- [Google's implementation for go](https://godoc.org/github.com/google/uuid#NewUUID) has chosen
  `v1` to be the "default" export whose generator method is called `NewUUID()`, whereas the other
  exposed methods have names closer to the abstraction we propose: `NewRandom()` for `v4`,
  `NewMD5()` for `v3`, `NewSHA1()` for `v5`.
- [Python](https://docs.python.org/3/library/uuid.html) provides methods for generating UUIDs named
  after the version for all 4 versions (`uuid.uuid1()`, `uuid.uuid3()`, `uuid.uuid4()` and
  `uuid.uuid5()`) plus a `UUID` class to represent UUIDs and transform them into various
  representations.
- [Rust](https://docs.rs/uuid/latest/uuid/) provides methods for generating UUIDs named after the
  version for all 4 versions (`Uuid::new_v1()`, `Uuid::new_v3()`, `Uuid::new_v4()` and
  `Uuid::new_v5()`) as static members of a `Uuid` class which is used to represent UUIDs and
  transform them into various representations.

### Stakeholder Feedback

_Section to come._

### References & acknowledgements

Many thanks for valuable feedback and advice from:

* [Daniel Ehrenberg](https://github.com/littledan).
* [Domenic Denicola](https://github.com/domenic).
* [Jan Olaf Krems](https://github.com/jkrems).
* [Kevin Gibbons](https://github.com/bakkot).
* [Ron Buckton](https://github.com/rbuckton).

Along with many other folks who've helped us navigate the standardization
process for `randomUUID`.

[rfc-4122]: https://tools.ietf.org/html/rfc4122
[tifu]: https://medium.com/@betable/tifu-by-using-math-random-f1c308c4fd9d
