# Day 2 - process listing on Linux

Ok, so easy stuff is out of the way, let's dive directly into system programming. Today we will write a simple function that lists running processes on Linux.

For this we will use a `procfs` crate (crates are basically external libraries).

## Dependencies

To use a crate we need to add a dependency to the `Cargo.toml` file in our cargo project:

```
[dependencies]
procfs = "=0.17.0"
```

## Code

```rust
fn main() {
    let tps = procfs::ticks_per_second();
    println!("{: >5} {: <8} {: >8} {}", "PID", "TTY", "TIME", "CMD");

    for prc in procfs::process::all_processes().unwrap() {
        let prc = prc.unwrap();
        let stat = prc.stat().unwrap();
        // total_time is in seconds
        let total_time =
            (stat.utime + stat.stime) as f32 / (tps as f32);
        println!(
            "{: >5} {: <8} {: >8} {}",
            stat.pid, stat.tty_nr, total_time, stat.comm
        );
    }
}
```

It's a modified example from `procfs` documentation but instead listing processes running only on the same TTY, it lists all running processes.

It's a simple code that iterates in a for loop over each object from `procfs::process::all_processes().unwrap()` than queries simple process stats like PID, TTY, Running time and Command line and prints them to the terminal.

## Execution

Let's test our code

```
vulcan@vulcan:~/Documents/rust/100days/day2/target/debug$ ./day2 
  PID TTY          TIME CMD
    1 0            0.83 systemd
    2 0            0.02 kthreadd
    3 0               0 rcu_gp
    4 0               0 rcu_par_gp
    5 0               0 slub_flushwq
    6 0               0 netns
[TRUNCATED]
```

I truncated the outcome for brevity, but you can see it is working.

## Notes

One more thing I will try to do each day is to add some notes about Rust-specific concepts use in the code, this is both for my benefit and anyone who wants to follow my journey of learning rust.

In today code there were 2 concepts I figured I don't understand so well in Rust:

1. String formatting: we use this instruction to print a string into the console: `println!("{: >5} {: <8} {: >8} {}", "PID", "TTY", "TIME", "CMD");`. String formatting is a concept I understand and use a lot in python but I wanted to learn what these decorators mean exactly. Based on documentation (linked below):
    * `{: >5}` is a right-aligned (`>`) string of width `5` that is filled with spaces `: `. This means if our string is shorter than 5 characters it will be filled with spaces and aligned to the right.
    * `{: <8}` similarly is a left-aligned (`<`) string of width `8` that is filled with spaces `: `.
    * `{: >8}` right-aligned (`>`) string of width `8` that is filled with spaces `: `.

If you look at the output it makes sense:

```
  PID TTY          TIME CMD
    1 0            0.83 systemd
    2 0            0.02 kthreadd
    3 0               0 rcu_gp
```

You can count characters and see that PID has length 5 and TTY and TIME are both 8 characters long. You can also see how TTY is aligned to the left, while both PID and TIME are right-aligned.

2. `unwrap()` method. According to the documentation, `unwrap()` will return result for `Option` and `Result` enums. If unwrap encounters an error `Err` or a `None`, it will panic and stop the program execution.

Enums or Enumerations are concept that are not exclusive to Rust and you can find them in Python (and other languages) as well. If you want to know more about them read the documentation link provided below, but in short enum is just set of expected variants that a specific value can have.

`Result` enum can basically take two variants, either success (`Ok`) or error (`Err`) but it also contains a resulting value in case of `Ok` or error message in case of `Err`. This is a wrapper that is used to easier check for success and failure and facilitate error handling. 

In the layman terms though, `unwrap()` will simply allow us to get a return value of a specific function without having to manually handle and unwrap 2 different types of `Result` enum.

## Sources

* https://rust-plug.yonkeltron.com/
* https://crates.io/crates/procfs
* https://docs.rs/procfs/latest/procfs/process/index.html
* https://doc.rust-lang.org/std/fmt/
* https://stackoverflow.com/questions/36362020/what-is-unwrap-in-rust-and-what-is-it-used-for
* https://www.programiz.com/rust/unwrap-and-expect
* https://doc.rust-lang.org/reference/items/enumerations.html
* https://doc.rust-lang.org/std/result/enum.Result.html