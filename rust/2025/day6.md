# Day 6 - cp program using syscalls

So far we used only safe Rust libraries to perform all the operations. While this is definitely recommended, if we want to learn system programming, we sometimes need to come down directly to an unsafe syscall level. Today we will implement the same program from day5 but using Linux syscalls.

## Code

```rust
use std::env;
use std::io;
use std::ffi::CString;
use std::io::ErrorKind;
use libc;
use linux_syscall::*;

fn main() -> io::Result<()>  {

    // 
    let args: Vec<String> = env::args().collect();
    
    if args.len() != 3 {
        return Err(io::Error::new(ErrorKind::Other, "Wrong amount of arguments!"));
    }

    let arg1 = &args[1]; // Access the first argument directly
    let arg2 = &args[2]; // Access the second argument directly
    let filename1 = CString::new(arg1.clone()).unwrap();
    let filename2 = CString::new(arg2.clone()).unwrap();

    let flags_read = libc::O_RDONLY; 
    let flags_write = libc::O_WRONLY | libc::O_CREAT | libc::O_TRUNC; 
    let mode = 0o644;

    // Open the file1
    let result = unsafe {
        syscall!(
            SYS_open,
            filename1.as_ptr(),
            flags_read,
            mode
        )
    };

    // Handle the Result type
    let fd1: i64 = result.try_i64().unwrap();

    if fd1 == -1 {
        panic!("Failed to open file: {}", std::io::Error::last_os_error());
    }

    // Open the file2


    let result = unsafe {
        syscall!(
            SYS_open,
            filename2.as_ptr(),
            flags_write,
            mode
        )
    };

    // Handle the Result type
    let fd2: i64 = result.try_i64().unwrap();

    if fd2 == -1 {
        panic!("Failed to open file: {}", std::io::Error::last_os_error());
    }

    // Read from the file1
    let mut buffer = [0u8; 1024];
    let result = unsafe { syscall!(SYS_read, fd1, buffer.as_mut_ptr(), buffer.len()) };
    let bytes_read: i64 = result.try_i64().unwrap();
    if bytes_read == -1 {
        panic!("Failed to read from file: {}", std::io::Error::last_os_error());
    }

    // Write to the file2

    let result = unsafe { syscall!(SYS_write, fd2, buffer.as_mut_ptr(), buffer.len()) };
    let bytes_read: i64 = result.try_i64().unwrap();
    if bytes_read == -1 {
        panic!("Failed to write to file: {}", std::io::Error::last_os_error());
    }

    // Close the files
    let result_result = unsafe { syscall!(SYS_close, fd1) };
    let result: i64 = result_result.try_i64().unwrap();
    if result == -1 {
        panic!("Failed to close file: {}", std::io::Error::last_os_error());
    }
    let result_result = unsafe { syscall!(SYS_close, fd2) };
    let result: i64 = result_result.try_i64().unwrap();
    if result == -1 {
        panic!("Failed to close file: {}", std::io::Error::last_os_error());
    }
    Ok(())
}
```

## Execution


```
vulcan@vulcan:~/Documents/rust/100days/day6/target/debug$ echo "test1234" > test.txt
vulcan@vulcan:~/Documents/rust/100days/day6/target/debug$ cat test.txt 
test1234
vulcan@vulcan:~/Documents/rust/100days/day6/target/debug$ cat test2.txt
cat: test2.txt: No such file or directory
vulcan@vulcan:~/Documents/rust/100days/day6/target/debug$ ./day6 test.txt test2.txt
vulcan@vulcan:~/Documents/rust/100days/day6/target/debug$ cat test2.txt
test1234
```

```
vulcan@vulcan:~/Documents/rust/100days/day6/target/debug$ echo $(python3 -c "print('A' * 1500)") > test3.txt
vulcan@vulcan:~/Documents/rust/100days/day6/target/debug$ wc -c test3.txt
1501 test3.txt
vulcan@vulcan:~/Documents/rust/100days/day6/target/debug$ ./day6 test3.txt test4.txt
vulcan@vulcan:~/Documents/rust/100days/day6/target/debug$ wc -c test4.txt
1024 test4.txt
```


## Notes

Obviously this program has a few issues, as we can see in the second execution, it will only copy first 1024 bytes of the file. It also only works for 64-bit implementations. Because we are working with syscalls directly, we lose a lot of Rust security and convenience, and we need to approach amny things as we would be in C. This also means taking care of things like buffer sizes and checking if there is data left to read, etc.

There are w few keys to calling syscalls via rust:

1. first `linux_syscall` crate, it is important to remember that to install this crate you need to use `cargo add linux-syscall` or add `linux-syscall = "1.0.0"` to your toml file. Notice `-` instead of `_`? Yeah this caused me to waste some time, but Gemini came to the rescue (although also not without issues because it's initial answers didnt use any `use` statement). The crate documentation can be found below, but it is basically what you would expect so a syscall implementation in Rust via usage of `syscall!` macro.
2. `unsafe` keyword. In Rust to use any unsafe code, you need to declare an `unsafe` block. This will allow you to do more things than in normal "safe" Rust (bue there are still some safeguards in place), but it will also allow Rust to recover better from potential errors in the unsafe code. More explanation about `unsafe` rust in the links below, but safe to say that using direct syscalls is considered extremely unsafe, so it always has to be wrapped in the `unsafe` block (it won;t compile without this)
3. `syscall!` macro. This is a macro to execute syscalls directly from rust. You need to provide a `SYS_` constant with a name of syscall you want call and all the arguments to the syscall. Since Linux kernel is written in C, and so syscalls traditionally expect C-like structures, there are some conversions that are required. For example we need to declare a CString (null terminated string) type like this `let filename1 = CString::new(arg1.clone()).unwrap();` so we can pass it to the syscall as a pointer in a true C-fashion: `filename1.as_ptr()`. We can also declare syscall flags by using `libc` crate: `let flags_write = libc::O_WRONLY | libc::O_CREAT | libc::O_TRUNC;`
4. Syscalls return a very specific Result type, dependent on architecture, in this case `linux_syscall::arch::x86_64.Result`. To assign it to a type `i64` variable required by another syscall, we first need to unpack this result by calling `try_i64()` which in turn will return `Result<i64, Error>` which we can unwrap as usual.

## Sources

* https://docs.rs/linux-syscall/latest/linux_syscall/
* https://docs.rs/linux-syscall/latest/linux_syscall/macro.syscall.html
* https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html
* https://shadow.github.io/docs/rust/linux_syscall/arch/x86_64/struct.Result.html
* https://man7.org/linux/man-pages/man2/open.2.html
* https://man7.org/linux/man-pages/man2/read.2.html
* https://man7.org/linux/man-pages/man2/write.2.html
* https://man7.org/linux/man-pages/man2/close.2.html