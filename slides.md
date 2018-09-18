name: none
layout: true

---

name: inverse
layout: true
class: left, middle, inverse

.footnote[[bit.ly/TODO](https://bit.ly/TODO)]

---

name: normal
layout: true
class: left, middle

.footnote[[bit.ly/TODO](https://bit.ly/TODO)]

---

class: middle, center

# Rust 🦀 and Wasm 🕸

#### Nick Fitzgerald
#### [@fitzgen](https://twitter.com/fitzgen) | [@rustwasm](https://twitter.com/rustwasm)

![rustwasmjs](public/img/rustwasmjs.png)

.footnote[[bit.ly/TODO](https://bit.ly/TODO)]

???

---

template: inverse

# Use Rust-generated WebAssembly to speed up your performance-sensitive JavaScript

???

* This is the ideal, if not the reality in all cases yet.

---

template: inverse

# Do <u>NOT</u> Rewrite &mdash; Integrate

???

* Corollary:
  * since we are surgically replacing performance-sensitive code paths, the
    other code paths can remain the same
  * which means that Rust+Wasm is *augmenting* your JS
  * not replacing it
* furthermore, you should be able to leverage rust-generated wasm packages from
  npm transparently
  * nothing in your workflow changes
  * you just get faster dependencies

---

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn greet() {
    web_sys::alert_with_message("Hello, World!");
}
```

???

1. `use` is bringing `wasm-bindgen`'s common functionality into scope
2. exporting a `greet` function
   * `#[wasm_bindgen]` on a function makes it an export in the `.wasm` binary
3. `web_sys` crate provides bindings to DOM and Web APIs
4. `web_sys::alert_with_message` is calling the `window.alert` function with a
   message

---

```js
import { greet } from "./hello_world";

greet();
```

???

* Rust-generated wasm is consumable as an ES module
* Can't tell if `"./hello_world"` module is JS or wasm
  * this is the level of transparent, it-just-works integration we aim for

---

# 🗺 Roadmap

<hr/>

### ~~1. 🌍 Hello, World!~~
### 2. 🤔 Why Rust and WebAssembly?
### 3. 🏋 Using Rust and WebAssembly
### 4. 🌭 How the Sausage is Made

---

class: middle, center

<video src="./public/img/why.mp4" autoplay="" loop="" playsinline=""></video>

???

* Those of you following along closely will know that this next section is about
  "why rust and webassembly?"
  * dive into both:
      1. why **rust** and wasm?
      2. why rust and **wasm**?

---

template: none
class: middle, center

![](./public/img/oxidizingsourcemaps.png)

.footnote[[hacks.mozilla.org/2018/01/oxidizing-source-maps-with-rust-and-webassembly/](https://hacks.mozilla.org/2018/01/oxidizing-source-maps-with-rust-and-webassembly/)]

???

* motivating example: speeding up the `source-map` library by integrating rust+wasm
* this experience ended up informing a lot of our WG's efforts
* different direction from emscripten:
  * trying to integrate into JS ecosystem
  * not trying to port native applications to the Web

---

# Idiomatic vs. Fast

#### Idiomatic JavaScript

```js
function decodeVlq(input) {
  // ...
  return { result, rest };
}
```

#### Faster-but-less-idiomatic JavaScript

```js
function decodeVlq(input, out) {
  // ...
  out.result = result;
n  out.rest = rest;
}
```

???

* over the years, the original JS accumulated convoluted code in the name of
  performance
* often found abstraction and idiomatic code at odds with run time performance
* this example: reuse `out` object on every call = no allocation on every call
  * in theory, JITs *could* optimize the allocation away with escape analysis,
    but we found that it was not consistently happening across all JS engines

---

# Idiomatic vs. Fast

* `return -1;` instead of `throw new Error`
* JIT's type inference not recognizing "shapes" it should have
* Custom Quick Sort implementation to allow inlining comparator function

---

template: none
class: middle, swing-crab-bg

## Low-level<br/>Control
<br/>
## High-level<br/>Ergonomics

.footnote[Image credit: [reddit.com/u/ConeCandy](https://www.reddit.com/user/ConeCandy)]

---

## Control Over
<br/>
#### ✔ Memory layout and indirections
#### ✔ Allocations
#### ✔ Monomorphization vs. dynamic dispatch
#### ✔ Inlining

---

<video style="float:right; max-width: 50%" src="./public/img/cake.mp4" autoplay="" loop="" playsinline=""></video>

## Zero-overhead abstractions
<br/>
#### `Result<T, E>`
#### `Iterator<Item = T>`

<br/>

> What you don’t use, you don’t pay for. And further: What you do use, you
> couldn’t hand code any better.

– Bjarne Stroustrup

???

* when making JS fast, end up writing "C" inside JS
  * but without even the abstractions C offers, such as `struct`s and naming things
  * because those abstractions are not without overhead in JS
  * imply allocations, for example
* Rust has "zero-overhead abstractions"
  * Example 1: if we tried to make a `Result` type in JS, it implies heap
    allocation
  * Example 2: a function that is generic over some type, monomorphized it is as
    if you wrote versions specialized to the generic input by hand

---

class: middle, center

<a href="./public/img/pause-at-exception.svg">
  <object data="./public/img/pause-at-exception.svg" type="image/svg+xml"></object>
</a>

???

* time it takes to query a source map for info needed when a debugger pauses at
  an exception the first time
  * lower is better
  * red = original, pure-JS implementation
  * blue = new hybrid implementation with rust+wasm for core, perf-sensitive
    code paths
* straight port led to ~6x faster than original implementation
* further algorithmic improvements got it up to ~11x faster (not reflected in
  this graph)
* still a JS library! just core compute-bound kernel in rust+wasm
* Ultimately:
  * still have to rely on profiling!
  * algorithms are still important!
  * JS performance tuning
      * finicky
      * need to know JS implementation and internals
      * working without abstractions
  * Rust
      * idioms guide us towards performant code
      * don't give up abstraction to get speed
      * Don't have to be JIT wizards to get fast code

---

### Small `.wasm` Sizes
<br/>
### Plays Well With Others
<br/>
### The Amenities You Expect

---
