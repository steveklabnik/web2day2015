% Why Rust?

Steve Klabnik

# About Rust

Rust is a *systems* programming language focusing on three things:

* Safety
* Speed
* Concurrency

# Origins
            
Rust was written by Graydon Hoare. Mozilla has taken stewardship.
            
Mozilla writes lots of C++, and feels that pain.

# Why Rust?

> A language that doesn’t affect the way you think about programming, is not
> worth knowing.
>
> Alan Perlis

# Why Rust?

In a word: Ownership

# Why Rust?

Rust is poised to be many people’s first systems language. Newbies welcome!

The compiler is here to help you, and so are we.

# Stability

* Rust is currently at version 1.0

* New versions every six weeks!

* Backwards compatible

# Segfaults

```c
int main(void)
{
     char *s = "hello world";
     *s = 'H';
}
```

# Segfaults

```rust
fn main() {
    let s = &"hello world";
    *s = "H";
}
```

```text
segfault.rs:3:4: 3:6 error: type &str cannot be dereferenced
segfault.rs:3     *s = "H";
                  ^~
error: aborting due to previous error
task 'rustc' failed at 'explicit failure', ...
```

# Hello world

```rust
fn main() {
    println!("Hello, world");
}
```

# A little more complex

```rust
use std::thread;

fn main() {
    let nums = [1, 2];

    let odds = nums.iter().map(|&x| x * 2 - 1);
}
```

# References

```rust
fn plus_one(x: &i32) -> i32 {
    *x + 1
}

fn main() {
    println!("{}", plus_one(&5));
}
```

# Boxes

```rust
fn plus_one(x: &i32) -> i32 {
    *x + 1
}

fn main() {
    let x = Box::new(5);
    println!("{}", plus_one(x));
}
```

# Threads

```rust
use std::thread;

fn main() {
    let nums = [1, 2];
    let noms = ["Tim", "Eston", "Aaron", "Ben"];

    let odds = nums.iter().map(|&x| x * 2 - 1);

    for num in odds {
        thread::spawn(move || {
            println!("{} says hello from a thread!", noms[num]);
        });
    }

    thread::sleep_ms(100);
}
```

# Channels

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let result = 5;
        tx.send(result);
    });

    let result = rx.recv().unwrap();
    println!("{}", result);
}
```

# Threads and safety

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();
    let x = Box::new(5);

    thread::spawn(move || {
        let result = 5 + *x;
        tx.send(result);
    });

    let result = rx.recv().unwrap();
    println!("{}", result);
}
```

# Threads and safety

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();
    let x = Box::new(5);

    thread::spawn(move || {
        let result = 5 + *x;
        tx.send(result);
    });

    *x += 5;

    let result = rx.recv().unwrap();
    println!("{}", result);
}
```

# Threads and safety

<pre>
        13:12 error: use of moved value: `*x` [E0382]
         *x += 5;
         ^~~~~~~
        11:6 note: `x` moved into closure environment here because it has type `[closure(())]`, which is non-copyable
        thread::spawn(move || {
            let result = 5 + *x;
             tx.send(result);
         });
        note: in expansion of closure expansion
        11:6 note: expansion site
        11:6 help: perhaps you meant to use `clone()`?
        13:12 error: cannot assign to immutable `Box` content `*x`
         *x += 5;
         ^~~~~~~
</pre>

# Pattern matching

```rust
match my_number {
  0     => println("zero"),
  1 | 2 => println("one or two"),
  3..10 => println("three to ten"),
  _     => println("something else")
}
```

# Option

```rust
let msg = Some("howdy");

// Take a reference to the contained string
match msg {
    Some(ref m) => println!("{}", m),
    None => ()
}
```

# Closures

```rust
let nums = [1, 2, 3];
let square = |x| x * x;

let max: Vec<i32> = nums.iter().map(square).collect();
```

# Generics

```
fn foo<A, B>(x: A, y: B) {
    // ...
}
```

# Structs and traits

```
struct TimeBomb {
    explosivity: uint
}

impl Drop for TimeBomb {
    fn drop(&mut self) {
        for _ in range(0, self.explosivity) {
            println("blam!");
        }
    }
}
```

# More Traits

```
trait Seq<T> {
    fn length(&self) -> u32;
}

impl<T> Seq<T> for Vec<T> {
    fn length(&self) -> u32 { self.len() }
}
```

# More Traits

```
extern mod active_support;
use active_support::Period;
use active_support::Time;

fn main() {
  let time = Time::now();
  println!("{:?}", time);
  println!("{:?}", 2.days().from_now());
  println!("{:?}", 2.weeks().from_now());
  println!("{:?}", 2.months().from_now());
  println!("{:?}", 2.years().from_now());
}
```

# More Traits

```
use TimeChange;

pub trait Period {
  fn seconds(&self) -> TimeChange;
  fn minutes(&self) -> TimeChange;
  // ...
}

impl Period for uint {
  fn seconds(&self) -> TimeChange {
    TimeChange::new().seconds(*self as f32)
  }

  fn minutes(&self) -> TimeChange {
    TimeChange::new().minutes(*self as f32)
  }

  // ...
}
```

# FFI

```
use std::libc::size_t;

#[link(name = "snappy")]
extern {
    fn snappy_max_compressed_length(source_length: size_t) -> size_t;
}

fn main() {
    let x = unsafe { snappy_max_compressed_length(100) };
    println!("max compressed length of a 100 byte buffer: {}", x);
}
```

# FFI

```
pub fn validate_compressed_buffer(src: &[u8]) -> bool {
    unsafe {
        snappy_validate_compressed_buffer(src.as_ptr(), src.len() as size_t) == 0
    }
}
```

# FFI

```
pub fn uncompress(src: &[u8]) -> Option<Vec<u8>> {
    unsafe {
        let srclen = src.len() as size_t;
        let psrc = src.as_ptr();

        let mut dstlen: size_t = 0;
        snappy_uncompressed_length(psrc, srclen, &amp;mut dstlen);

        let mut dst = vec::with_capacity(dstlen as uint);
        let pdst = dst.as_mut_ptr();

        if snappy_uncompress(psrc, srclen, pdst, &amp;mut dstlen) == 0 {
            dst.set_len(dstlen as uint);
            Some(dst)
        } else {
            None // SNAPPY_INVALID_INPUT
        }
    }
}
```

# Embedding in other languages

Write a Ruby gem in Rust!

Write a Node module in Rust!

Write a Python extension in Rust!

# Cargo

```sh
$ cargo new foo --bin
$ cd foo
$ cargo run
   Compiling foo v0.1.0 (file:///home/steve/tmp/foo)
     Running `target/debug/foo`
Hello, world!
```

# Cargo.toml

```toml
[package]
name = “foo”
version = “0.1.0”
authors = [“Steve Klabnik <steve@steveklabnik.com>”]
```

# Cargo.toml

```toml
[package]
name = "foo"
version = "0.1.0"
authors = ["Steve Klabnik <steve@steveklabnik.com>"]

[dependencies]
regex = "0.1.33"
```

# Cargo

```sh
$ cargo build
    Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading regex v0.1.33
 Downloading regex-syntax v0.1.2
   Compiling regex-syntax v0.1.2
   Compiling regex v0.1.33
   Compiling foo v0.1.0 (file:///home/steve/tmp/foo)
```

# User feedback

> Yes thanks everyone, Rust has been a much more significant move than we had
> thought. We have performance, increased safety, modularity, traits  etc. In
> addition we have found something even more significant, first time working
> code.  By this I mean debugging time is almost zero (beyond logic errors of
> course) in comparison, for example our last IP library (rUDP in c++) was a 5
> month job to remove data races (luckily we used all clang sanitisers and
> static analysis in build and CI, but never the less). 

> Same task in Rust, worked of the bat, same with next layer (routing) and then
> next layer (vaults). This is very significant indeed and I am happy to
> discuss this further. I think it is vastly underestimated. 

> I have also found an interesting paradigm here, Ruby, Python, Jave, c# etc.
> programmers just lift Rust and code almost immediately (seeking out libraries
> and asking questions). C/C++ (especially library level) actually take a huge
> time to transition, it is like they have to “unlearn fear”. I say this as
> somebody who did a lot of c++ (dirvine on github). Not true of all, of
> course, but also a surprise. 

# Learning More

* The Rust Programming Language
* Rust By Example

# Discussion Fora

* http://{users,internals}.rust-lang.org
* /r/rust
* #rust on irc.mozilla.org
* This Week in Rust
* Stack Overflow

# Code

https://github.com/rust-lang/rust

# Thank you!

<3

Steve Klabnik
