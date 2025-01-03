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

I truncated the outcome for brevity, but you can see it is working. This concludes day 2.

## Sources

* https://rust-plug.yonkeltron.com/
* https://crates.io/crates/procfs
* https://docs.rs/procfs/latest/procfs/process/index.html