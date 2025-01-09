# Day 9 - IPC - unnamed pipe and rand

We already knwo how to spawn new processes, today we will look at Inter-process communication methods (IPC). There are multiple IPC methods supported by Rust, some are platform independent Rust APIs, and some are OS dependent. We will look at more of them in the coming days, but we will start with unnamed pipes. 

To quote Rust docs:

> Unlike named pipes, unnamed pipes are only accessible through their handles – once an endpoint is closed, its corresponding end of the pipe is no longer accessible. Unnamed pipes typically work best when communicating with child processes.

These are Rust specific object, that is an abstraction layer over OS-level IPC methods. It is pretty easy to use, as it only requires declaring a pipe that will serve as a 2-way buffer between processes that we can write to and read from, as long as we have access to the open handle.

Another important concept we will introduce today is randomness. We will look at randomness in Rust in more details in the future, but today we simply use a standard `rand` crate to generate a random byte. 

## Code

```rust
use std::io::Read;
use std::io::Write;
use fork::{fork, waitpid, Fork};
use rand;
use interprocess::unnamed_pipe::pipe;

fn main() -> Result<(),()> {
    let (mut tx, mut rx) = pipe().unwrap();
    match fork() {
       Ok(Fork::Parent(child_pid)) => {
            println!("Continuing execution in parent process, new child has pid: {}", child_pid);
            let mut buf: [u8; 1] = [0; 1];
            match waitpid(child_pid) {
                Ok(_) => {
                    rx.read(&mut buf).unwrap();
                    println!("Child returned {}", &buf[0]);
                },
                Err(_) => eprintln!("Failted to wait on child"),
            }
        return Ok(())
       }
       Ok(Fork::Child) => {
            println!("I'm a child, calculating a new value and passing to the parent.");
            let x = rand::random::<u8>();
            tx.write(&[x]).unwrap();
            println!("Number sent to parent {}", x);
            println!("Child exiting");
        }
       Err(_) => eprintln!("Failed to fork"),
    }
    Ok(())
}
```

## Execution


```
vulcan@vulcan:~/Documents/rust/100days/day9/target/debug$ ./day9
Continuing execution in parent process, new child has pid: 143500
I'm a child, calculating a new value and passing to the parent.
Number sent to parent 33
Child exiting
Child returned 33
```


## Notes

1. First we need to declare unnamed pipe: `let (mut tx, mut rx) = pipe().unwrap();` - these need to be mutable as they will change their value. Since we are using forking here as a method of spawning a child process, we don't need to pass the handle to the pipe to a child process as the forked process inherits all handles and all variables in scope when the fork is performed. We will discover other methods of passing the handle to the other process in the future.
2. Later we need to initialize a 1-byte buffer in the parent process: `let mut buf: [u8; 1] = [0; 1];` this is enough as we only want to pass one byte between processes.
3. Now we wait for child to finish and try to read from the buffer: `rx.read(&mut buf).unwrap();` - if all goes well we can read a value passed by our child process and print it.
4. In the meantime in the child process we will: chose a random byte `let x = rand::random::<u8>();`
5. Write it to the buffer: `tx.write(&[x]).unwrap();`

The code is pretty simple, the only issue I had was that I initially used a bigger 100-byte buffer `let mut buf: [u8; 100] = [0; 100];` and I used ` rx.read_exact(&mut buf)` method instead of `read()`. This caused the parent process to wait indefinitely for the entire 100-byte buffer to be filled-in before reading it (read exact, reads the exact number of bytes equal to the size of the buffer). Changing `read_exact` to `read` was enough for this code to work, but it is generally a better practice to use a smaller buffer if we know the size of the input.


## Sources

* https://docs.rs/interprocess/latest/interprocess/
* https://docs.rs/interprocess/latest/interprocess/unnamed_pipe/index.html
* https://docs.rs/rand/latest/rand/
* https://doc.rust-lang.org/std/io/trait.Read.html#method.read_exact
* https://doc.rust-lang.org/std/io/trait.Read.html#tymethod.read