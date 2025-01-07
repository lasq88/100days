# Day 7 - creating new processes and process info

In programming, and especially system programming sometimes we need to span a new process to do something for us. Today we will learn how to create new processes in Rust and also how to learn a little bit about the current running process.

## Code

```rust
use std::process::Command;
use std::env;
use procfs::process::Process;

fn main() -> Result<(),()> {
    let current_process = Process::myself().unwrap();
    let parent_pid = current_process.stat().unwrap().ppid;
    let pid = current_process.stat().unwrap().pid;

    // Get more information about the parent process
    let parent_process = Process::new(parent_pid);
    let parent_process_name = match parent_process {
        Ok(parent_process) => parent_process.stat().unwrap().comm,
        Err(e) => panic!("Error getting parent process info: {}", e),
    };

    println!("Parent Process name is: {}", parent_process_name);
    println!("Parent Process ID (PPID): {}", parent_pid);

    if parent_process_name == "day7" {
        println!("Hello from the child process! This process pid is: {}", pid);
        return Ok(())
    }
    else {
        println!("Hello from the parent process! This process pid is: {}", pid);
        println!("Executing child process...");
        let curr_path = match env::current_exe() {
            Ok(exe_path) => exe_path,
            Err(e) => panic!("failed to get current exe path: {e}"),
        };
        let child = Command::new(curr_path).output().expect("Failed to execute command");
        println!("{}", String::from_utf8_lossy(&child.stdout));
        return Ok(())
    }
}
```

## Execution


```
vulcan@vulcan:~/Documents/rust/100days/day7/target/debug$ ./day7
Parent Process name is: bash
Parent Process ID (PPID): 6288
Hello from the parent process! This process pid is: 102192
Executing child process...
Parent Process name is: day7
Parent Process ID (PPID): 102192
Hello from the child process! This process pid is: 102193
```


## Notes

First we will get some information about currently running process and also about its parent. To do that, we will use `procfs` crate that we already used on day 2. Nothing really groundbreaking here, and most of the concepts are pretty intuitive now for me.

Based on the parent process name we will decide if we are parent or a child and then perform different action. If you are experienced C programmer you can say: hey isn't it what `fork()` is supposed to do? It is, and we will learn how to do forks in Rust later. For now I wanted to use a basic `procfs::process::Process;` call to spawn a new process. 

This line is where all the magic happens:

` let child = Command::new(curr_path).output().expect("Failed to execute command");`

First we need to declare a new Command object, and only after calling the `output()` method it will be executed (this is important). Then we need to get the output from the child object and print it to the console, because it won't automatically be displayed (all output is by default captured by the parent process object if I understand this correctly). We can convert bytes to string with a following method `String::from_utf8_lossy(&child.stdout)` and then simply print it.


## Sources

* https://doc.rust-lang.org/std/process/index.html
* https://doc.rust-lang.org/std/env/fn.current_exe.html
* https://docs.rs/procfs/latest/procfs/