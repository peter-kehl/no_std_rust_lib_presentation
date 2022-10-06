<!-- The following comment hides this section from being shown by
     https://peter-kehl.github.io/no_std_rust_lib_presentation.
-->
<!-- .slide: data-visibility="hidden" -->
# Slides and alternative navigation

If you are seeing this, consider viewing [presentation
slides](https://peter-kehl.github.io/no_std_rust_lib_presentation) instead. Or see
[README_NAVIGATION.md (online)](https://github.com/peter-kehl/present_on_github_with_reveal.js/blob/main/README_NAVIGATION.md)
for

- alternative online or local navigation, or
- VS Code tour, or
- video recording(s) (if any).

---

# Audience and purpose

- general Rust developers moving to low level or `no_std`
- non-Rust low level developers moving to Rust

# In scope

- Rust only; focusing on the language (less so on development tools)
- [`no_std`](https://docs.rust-embedded.org/book/intro/no-std.html) library `crates` (without `std`
   library), _with_ or _without_ heap

---

# Related, but mostly out of scope

- hardware, deployment, embedded debugging
- low level development in general (techniques, architectures, tools)
- specifics of [real
  time](https://doc.rust-lang.org/nightly/embedded-book/interoperability/index.html#interoperability-with-rtoss)
  applications and use with RTOS (Real Time OS)
  - Rust _is_ suitable for real time (because of no garbage collection)
  - this applies to both `no_std` and `std`
- `wasm` (Web Assembly), although [`no_std` crates are usually
   wasm-friendly](https://rahul-thakoor.github.io/using-no-standard-library-crates-with-webassembly)
- [ABI](https://doc.rust-lang.org/nightly/reference/abi.html) (Application Binary Interface),
   especially
  - [`#[no_mangle]`](https://doc.rust-lang.org/nightly/reference/abi.html#the-no_mangle-attribute)
  - [type layout](https://doc.rust-lang.org/nightly/reference/type-layout.html) and the
     [Rustonomicon (Unsafe Rust)](https://doc.rust-lang.org/nightly/nomicon) > [Alternative
     Representations](https://doc.rust-lang.org/nightly/nomicon/other-reprs.html).
    - Rustonomicon has parts applicable to safe and/or `std` Rust, too. One of its gems:
       [Higher-Rank Trait Bounds](https://doc.rust-lang.org/nightly/nomicon/hrtb.html).
- [`async/await` in no_std](https://ferrous-systems.com/blog/stable-async-on-embedded)

---

- [`unsafe` code](https://doc.rust-lang.org/nightly/book/ch19-01-unsafe-rust.html)
  - is... unsafe, indeed. Especially so in `std`, because some mistakes are very difficult to notice
     and/or reproduce. Even on the same machine, model or architecture, incorrect memory access
     (race conditions...) may show up only under specific conditions (threads and the related state,
     CPU caches per-core or shared between cores...).
  - more complicated across CPU's/architectures and on NUMA (non uniform memory architecture) - on
     x86 it used to be mostly AMD, but Intel has had it on server CPU's, plus on desktops since
     [12th generation CPU's](https://en.wikipedia.org/wiki/Alder_Lake#Scheduler_support)).

     Even if a multi-level cache CPU or NUMA can detect cross-core access and it flushes/reloads the
     relevant data in the cache(s), that may counteract any gains expected from `unsafe`.
  - probably easier on `no_std` embedded, because of no threads & no other applications or processes
  - FFI (Foreign Function
     Interface)/[Interoperability](https://doc.rust-lang.org/nightly/embedded-book/interoperability/index.html)
    - [calling external code from Rust:
       `extern`](https://doc.rust-lang.org/nightly/book/ch19-01-unsafe-rust.html#using-extern-functions-to-call-external-code)
    - [calling Rust functions from
       C](https://dev.to/dandyvica/how-to-call-rust-functions-from-c-on-linux-h37)
    - the mainstream [Rust from
       Python](https://saidvandeklundert.net/learn/2021-11-18-calling-rust-from-python-using-pyo3)
       with [pyo3 and
       maturin](https://towardsdatascience.com/nine-rules-for-writing-python-extensions-in-rust-d35ea3a4ec29)
       is for standard Python, so `std`-only.
       [pyo3::with_embedded_python_interpreter()](https://docs.rs/pyo3/latest/pyo3/fn.with_embedded_python_interpreter.html)
       and [RustPython/RustPython](https://github.com/RustPython/RustPython) "embed" Python into
       Rust, but they don't focus on embedded systems (and they most likely require `std`).
- [Rust](https://doc.rust-lang.org) and [Cargo &
   dependencies](https://doc.rust-lang.org/nightly/cargo/reference/specifying-dependencies.html) in
   general

---

# Limited scope

- cross-platform
  - also for `std` (where `std` library exists)
  - Cargo documentation keeps this [quite
     hidden](https://doc.rust-lang.org/nightly/cargo/faq.html#does-cargo-handle-multi-platform-packages-or-cross-compilation)
    - [`.cargo/config.toml` -->
       `build.target`](https://doc.rust-lang.org/nightly/cargo/reference/config.html#buildtarget),
       for example:

       ```rust
       [build]
       target = "aarch64-unknown-none-softfloat"
       ```

    - or: [`cargo build
       --target`](https://doc.rust-lang.org/nightly/cargo/commands/cargo-build.html#compilation-options)
  - Architectures supported by Rust - in [three
     tiers](https://doc.rust-lang.org/nightly/rustc/target-tier-policy.html). See also [the rustc
     book](https://doc.rust-lang.org/nightly/rustc) > [Platform
     Support](https://doc.rust-lang.org/nightly/rustc/platform-support.html). Beware many Tier 2
     wouldn't build a simple application (not even as heapless `no_std`).
  - [Cargo book > Platform specific
     dependencies](https://doc.rust-lang.org/nightly/cargo/reference/specifying-dependencies.html#platform-specific-dependencies)
  - per-platform build/linking configuration - have
     [`.cargo/config.toml`](https://doc.rust-lang.org/cargo/reference/config.html)
  - [`features`](https://doc.rust-lang.org/nightly/cargo/reference/features.html)
    - compile time-selectable subsets of library
     [`crates`](https://doc.rust-lang.org/nightly/cargo/appendix/glossary.html#crate)

---

## Ecosystem

- the focus here is on Rust [`#![no_std]`]((<https://docs.rust-embedded.org/book/intro/no-std.html>)
   language challenges, patterns and tips
- we mention very little about architectures, `no_std` crates or tools
<!--  and only
   * main stream, or
   * ones with the least development pain (easy debugging), or
   * ones with high or emerging Rust support
   * ones with an emulator
 -->
---

# Prerequisites

- no need for low level experience
- common (and a few uncommon) aspects of general (but not necessarily low level) Rust, especially
  - setting up a project and basics of
     [cargo](https://doc.rust-lang.org/nightly/book/ch01-03-hello-cargo.html)
  - [package layout](https://doc.rust-lang.org/nightly/cargo/guide/project-layout.html)
  - [dependencies](https://doc.rust-lang.org/nightly/cargo/guide/dependencies.html)
  - [`features`](https://doc.rust-lang.org/nightly/cargo/reference/features.html)
  - architecture or `feature`-based [conditional
     compilation](https://doc.rust-lang.org/nightly/reference/conditional-compilation.html) with
     `#[cfg(...)]`(<https://doc.rust-lang.org/rust-by-example/attribute/cfg.html>) attribute.
  - [slices](https://doc.rust-lang.org/nightly/book/ch04-03-slices.html)
  - [enums](https://doc.rust-lang.org/nightly/book/ch06-01-defining-an-enum.html)
    - not as simple as Java/C++ enums, but potentially carrying variant-specific data field(s) (like
     Scala's and Haskell's algebraic types)
  - [iterators](https://doc.rust-lang.org/nightly/book/ch13-02-iterators.html) (like [C#
     iterators](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/iterators),
     or
     [java.util.stream.Stream](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.html);
     but not like
     [java.util.Iterator](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Iterator.html)!)

---

- [generics](https://doc.rust-lang.org/nightly/book/ch10-00-generics.html) and especially [generic
     data types](https://doc.rust-lang.org/nightly/book/ch10-01-syntax.html). (Like templates in
     C++, or [generics in other languages](https://www.wikiwand.com/en/Generic_programming). But no
     type erasure like in [Java
     generics](https://docs.oracle.com/javase/tutorial/java/generics/erasure.html) or some [Haskell
     generics](https://wiki.haskell.org/Generics).)
- [const generics](https://rust-lang.github.io/rfcs/2000-const-generics.html). As of mid 2022, const
     generics are not in the Rust book (not even in `nightly`). See the above or the [Rust
     Reference](https://doc.rust-lang.org/nightly/reference/items/generics.html#const-generics).
     (See also [Vancouver Rust's
     presentation](https://github.com/vancouver-rs/talks/tree/master/const-generics).)
- anything in this document starting with "_New to Rust?_"
- optional (for deeper understanding of some code examples):
     [core::option](https://doc.rust-lang.org/nightly/core/option/index.html) and
     [core::result](https://doc.rust-lang.org/nightly/core/result/index.html)
- experience with a statically typed and compiled language
- basics of linking, heap and stack
- Rust [installation](https://doc.rust-lang.org/nightly/book/ch01-01-installation.html) including
   [`rustup`](https://rust-lang.github.io/rustup) and
   [`cargo`](https://doc.rust-lang.org/nightly/cargo) (the recommended way)
- Linux/Mac OS/Unix file path notation (in case you are on Windows)

---

# no_std

- for low level (without an operating system, or a part of an OS kernel)
- Have [`#![no_std]`]((<https://docs.rust-embedded.org/book/intro/no-std.html>) line at the top of
   your [crate](https://doc.rust-lang.org/nightly/cargo/appendix/glossary.html#crate) (`lib.rs` or a
   top level source file for a binary). See also
   [`package`](https://doc.rust-lang.org/nightly/cargo/appendix/glossary.html#package) and
   [`target](https://doc.rust-lang.org/nightly/cargo/appendix/glossary.html#target).
- no default fatal error
   ([`panic`](https://doc.rust-lang.org/nightly/book/ch09-01-unrecoverable-errors-with-panic.html#unwinding-the-stack-or-aborting-in-response-to-a-panic))
   handler; you must choose either
- you can provide a custom
   [`#[panic_handler]`](https://doc.rust-lang.org/nightly/std/alloc/trait.GlobalAlloc.html))
- no [`std`](https://doc.rust-lang.org/nightly/std/index.html) library (no modules starting with
   `std::`), but
- a limited subset of `std` is available as
   [`core`](https://doc.rust-lang.org/nightly/core/index.html). For example, instead of
   [`std::option::Option`](https://doc.rust-lang.org/nightly/std/option/enum.Option.html), use
   [`core::option::Option`](https://doc.rust-lang.org/nightly/core/option/enum.Option.html). There's
   also [`core::result::Result`](https://doc.rust-lang.org/nightly/core/result/enum.Result.html),
   [`core::iter::Iterator`](https://doc.rust-lang.org/nightly/core/iter/trait.Iterator.html),
   [`alloc::borrow::Cow](https://doc.rust-lang.org/alloc/borrow/enum.Cow.html)...

---

- stack-only (no heap: no
   [`std::boxed::Box`](https://doc.rust-lang.org/nightly/std/boxed/struct.Box.html), no
   [`std::vec::Vec`](https://doc.rust-lang.org/nightly/std/vec/struct.Vec.html), not even
   [`std::String`](https://doc.rust-lang.org/nightly/std/string/struct.String.html)...) by default,
   [`std::rc::Rc`](https://doc.rust-lang.org/std/rc/struct.Rc.html),
   [`std::sync::Arc`](https://doc.rust-lang.org/std/sync/struct.Arc.html)
- heap - only if you have an `allocator`:
  - register
     [`#[global_allocator]`](https://doc.rust-lang.org/nightly/std/prelude/v1/macro.global_allocator.html)
  - then use [`alloc`](https://doc.rust-lang.org/nightly/alloc/index.html) library
  - `alloc` is equivalent to the respective subset of `std`. Actually, `std` [re-exports `alloc` and
     `core` parts](https://doc.rust-lang.org/nightly/src/std/lib.rs.html).
  - [`alloc::boxed::Box`](https://doc.rust-lang.org/nightly/alloc/boxed/struct.Box.html),
     [`alloc::vec::Vec`](https://doc.rust-lang.org/nightly/alloc/vec/struct.Vec.html),
     [`vec!`](https://doc.rust-lang.org/nightly/alloc/macro.vec.html) macro (import it with `use
     alloc::vec;`),
     [`alloc::collections::VecDeque`](https://doc.rust-lang.org/nightly/alloc/collections/index.html#reexport.VecDeque),
     [`alloc::string::String`](https://doc.rust-lang.org/nightly/alloc/string/struct.String.html),
     [`alloc::rc::Rc`](https://doc.rust-lang.org/alloc/rc/struct.Rc.html),
     [`alloc::sync::Arc`](https://doc.rust-lang.org/alloc/sync/struct.Arc.html)
  - [`alloc::collections::BTreeSet`](https://doc.rust-lang.org/nightly/alloc/collections/index.html#reexport.BTreeSet),
     [`alloc::collections::BTreeMap`](https://doc.rust-lang.org/nightly/alloc/collections/index.html#reexport.BTreeMap)
     if your items/keys implement
     [`core::comp::Ord`](https://doc.rust-lang.org/nightly/core/cmp/trait.Ord.html).

     Even if our `struct` (or `enum`) instances can't be ordered in human terms, or if the actual
     order doesn't matter for us, we could define some (predictable) order and use
     `BTreeSet/BTreeMap` for most types.
- either way (heapless or with heap)
  - no
     [`std::collections::HashSet`](https://doc.rust-lang.org/nightly/std/collections/struct.HashSet.html),
     nor
     [`std::collections::HashMap`](https://doc.rust-lang.org/nightly/std/collections/struct.HashMap.html)
     (since computing hashes needs a source of entropy).

     Are you creating a library for both `std` and `no_std`, and you'd like to use `HashSet/HashMap`
     on `std`? You could define a trait, and implement it for `HashSet` (or `HashMap`) and for your
     `no_std`-compatible type. Then makde your API use
    - const generics (heapless `no_std`-compatible, and can be instantiated and then returned from a
       function), or
    - or `&dyn` _`trait`_ (heapless `no_std`-compatible, but can't be instantiated and then returned
       from a function), or
    - `Box<&dyn` _`trait>`_ (`no_std` with heap)
  - no [`std::thread::Thread`](https://doc.rust-lang.org/nightly/std/thread/struct.Thread.html) (and
     no multi-threading)
- Rust has `#![no_std]`, but it doesn't have `#![std]`. (Instead, availability of `std` library is
   implied if you don't have `#![no_std]`). Here we use `std` to refer to
   [`std`](https://doc.rust-lang.org/nightly/std/index.html) library, or to general purpose (no
   `no_std`) crates.

---

- Embrace [`nightly` channel](https://rust-lang.github.io/rustup/concepts/toolchains.html) (version)
   of Rust. `no_std` development is challenging enough. Help yourself by new language ergonomics &
   `core` library API (for example,
   [`#[bench]`](https://doc.rust-lang.org/nightly/cargo/reference/cargo-targets.html#benchmarks)). A
   lot of `nightly` API has become part of `beta` and `stable` (and anything new goes through
   `nightly` and `stable`, anyway). See the [Rust Forge](https://forge.rust-lang.org) schedule.

   Also, embedded devices often have no/restricted connectivity, and no other software running, so
   `nightly` may be secure enough. Plus, any upgrades replace the whole application, so even if
   `nightly` API changes, nothing outside of your embedded application changes on because of it (the
   updated Rust's API). You you can go ahead and apply such changes bravely.

   If you need `beta` or `nightly`, specify it per-project in
   [`rust-toolchain.toml`](https://rust-lang.github.io/rustup/overrides.html#the-toolchain-file).
   See also [nightly Rust](https://doc.rust-lang.org/nightly/book/appendix-07-nightly-rust.html),
   [channels](https://rust-lang.github.io/rustup/concepts/channels.html), [Rustup
   overrides](https://rust-lang.github.io/rustup/overrides.html) and [Rustup
   profiles](https://rust-lang.github.io/rustup/concepts/profiles.html).

   Because of that, all Rust links here to Rust
   [`core`](https://doc.rust-lang.org/nightly/core/index.html)/[`alloc`](https://doc.rust-lang.org/nightly/alloc/index.html)
   API, [Rust book](https://doc.rust-lang.org/nightly/book) and [Cargo
   book](https://doc.rust-lang.org/nightly/cargo) have a `nightly` prefix. They clearly mention
   which parts are `nightly` (or `beta`) only. You can access it as `beta` by changing the prefix,
   or seee the stable by removing the prefix.

   See also the [Rust RFC book](https://rust-lang.github.io/rfcs).

---

# no_std with heap

- Import `core::` and `alloc::` (instead of `std::`) wherever you can - even in `std` development.
   That brings awareness about what parts of your library are `no_std`-friendly. `std` [re-exports
   `core` and `alloc` parts](https://doc.rust-lang.org/nightly/src/std/lib.rs.html), so your library
   (whether `no_std` or `std`) will automatically work with `std` crates, too.

# no_std without heap

- like with `no_std` with heap, but without `alloc`
- common for microcontrollers

---

<!-- markdownlint-disable MD033 -->
<pre class="language-rust r-stretch code_relative_to_code_github_repo_raw">
<code
data-url="src/lib.rs"
data-line-start-delimiter="#![allow(rustdoc::bare_urls)]" data-line-end-delimiter="pub mod
index;">
</code>
</pre>
<!-- markdownlint-enable MD033    -->