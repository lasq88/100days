# Day 8 - fork and measuring time

Today we will implement a similar program to the last one, but we will use a fork crate, instead of detecting child and parent processes ourselves. `fork()` is an important concept in Linux system programming, and it is what allows many programs, especially servers to handle multiple concurrent tasks (like connections) without blocking. Fork is not natively supported by the Rust standard library, due to design decisions, but we have a crate that can help us with this.

We will also look at simple functions to measure elapsed time.

## Code

```rust
use fork::{fork, waitpid, Fork};
use std::{thread, time};

fn main() -> Result<(),()> {
    match fork() {
       Ok(Fork::Parent(child_pid)) => {
           println!("Continuing execution in parent process, new child has pid: {}", child_pid);
           match waitpid(child_pid) {
            Ok(_) => println!("Child exited"),
            Err(_) => eprintln!("Failted to wait on child"),
        }
       }
       Ok(Fork::Child) => {
            let timer = time::Duration::from_secs(5);
            let now = time::Instant::now();
            println!("I'm a child, sleeping for {} seconds.", timer.as_secs());
            thread::sleep(timer);
            println!("{} seconds passed...", now.elapsed().as_secs())
        }
        Err(_) => eprintln!("Failed to fork"),
    }
    Ok(())
}
```

## Execution


```
vulcan@vulcan:~/Documents/rust/100days/day8/target/debug$ time ./day8
Continuing execution in parent process, new child has pid: 127247
I'm a child, sleeping for 5 seconds.
5 seconds passed...
Child exited

real    0m5.001s
user    0m0.001s
sys     0m0.000s
```


## Notes

Today's program is a pretty simple one because I didn't have much time to work with Rust, but I have an idea on how to expand it into other topics in the next days. Nevertheless few important system programming concepts to unroll. First one is fork, which I already mentioned in the introduction. It allows us to "clone" a concurrent process in the same state as a parent, for handling a specific task. More info about fork, as always, in the documentation. 

Another important concept is `waitpid` which basically instructs the parent process to wait for the child to finish before continuing execution. I hope to explore inter-process communication and synchronization more in the future of this challenge.

Lastly we use sleep from the thread crate to task the child program with sleeping for 5 seconds. Time is handled by a separate crate in rust btw. and it uses a special objects to facilitate working with it, which is pretty nice. Here we use `time::Duration::from_secs(5)` to declare duration of time lasting 5 seconds, that we later pass to the sleep method. We also use `time::Instant::now();` to represent current point in time, and later we can use a useful method `now.elapsed()` to measure how much time elapsed from this moment.

We also use Linux time command to measure real time that it took to execute our program and compare it with its internal measurement.

## Sources

* https://docs.rs/fork/latest/fork/enum.Fork.html
* https://docs.rs/fork/latest/fork/fn.waitpid.html
* https://internals.rust-lang.org/t/why-no-fork-in-std-process/13770
* https://man7.org/linux/man-pages/man2/fork.2.html
* https://man7.org/linux/man-pages/man3/waitpid.3p.html
* https://doc.rust-lang.org/std/time/index.html