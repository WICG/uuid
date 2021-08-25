# uuid

This is the repository for uuid. You're welcome to
[contribute](CONTRIBUTING.md)!

## Implementation status

 * [Chromium](https://bugs.chromium.org/p/chromium/issues/detail?id=1197594) - Shipping 92.
 * [Gecko](https://bugzilla.mozilla.org/show_bug.cgi?id=1723674) - Work in progress.
 * [WebKit](https://bugs.webkit.org/show_bug.cgi?id=229240) - Implemented, not shipped yet.

### Other runtimes

* [Deno](https://github.com/denoland/deno/pull/10848) - v1.11.
* [Node.js](https://github.com/nodejs/node/pull/36729) - v14.17.0.

## Usage example

```JS
const uuid = crypto.randomUUID();
```
## Motivation

### UUID generation is a common software requirement

The <a href="https://www.npmjs.com/package/uuid">uuid library</a> on npm
currently receives 131,000,000 monthly downloads and is relied on by over
2,600,000 repositories (as of June 2019).

The ubiquitous nature of the `uuid` module demonstrates that UUID generation is a common
requirement for JavaScript software applications, making the functionality a good candidate for the
standard library.
### Developers "re-inventing the wheel" is potentially harmful

Developers who have not been exposed to [RFC4122](https://www.rfc-editor.org/rfc/rfc4122) might naturally opt to invent their own approaches
to UUID generation, potentially using `Math.random()`.

There's an in-depth discussion of why a Cryptographically-Secure-Pseudo-Random-Number-Generator
(CSPRNG) should be used when generating UUIDs. Of primary concern is that, without a high-quality source
of randomness, collisions can frequently occur.

Introducing a UUID standard library, which dictates that a CSPRNG must be used, helps protect developers from security pitfalls.
