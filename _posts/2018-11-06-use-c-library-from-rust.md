---
layout: post
title: "Using a C Library From Rust"
author: {{ site.author.name }}
tags:
    - rust
    - c
    - ffi
---

# Introduction

[Rust](https://www.rust-lang.org/en-US/) is a relatively new systems programming language designed and developed at Mozilla. It is touted as a competitor to C and C++, but is designed in a way that allows you to write high-level-esque code with zero overhead.

At Cisco, I work on writing low-level code for IOS-XR, one of Cisco's most popular router operating systems.  The vast majority of the code we write is in C, so I decided to gauge how easy (or difficult!) it is to use leverage existing C libraries developed in-house in new Rust code. I have also been interested in learning Rust for quite some while, so it's a win-win!

# chip: A Sample C Library

Suppose we want to use a library called `chip` in our Rust code. The library exports the following two functions:

```c
/**
* Returns a list of temperatures for the given chip instance.
*/
int chip_get_temps(uint32_t chip_inst, float *ret_temps);

/**
* Returns the state of the given chip instance.
*/
int chip_get_state(uint32_t chip_inst, chip_state_t *state);
```

Note the custom type in the second function. It is defined as follows:

```c
typedef enum {
    POWERED_OFF, POWERED_ON
} chip_state_t;
```

Based on the above, the `chip` library is probably talking to some kind of hardware device, but we don't need to worry about that.

# Setting Up a Project

After installing Rust, you can use Cargo to create a new project: `cargo new <my-project>`.  This will create a new folder in the current directory, initialize a Git repo, and create a Cargo.toml file.

If you look under `<my-project>/src`, you will see a barebones `main.rs` with a `main()` function. This is the entry point for your program.

Let's create a `deps/` directory and copy our library -- `libchip.so` -- into that directory.

To tell the Rust compiler (`rustc`) that we are linking against a C library, we need to create a build script. Put the following code into `build.rs` in the root project directory:

```rust
fn main() {
    let deps_dir = "./deps";
    println!("cargo:rustc-link-search=native={}", deps_dir);
    println!("cargo:rustc-link-lib=dylib=chip");
}
```

Our build script is simple: it prints out two "commands" that will be parsed by `rustc`:

1. "Add `deps/` to your library search path"
2. "Link against a library called `chip`"

With this, we should be ready to write some Rust.

# Wrapping in Rust

Before we can use anything from `chip`, we need to define the relevant functions in our Rust code.

Let's start with the `enum`. Add the following to the top of `main.rs`:

```rust
#[repr(C)]
enum ChipState {
    PowerOff, PowerOn
}
```

We are using a standard Rust enum, but with a `repr(C)` attribute. Basically, this tells the compiler to set the `enum`'s layout to that of C.

Next, let's define the function signatures in Rust:

```rust
extern "C" {
    fn chip_get_temps(chip_inst: u32, ret_temps: *mut f32);
    fn chip_get_state(chip_inst: u32, state: *mut ChipState);
}
```

The mapping from C to Rust is straightforward. The signatures are wrapped in `extern "C"` to indicate that their implementations will be linked in from a C library. We use raw pointers (`*`) because this is what C expects. The pointers are marked as `mut *` because they will be mutated within the C functions.

# Calling C from Rust

Using the above, we can easily call both of these functions:

```rust
// Number of temperatures reported per instance
// This would be a #define in the chip header file
const NUM_TEMPS: isize = 10;

fn main() {
    // Random chip instance
    let chip_inst: u32 = 3;

    // Create a Vec of floats of the required size
    // Note: This is where the C call will write its results
    let mut temps_buf: Vec<f32> = vec![0.0; NUM_TEMPS];

    // Get handle to the Vec's underlying raw ptr
    let temps_buf_ptr = temps_buf.as_mut_ptr();

    // All C calls must be wrapped in an unsafe block!
    unsafe {
        let err = chip_get_temps(chip_inst, temps_buf_ptr);
        if err != 0 {
            panic!("Error getting temps!");
        }
    }

    println!("Temperatures = {:?}", temps_buf);

    // Now let's query the chip state
    let mut state: ChipState = PowerOff; // Default

    unsafe {
        // We pass in a mutable pointer to the state variable
        let err = chip_get_state(chip_inst, &mut state);
        if err != 0 {
            panic!("Error getting state!");
        }
    }

    match state {
        ChipState::PowerOff => println!("Chip is powered off..."),
        ChipState::PowerOn => println!("Chip is powered on!")
    }
}
```

