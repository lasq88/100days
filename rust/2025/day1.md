# Day 1 - Using Cargo and Hello World

It's not the first Hello World I write in Rust, but it is traditional to start a challenge with it. During that day I also created and configured my development environment and started to get used to using cargo.

## Development environment 

I will use Sublime 4 (my favorite editor) together with `Rust-Enhanced` plugin as my IDE. I will run it on a VM with Debian 12 as the goal is not only to learn rust but to learn Linux system programming in rust.

## Cargo

So first let's create a new project in Cargo. It is as simple as that:

```
vulcan@vulcan:~/Documents/rust/100days$ cargo new day1
    Creating binary (application) `day1` package
note: see more `Cargo.toml` keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html
```

## Code

Now we will create a Hello World program. In fact the basic Hello world is already created by Cargo in `day1/src/main.rs` and it looks like this:

```rust
fn main() {
    println!("Hello World!");
}
```

Let's modify it a little bit to include some variables and string formatting:

```rust
fn main() {
    let day = 1;
    println!("Hello, world! It is a day {day} of 100 days of Rust challenge");
}
```

## Execution

Now let's compile our program (simple ctrl+b in Sublime) and run it to see if it works. Our binary will be created in `day1/target/debug` since this is a default build mode (we can change it later):

```
vulcan@vulcan:~/Documents/rust/100days/day1/target/debug$ ./day1
Hello, world! It is a day 1 of 100 days of Rust challenge
```

It works! This concludes day 1 of 100 days of Rust challenge. Below you can find some sources I used for day 1 challenge.

## Sources

* https://rust-lang.github.io/rust-enhanced/
* https://github.com/rust-lang/rust-enhanced
* https://doc.rust-lang.org/book/ch01-03-hello-cargo.html
* https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html