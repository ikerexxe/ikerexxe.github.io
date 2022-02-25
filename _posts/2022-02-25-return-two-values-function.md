---
layout:     post
title:      "Return two values in a function in Rust"
date:       2022-02-25 20:00:00 +0100
categories: rust
---

# Context

Taking advantage of the fact that today is the learning day we have every semester at Red Hat, I am going to focus on learning something new in the Rust programming language. My idea is to check the different possibilities to return two values in a function. A good example of such a function is a division with remainder, which returns both the quotient and the remainder.

# Create project

First we need to create the project with the following command:

```
cargo new rust_return_values --name rust_return_values
```

And execute it to check that everything is working fine:

```
cargo run
```

You should get the typical "Hello, world!" output.

```
   Compiling rust_return_values v0.1.0 (/home/ipedrosa/Documents/Personal/Development/test/24_rust_return_values)
    Finished dev [unoptimized + debuginfo] target(s) in 0.74s
     Running `target/debug/rust_return_values`
Hello, world!
```

# Division sample

A sample program that divides and returns the quotient would be the following:

```
fn division(dividend: i32, divisor: i32) -> i32 {
    let quotient: i32;

    quotient = dividend / divisor;

    return quotient;
}

fn main() {
    let dividend = 20i32;
    let divisor = 5i32;

    let quotient = division(dividend, divisor);

    println!("divider {}", dividend);
    println!("divisor {}", divisor);
    println!("quotient {}", quotient);
}
```

But it only returns one value and we are interested in returning two.

# Tuples

The most basic way to do this is to use Tuples, which allows us to return two values. First, we need to change the return statement. Then, declare the remainder and calculate its value. Finally, return the tuple.

Full function at a glance.

```
fn division(dividend: i32, divisor: i32) -> (i32, i32) {
    let quotient: i32;
    let remainder: i32;

    quotient = dividend / divisor;
    remainder = dividend % divisor;

    return (quotient, remainder);
}
```

# Structure

Another option is to use a structure. For that purpose first we need to define it.

```
struct Data {
    quotient: i32,
    remainder: i32,
}
```

Then we can instantiate the structure, calculate the division values and return its value.

```
fn division(dividend: i32, divisor: i32) -> Data {
    let mut data = Data{quotient: 0, remainder: 0};

    data.quotient = dividend / divisor;
    data.remainder = dividend % divisor;

    return data;
}
```

# References and Smart Pointers

Even if it's possible, which I doubt, it isn't a good idea to use References or Smart Pointers to complete such a task. But why? One of the strongest points of Rust it is (or claims) to be memory safe, and this is done by assigning a lifecycle to every data that we generate. Usually this lifecycle is defined inside blocks, like the ones used for functions. So, even if we create a heap allocated variable, it will be dropped when the function ends.

# Additional information
* [Smart pointers in the Rust book](https://doc.rust-lang.org/book/ch15-00-smart-pointers.html)
* [References and borrowing in the Rust book](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html)
* [Lifetimes in Rust](https://blog.thoughtram.io/lifetimes-in-rust/)
