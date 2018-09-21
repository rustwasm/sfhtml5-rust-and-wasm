name: none
layout: true

---

name: inverse
layout: true
class: left, middle, inverse

.footnote[[bit.ly/rust-and-wasm](https://bit.ly/rust-and-wasm)]

---

name: normal
layout: true
class: left, middle

.footnote[[bit.ly/rust-and-wasm](https://bit.ly/rust-and-wasm)]

---

class: middle, center

# Rust 🦀 and Wasm 🕸

#### Nick Fitzgerald
#### [@fitzgen](https://twitter.com/fitzgen) | [@rustwasm](https://twitter.com/rustwasm)

![rustwasmjs](public/img/rustwasmjs.png)

.footnote[[bit.ly/rust-and-wasm](https://bit.ly/rust-and-wasm)]

???

* rust+wasm WG

---

template: inverse

# Use Rust-generated WebAssembly to speed up your performance-sensitive JavaScript

???

* This is the ideal we're aiming for, if not the reality in all cases yet.

---

class: center

# Do <u>NOT</u> Rewrite &mdash; Integrate
<br/>
<img src="./public/img/lin_rustjs.png"/>

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
extern "C" {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet() {
    alert("Hello, World!");
}
```

???

* dive into hello world

---

```rust
*use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern "C" {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet() {
    alert("Hello, World!");
}
```

???

* `use` is bringing `wasm-bindgen`'s common functionality into scope

---

```rust
use wasm_bindgen::prelude::*;

*#[wasm_bindgen]
*extern "C" {
*   fn alert(s: &str);
*}

#[wasm_bindgen]
pub fn greet() {
    alert("Hello, World!");
}
```

???

* importing the `window.alert` function
* `#[wasm_bindgen]` on an `extern` block creates imports at the `.wasm` level

---

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern "C" {
    fn alert(s: &str);
}

*#[wasm_bindgen]
pub fn greet() {
    alert("Hello, World!");
}
```

???

* exporting a `greet` function
  * `#[wasm_bindgen]` on a `pub` function makes it an export in the `.wasm`
    binary

---

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern "C" {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet() {
*   alert("Hello, World!");
}
```

???

* calling imported `alert` function like we would any normal Rust function!

---

```js
import { greet } from "./hello_world";

greet();
```

???

* Rust-generated wasm is consumable as an ES module
* just looking at this, we can't tell if the `"./hello_world"` module is JS or
  wasm
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
  out.rest = rest;
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
#### ✔ Allocation and deallocation
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
* relative standard deviations fell: samples are much tighter together now
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

## Plays Well With Others
<br/>
#### ✔ Integrates with bundlers (e.g. Webpack)
#### ✔ Publish to NPM
<br/>
<video style="margin: 0 auto; display: block" src="./public/img/lion-dog-kiss.mp4" autoplay="" loop="" playsinline=""></video>

???

* but the source-map experience also showed us what we needed to work on some
  more: toolchain integration
* source-map work was done in late december/january last year, have been doing a
  lot of toolchain integration since and I think it shows

---

## Small `.wasm` Sizes
<br/>
#### *Example:* Using `document.querySelectorAll` doesn't entrench code for `window.alert`
<br/>
> What you don’t use, you don’t pay for.

???

* like tree shaking in JS
* some of this at `lld` level
* but carried through the rest of ecosystem as a design constraint as well
  * rustc
  * wasm-bindgen
  * js-sys
  * web-sys
* crucial for integration with JS:
  * it doesn't make sense to port one function / module to wasm if that means:
      * you have to include a whole 'nother GC and language runtime in the wasm
      * your download size explodes
      * and page loads take forever

---

class: middle, center

# Using Rust and Wasm
<br/>
<video src="./public/img/dog-lifting.mp4" autoplay="" loop="" playsinline=""></video>

---

# `wasm-pack`
<br/>
### 👷 Building `.wasm`
### 🎁 Creating and publishing NPM packages
### <img src="./public/img/female-scientist.png" style="max-height: 1em; max-width: 1em"/> Testing in headless browsers
### 📝 All your last-mile tasks

---

# `wasm-bindgen`
<br/>
## Wasm<sup>💬 🗨</sup>JavaScript

???

* facilitates communication between wasm and JS
* "bindgen" = "bindings generator"
* exported wasm functions only take `i32`/`i64`/`f32`/`f64`
  * but we want to pass strings, objects, DOM nodes, etc
  * wasm-bindgen enables that, all with the zero-overhead abstraction principle
    we keep revisiting

---

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern "C" {
    fn alert(s: &str);
}

/// Make a greeting!
#[wasm_bindgen]
pub fn greet() {
    alert("Hello, World!");
}
```

???

* take deeper look at hello world

---

class: center

# `wasm-pack build`
<br/>
![](./public/img/wasm-pack-build-pipeline.png)

???

* `wasm-pack build` generates:
  * a `.wasm` file, containing your Rust compiled to wasm
  * a JS file that provides nice JS APIs to your Rust code
  * a TypeScript interface definition file for type checking your JS/TS and
    getting autocompletions
  * a `package.json` for integrating with JS tooling, bundlers, publishing to
    NPM, etc...

---

### Generated Bindings for `greet`

```js
// ...

export function __wbg_alert_2c86be282863e459(arg0, arg1) {
    let varg0 = getStringFromWasm(arg0, arg1);
    alert(varg0);
}

// ...
```

???

* generated JS file has glue for wrapping imported functions and translating
  their arguments into something that wasm-bindgen can understand

---

### Generated Bindings for `greet`

```js
// ...

export function __wbg_alert_2c86be282863e459(arg0, arg1) {
*   let varg0 = getStringFromWasm(arg0, arg1);
    alert(varg0);
}

// ...
```

???

* read a Rust string from wasm memory

---

### Generated Bindings for `greet`

```js
// ...

export function __wbg_alert_2c86be282863e459(arg0, arg1) {
    let varg0 = getStringFromWasm(arg0, arg1);
*   alert(varg0);
}

// ...
```

???

* then call the actual `window.alert` function

---

### Generated Bindings for `greet`

```js
import * as wasm from './hello_world_bg';

// ...

/**
 * Make a greeting!
 * @returns {void}
 */
export function greet() {
    return wasm.greet();
}
```

???

* the generated JS also wraps the raw wasm exports and provides a nice JS
  interface to them

---

### Generated Bindings for `greet`

```js
*import * as wasm from './hello_world_bg';

// ...

/**
 * Make a greeting!
 * @returns {void}
 */
export function greet() {
    return wasm.greet();
}
```

???

* this is importing the actual `.wasm` file as an ES module

---

### Generated Bindings for `greet`

```js
import * as wasm from './hello_world_bg';

// ...

/**
 * Make a greeting!
 * @returns {void}
 */
export function greet() {
*   return wasm.greet();
}
```

???

* the generated JS also wraps the raw wasm exports and provides a nice JS
  interface to them
* it converts and wraps arguments from JS into something that wasm can
  understand
  *  some combination of `i32`/`i64`/`f32`/`f64`
* we just don't have any arguments here yet, so it doesn't do anything
* what if we wanted to write our greeting into a DOM node of the user's choice?

---

### Get a DOM node, set its `textContent`
<br/>

```rust
use wasm_bindgen::prelude::*;
use web_sys::Node;

#[wasm_bindgen]
pub fn greet2(node: &Node) {
   node.set_text_content(Some("Hello, World!"));
}
```

???

* this version of hello world is putting its greeting into some DOM node's text
  content
* no more importing common Web functions by hand
  * using `web_sys` instead

---

### Get a DOM node, set its `textContent`
<br/>

```rust
use wasm_bindgen::prelude::*;
*use web_sys::Node;

#[wasm_bindgen]
*pub fn greet2(node: &Node) {
   node.set_text_content(Some("Hello, World!"));
}
```

???

* now we are taking a DOM node parameter

---

### Get a DOM node, set its `textContent`
<br/>

```rust
use wasm_bindgen::prelude::*;
use web_sys::Node;

#[wasm_bindgen]
pub fn greet2(node: &Node) {
*  node.set_text_content(Some("Hello, World!"));
}
```

???

* and setting its `textContent` to "Hello, World!"

---

### Get a DOM node, set its `textContent`
<br/>

```js
import { greet2 } from "./hello_world";

greet2(document.body);
```

???

* to call this version of the function, we need to pass a DOM node
* we don't have to do anything fancy to pass the DOM node to wasm, the generated
  JS takes care of that

---

class: center

### `wasm-pack build`

<video src="./public/img/excited-corgi.mp4" autoplay="" loop="" playsinline=""></video>

---

### Generated Bindings for `greet2`

```js
import * as wasm from './hello_world_bg';

// ...

export function greet2(arg0) {
    try {
        return wasm.greet2(addBorrowedObject(arg0));
    } finally {
        stack.pop();
    }
}
```

???

* Not showing import wrappers
  * wrapping the `Node.prototype.textContent` setter is largely the same as
    wrapping `window.alert`

---

### Generated Bindings for `greet2`

```js
import * as wasm from './hello_world_bg';

// ...

export function greet2(arg0) {
    try {
*       return wasm.greet2(addBorrowedObject(arg0));
    } finally {
*       stack.pop();
    }
}
```

???

* `addBorrowedObject` pushes an object onto the stack and returns its index
* `stack.pop()` removes it from the stack after the call is done

---

class: center

### `greet2` took a <u>borrowed</u> DOM node...

--

<br/>
### What if we took <u>ownership</u> the DOM node?

---

### Taking ownership of DOM nodes

```rust
thread_local! {
    static ALL_NODES: RefCell<Vec<web_sys::Node>> =
        Default::default();
}

#[wasm_bindgen]
pub fn greet3(node: web_sys::Node) {
    ALL_NODES.with(|all_nodes| {
        let mut all_nodes = all_nodes.borrow_mut();
        all_nodes.push(node);

        for n in all_nodes.iter() {
            n.set_text_content(Some("Hello, World!"));
        }
    });
}
```

???

* now we are taking ownership of DOM nodes, instead of borrowing them
* that means we can do tricky things like keep a list of every DOM node we were
  ever given
* and write our greeting into *all* of their `textContent`s

---

### Taking ownership of DOM nodes

```rust
thread_local! {
    static ALL_NODES: RefCell<Vec<web_sys::Node>> =
        Default::default();
}

#[wasm_bindgen]
*pub fn greet3(node: web_sys::Node) {
    ALL_NODES.with(|all_nodes| {
        let mut all_nodes = all_nodes.borrow_mut();
        all_nodes.push(node);

        for n in all_nodes.iter() {
            n.set_text_content(Some("Hello, World!"));
        }
    });
}
```

???

* no more `&`; that's how you know we are taking ownership of the argument

---

### Taking ownership of DOM nodes

```rust
*thread_local! {
*   static ALL_NODES: RefCell<Vec<web_sys::Node>> =
*       Default::default();
*}

#[wasm_bindgen]
pub fn greet3(node: web_sys::Node) {
    ALL_NODES.with(|all_nodes| {
        let mut all_nodes = all_nodes.borrow_mut();
        all_nodes.push(node);

        for n in all_nodes.iter() {
            n.set_text_content(Some("Hello, World!"));
        }
    });
}
```

???

* we have a thread-local vector we are saving the nodes we are given in

---

### Taking ownership of DOM nodes

```rust
thread_local! {
    static ALL_NODES: RefCell<Vec<web_sys::Node>> =
        Default::default();
}

#[wasm_bindgen]
pub fn greet3(node: web_sys::Node) {
    ALL_NODES.with(|all_nodes| {
        let mut all_nodes = all_nodes.borrow_mut();
*       all_nodes.push(node);

        for n in all_nodes.iter() {
            n.set_text_content(Some("Hello, World!"));
        }
    });
}
```

???

* we add this most recently given node

---

### Taking ownership of DOM nodes

```rust
thread_local! {
    static ALL_NODES: RefCell<Vec<web_sys::Node>> =
        Default::default();
}

#[wasm_bindgen]
pub fn greet3(node: web_sys::Node) {
    ALL_NODES.with(|all_nodes| {
        let mut all_nodes = all_nodes.borrow_mut();
        all_nodes.push(node);

*       for n in all_nodes.iter() {
*           n.set_text_content(Some("Hello, World!"));
*       }
    });
}
```

???

* and finally we set the `textContent` for all of the nodes we've ever been
  given

---

class: center

### `wasm-pack build`

<video src="./public/img/breakdance.mp4" autoplay="" loop="" playsinline=""></video>

???

* run `wasm-pack build` again to regenerate the bindings

---

### Generated Bindings for `greet3`

```js
/// ...

export function greet3(arg0) {
    return wasm.greet3(addHeapObject(arg0));
}

/// ...
```

???

* how do the generated bindings for `greet3` differ from `greet2`?
* not doing a `stack.pop()` at the end anymore

---

### Generated Bindings for `greet3`

```js
/// ...

export function greet3(arg0) {
*   return wasm.greet3(addHeapObject(arg0));
}

/// ...
```

???

* instead of adding borrowed objects, we are adding heap objects
* these objects have dynamic lifetime, which requires more bookkeeping than
  stack objects, and are freed in the `web_sys::Node` destructor
* `wasm-bindgen` will look at ownership/borrowing in function signatures and
  generate more efficient bindings code when it is safe to do so

---

template: inverse
class: center

## Rust's ownership model helps us optimize Wasm ↔ JavaScript
<br/>
<video class="inverse" src="./public/img/portlandia-mindblown.mp4" autoplay="" loop="" playsinline=""></video>

---

### Exposing Rust `struct`s to JavaScript

```rust
#[wasm_bindgen]
pub struct StreamingStats {
    count: u32,
    sum: f64,
}
```

???

* let's take a look at a slightly more involved example: exposing a Rust
  struct+methods to JS
* this example is going to track some statistics in streaming fashion
  * not keeping all samples around, hogging a bunch of memory
      * mean = sum of samples / number of samples
      * can just store
        1. sum of samples and
        2. number of samples
      * and then compute mean without every sample all at the same time
  * maybe useful for an FPS counter or something like that
  * we could also compute variance, stddev, min, max, etc

---

### Exposing Rust `struct`s to JavaScript

```rust
*#[wasm_bindgen]
pub struct StreamingStats {
    count: u32,
    sum: f64,
}
```

???

* make sure struct has `#[wasm_bindgen]` annotation on it

---

### Exposing Rust `struct`s to JavaScript

```rust
#[wasm_bindgen]
*pub struct StreamingStats {
    count: u32,
    sum: f64,
}
```

???

* and that it has `pub` visibility
* that's all that's needed to get an ES class that mirrors the Rust `struct`!

---

### Exposing Methods to JavaScript

```rust
#[wasm_bindgen]
impl StreamingStats {
    // ...
}
```

???

* let's also add a constructor and some methods

---

### Exposing Methods to JavaScript

```rust
*#[wasm_bindgen]
impl StreamingStats {
    // ...
}
```

???

* to expose methods:
  * throw `#[wasm_bindgen]` on an impl block
  * then any `pub` method inside will be exposed to JS as well
      * as a method on the generated ES class

---

### Exposing Methods to JavaScript

```rust
// ...

#[wasm_bindgen]
impl StreamingStats {
*   pub fn add(&mut self, sample: f64) {
*       self.count += 1;
*       self.sum += sample;
*   }

    pub fn mean(&self) -> f64 {
        self.sum / (self.count as f64)
    }

    // ...
}
```

???

* to add a sample:
  * increment the `self.count`
  * add the sample to the rolling sum

---

### Exposing Methods to JavaScript

```rust
// ...

#[wasm_bindgen]
impl StreamingStats {
    pub fn add(&mut self, sample: f64) {
        self.count += 1;
        self.sum += sample;
    }

*   pub fn mean(&self) -> f64 {
*       self.sum / (self.count as f64)
*   }

    // ...
}
```

???

* to get the current mean of all samples seen so far, we divide the rolling sum
  by the number of smaples seen

---

### Exposing a Constructor to JavaScript

```rust
#[wasm_bindgen]
impl StreamingStats {
    // ...

    #[wasm_bindgen(constructor)]
    pub fn new() -> StreamingStats {
        StreamingStats {
            count: 0,
            sum: 0.0,
        }
    }
}
```

???

* also need to let JS create instances
* to add a constructor:

---

### Exposing a Constructor to JavaScript

```rust
#[wasm_bindgen]
impl StreamingStats {
    // ...

    #[wasm_bindgen(constructor)]
*   pub fn new() -> StreamingStats {
        StreamingStats {
            count: 0,
            sum: 0.0,
        }
    }
}
```

???

* static `new` function that returns `Self` type

---

### Exposing a Constructor to JavaScript

```rust
#[wasm_bindgen]
impl StreamingStats {
    // ...

*   #[wasm_bindgen(constructor)]
    pub fn new() -> StreamingStats {
        StreamingStats {
            count: 0,
            sum: 0.0,
        }
    }
}
```

???

* use the `constructor` attribute with `wasm-bindgen`
* this tells `wasm-bindgen` to create a JS constructor instead of a static
  method named `new`

---

class: center

### `wasm-pack build`

<video src="./public/img/qe-excitement.mp4" autoplay="" loop="" playsinline=""></video>

---

### Generated Bindings for `StreamingStats`

```js
import * as wasm from './hello_world_bg';

export class StreamingStats {
    constructor() {
        this.ptr = wasm.streamingstats_new();
    }
    free() {
        const ptr = this.ptr;
        this.ptr = 0;
        wasm.__wbg_streamingstats_free(ptr);
    }
    add(arg0) {
        wasm.streamingstats_add(this.ptr, arg0);
    }
    mean() {
        return wasm.streamingstats_mean(this.ptr);
    }
}
```

???

* I cleaned this up a little bit and hid unnecessary things

---

### Generated Bindings for `StreamingStats`

```js
import * as wasm from './hello_world_bg';

export class StreamingStats {
    constructor() {
        this.ptr = wasm.streamingstats_new();
    }
    free() {
        const ptr = this.ptr;
        this.ptr = 0;
        wasm.__wbg_streamingstats_free(ptr);
    }
*   add(arg0) {
        wasm.streamingstats_add(this.ptr, arg0);
    }
*   mean() {
        return wasm.streamingstats_mean(this.ptr);
    }
}
```

???

* each of the `pub` methods we created have a corresponding method on the ES
  class
* since we are using `f64`, the arguments don't need any special treatment
* same for the constructor

---

### Generated Bindings for `StreamingStats`

```js
import * as wasm from './hello_world_bg';

export class StreamingStats {
*   constructor() {
*       this.ptr = wasm.streamingstats_new();
*   }
    free() {
        const ptr = this.ptr;
        this.ptr = 0;
        wasm.__wbg_streamingstats_free(ptr);
    }
    add(arg0) {
        wasm.streamingstats_add(this.ptr, arg0);
    }
    mean() {
        return wasm.streamingstats_mean(this.ptr);
    }
}
```

???

* and the ES class constructor calls our `StreamingStats::new` constructor
* it saves a pointer into the wasm linear memory where the `StreamingStats`
  instance lives

---

### Generated Bindings for `StreamingStats`

```js
import * as wasm from './hello_world_bg';

export class StreamingStats {
    constructor() {
        this.ptr = wasm.streamingstats_new();
    }
*   free() {
*       const ptr = this.ptr;
*       this.ptr = 0;
*       wasm.__wbg_streamingstats_free(ptr);
*   }
    add(arg0) {
        wasm.streamingstats_add(this.ptr, arg0);
    }
    mean() {
        return wasm.streamingstats_mean(this.ptr);
    }
}
```

???

* there is also a `free` method
* Once we give `StreamingStats` to JS, it is JS's responsibility to manage its
  lifetime
* Rust compiler can't tell when it goes out of scope, like it normally can
* also encounter this with C and C++ compiled to wasm
* can also run into this when writing "C" in JS!

---

### Using `StreamingStats` from JavaScript

```js
import { StreamingStats } from "./streaming_stats";

const stats = new StreamingStats();

for (let i = 0; i < 1000; i++)
  stats.add(Math.random());

console.log(stats.mean());

stats.free();
```

???

* let's take a look at how we use `StreamingStats` from JS

---

### Using `StreamingStats` from JavaScript

```js
*import { StreamingStats } from "./streaming_stats";

const stats = new StreamingStats();

for (let i = 0; i < 1000; i++)
  stats.add(Math.random());

console.log(stats.mean());

stats.free();
```

???

* again, using ES modules to import the `StreamingStats` ES class

---

### Using `StreamingStats` from JavaScript

```js
import { StreamingStats } from "./streaming_stats";

*const stats = new StreamingStats();

for (let i = 0; i < 1000; i++)
  stats.add(Math.random());

console.log(stats.mean());

stats.free();
```

???

* constructing is just like any plain JS class

---

### Using `StreamingStats` from JavaScript

```js
import { StreamingStats } from "./streaming_stats";

const stats = new StreamingStats();

for (let i = 0; i < 1000; i++)
* stats.add(Math.random());

*console.log(stats.mean());

stats.free();
```

???

* calling the `add` or `mean` method is just liek any plain JS class as well

---

### Using `StreamingStats` from JavaScript

```js
import { StreamingStats } from "./streaming_stats";

const stats = new StreamingStats();

for (let i = 0; i < 1000; i++)
  stats.add(Math.random());

console.log(stats.mean());

*stats.free();
```

???

* the only different thing is that you have to remember to call `free` when
  you're done with it.

---

class: center

## Managing Lifetimes from JavaScript
<!-- <br/> -->
<!-- #### 🏗 &emsp; Component lifetime hooks -->
<!-- #### 🥑 &emsp; `with` functions -->
<!-- #### 🤸 &emsp; *Future:* GC weak refs and finalizers -->

---

### Component Lifetime Hooks

```js
class MyCustomElement {
  connectedCallback() {
    this.stats = new StreamingStats();
  }

  // ...

  disconnectedCallback() {
    this.stats.free();
  }
}
```

???

* use webcomponents and custom elements component lifetime callbacks
* similar hooks for React components
  * `componentMounted` or whatever

---

### `with` Functions

```js
function withStreamingStats(f) {
  const stats = new StreamingStats();
  try {
    f(stats);
  } finally {
    stats.free();
  }
}
```

???

* similar to Python's `with` statement
* RAII in JS
* can also have `async` function version
* execute the callback `f` and make sure that resource is properly freed at the
  end

---

### `with` Functions

```js
withStreamingStats(stats => {
  for (let i = 0; i < 1000; i++)
    stats.add(Math.random());

  console.log(stats.mean());
});
```

???

* todo

---

### *Future:* GC Weak Refs and Finalizers

---

class: center

# How it's Made

<img src="./public/img/bigbird.gif" style="max-height: 50%; max-width: 50%"/>

---

---

class: center

# Learn more at [rustwasm.github.io/book](https://rustwasm.github.io/book)

---

class: center

# [Join the Rust and Wasm Working Group!](https://github.com/rustwasm/team/blob/master/README.md#get-involved)

<img src="./public/img/wasmwg.png" style="max-width: 50%; max-height: 50%"/>

---

template: inverse
class: center

# THANK YOU!!
<br/>
### Nick Fitzgerald
<br/>
### [@fitzgen](https://twitter.com/fitzgen) | [@rustwasm](https://twitter.com/rustwasm)
