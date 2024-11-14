+++
tags = ["Rust"]
date = "2023-02-18T12:00:00-08:00"
title = "Rust unit test layout"
+++

I've been succeeding in my fourth attempt at learning to use Rust.  I come from mostly a C++ background, and I've struggled the first couple of times I've tried to pick up Rust, ultimately stopping the effort to work on the next shiny quarter.

I started porting over my Nintendo 6502 processor emulator from C++ as a larger project to play around with.  One thing I had done in the C++ one is have tons of tests for the different instructions - checking clock cycles and internal register states for the emulated CPU.  Most of these tests ended up split up across different files to keep the codebase manageable.

## Unit Testing in Rust

In Rust, the guides all mention keeping your unit tests in the same file in a sub module declared in the file and marked as a test module with `#[cfg(test)]`.  

You can also have an integration tests folder that lives as a sibling folder to the `src` folder, but there the idea is to handle the modules as black boxes without the ability to peek into the internals - which in some cases can be useful in unit tests.

## Splitting your Rust Unit Tests into Multiple Files

It turns out to not be very difficult to split up your unit tests into separate files, but it took me a bit to figure out and thought it might useful to others.  Here is what the layout of my dummy project looks like now:

```
.
├── Cargo.lock
├── Cargo.toml
├── src
│   ├── cpu.rs
│   ├── lib.rs
│   └── tests
│       ├── cpu_tests.rs
│       └── mod.rs
```

All I've done is move the tests module's code into the `cpu_tests.rs` file in the subfolder `tests`. 

#### `cpu_tests.rs`
```rust
use crate::*;

#[test]
fn it_works() {
    let result = add(2, 2);
    assert_eq!(result, 4);
}
```

 I then have a `tests/mod.rs` that includes all of the test files, `tests/cpu_tests.rs` only in this case.  

#### `mod.rs`
```rust
pub mod cpu_tests;
```

Then by including that module in the top-level `lib.rs` and marking it as a test module, it will still get run as a unit test without cluttering up lib.rs itself.

#### `lib.rs`
```rust
pub fn add(left: usize, right: usize) -> usize {
    left + right
}

#[cfg(test)]
mod tests;
```

And the results of running the unit tests when I do a `cargo test`:

```bash
❯ cargo test
    Finished test [unoptimized + debuginfo] target(s) in 0.01s
     Running unittests src/lib.rs (target/debug/deps/nesters-0b3b29c8edf564ce)

running 1 test
test tests::cpu_tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

## Conclusion

Rust has been a great language to dig into a bit deeper, since it has some nice defaults, built in unit testing and the package and project management through cargo gets rid of a lot of boiler plate that I've constantly needed to duplicate across project for C++.

There are still some areas where I'm learning how to structure my code in a way that feels the most manageable and comfortable to me.  Given that the searches I did didn't show this solution published elsewhere, I hope others also find this useful.

<div id="commento"></div>
<script src="https://cdn.commento.io/js/commento.js"></script>
