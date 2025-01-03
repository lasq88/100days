# Day 3 - program arguments

One of the important programming concepts is passing arguments to the program. This program can later have different outcomes based on what argument was passed to it. Today we will try to modify a program from yesterday, in a way that we will pass it number of TTY and it will print only processes with this specific TTY.

## Code

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    let arg1 = args.get(1);
    let tty_b = match &arg1 {
        Some(n) => n.parse::<i32>(),
        None => Ok(0)
    };
    let tty_n = match tty_b {
        Ok(n) => n,
        Err(_e) => panic!("Please provide a numerical argument")
    };

    let tps = procfs::ticks_per_second();
    let tty = format!("pty/{}", tty_n);
    println!("{: >5} {: <8} {: >8} {}", "PID", "TTY", "TIME", "CMD");

    for prc in procfs::process::all_processes().unwrap() {
        let prc = prc.unwrap();
        let stat = prc.stat().unwrap();
        let curr_tty = stat.tty_nr().1;
        if curr_tty == tty_n {
            // total_time is in seconds
            let total_time =
                (stat.utime + stat.stime) as f32 / (tps as f32);
            println!(
                "{: >5} {: <8} {: >8} {}",
                stat.pid, tty, total_time, stat.comm
            );
        }
    }
}
```

## Execution

We can now execute program with an argument and have different outputs:

```
vulcan@vulcan:~/Documents/rust/100days/day3/target/debug$ ./day3 0
  PID TTY          TIME CMD
    1 pty/0        0.88 systemd
    2 pty/0        0.02 kthreadd
    3 pty/0           0 rcu_gp
    4 pty/0           0 rcu_par_gp
    5 pty/0           0 slub_flushwq
    6 pty/0           0 netns
    8 pty/0           0 kworker/0:0H-events_highpri
[TRUNCATED]

vulcan@vulcan:~/Documents/rust/100days/day3/target/debug$ ./day3 1
  PID TTY          TIME CMD
  694 pty/1           0 agetty
 7844 pty/1           0 bash
```

**Note:** if you don't have any output for pty/1, try login a second ssh session to your Linux.

We also see that if we run a program without any argument it defaults pty to 0 and gives the same output as `./day3 0`:

```
vulcan@vulcan:~/Documents/rust/100days/day3/target/debug$ ./day3
  PID TTY          TIME CMD
    1 pty/0        0.88 systemd
    2 pty/0        0.02 kthreadd
    3 pty/0           0 rcu_gp
    4 pty/0           0 rcu_par_gp
    5 pty/0           0 slub_flushwq
    6 pty/0           0 netns
    8 pty/0           0 kworker/0:0H-events_highpri
```

Additionally if we call it with an argument that is not a number, it will raise a customized error:

```
vulcan@vulcan:~/Documents/rust/100days/day3/target/debug$ ./day3 abc
thread 'main' panicked at src/main.rs:12:20:
Please provide a numerical argument
```

## Notes

There are few things to unroll here.

1. First we need to get program arguments, we do that by calling: `let args: Vec<String> = env::args().collect();`. It's a simple piece of code that we can basically copy + paste to get arguments, but since our goal is to lean Rust and not copy - paste the code, let's unroll this.
    * Firstly we have `Vec<String>`. It's a Vector of Strings. If you are confused like me, Vectors are basically like arrays but with more capabilities. You can read more about vectors in the linked documentation.
    * Than we have a `collect()` method. It basically transforms an Iterator into a collection of a specific type (in this case `Vec<String>`). Concept of `Iterator` is pretty similar in Rust that it is in Python and many other languages so I won't dive into this, but please read the documentation if you are confused. 
2. Now we have a collection of arguments, we could simply try to access value like this: `let arg1 = &args[1];` but from now on, I will try to implement a correct error handling, especially when it comes to the user input. In this case we don't know if user provided correct argument, so we will have to handle situations when they won't. So instead of accessing the value directly, we can use a `.get()` method. This method wil lreturn an `Option` enum. This enum is similar to the `Result` enum we discovered on day 2, but instead of `Ok(T)` and ` Err(E)`, it will return `Some(T)` in case that value could be retrieved from the collection or `None` in case it couldn't.
3. To handle `Option` enum we can use a `match` keyword which will match one of the enum variables and take specific actions based on that. In this case we will return a value n in case it could've been retrieved (`Some(n) => n.parse::<i32>(),`) or we will default to 0 in case it couldn't (`None => Ok(0)`)
4. Since program arguments are collected into Vector of Strings, we will need to convert them to a numerical type if we want to use them in comparisons later. We could just do `n.parse::<i32>().unwrap()` but again what if user provides argument that cannot be parsed to a number. So instead of `unwrap()` we will use the same `match` keyword to handle the `Result` enum returned by `parse()` method. In this case we will simply return a number in case of `Ok(n)` or raise an error if value couldn't be parsed.


## Sources

* https://doc.rust-lang.org/book/ch12-01-accepting-command-line-arguments.html
* https://stackoverflow.com/questions/27043268/convert-a-string-to-int
* https://doc.rust-lang.org/std/primitive.slice.html#method.get
* https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html
* https://doc.rust-lang.org/book/ch06-02-match.html
* https://doc.rust-lang.org/std/option/enum.Option.html
* https://doc.rust-lang.org/nightly/std/iter/trait.Iterator.html#method.collect
* https://dhghomon.github.io/easy_rust/Chapter_21.html
* https://doc.rust-lang.org/book/ch13-02-iterators.html