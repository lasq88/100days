# Day 4 - writing to files

Another important concept for system programming are I/O operations, and specifically an interaction with filesystem. On day 4 we will write a simple program that will take as arguments a name of the file and a string and it will write this string to this file. If the given file exists, werror will be thrown.

## Code

```rust
use std::io::ErrorKind;
use std::env;
use std::io::Write;
use std::fs::File;
use std::io;
use std::path::Path;

fn check_file_exists(path: &Path) -> io::Result<()> {
    if path.exists() {
        Err(io::Error::new(
            io::ErrorKind::Other,
            "This file already exists"
        ))
    } else {
        Ok(())
    }
}

fn main() -> io::Result<()> {
    let args: Vec<String> = env::args().collect();
    
    if args.len() != 3 {
        return Err(io::Error::new(ErrorKind::Other, "Wrong amount of arguments!"));
    }

    let arg1 = &args[1]; // Access the first argument directly
    let arg2 = &args[2]; // Access the second argument directly

    let path = Path::new(arg1);

    check_file_exists(&path)?;

    let mut file = File::create(arg1)?;
    if let Err(e) = writeln!(file, "{arg2}") {
        eprintln!("Couldn't write to file: {}", e);
    }

    Ok(())
}
```

## Execution

```
vulcan@vulcan:~/Documents/rust/100days/day4/target/debug$ ./day4 test.txt "Hello World!"
vulcan@vulcan:~/Documents/rust/100days/day4/target/debug$ cat test.txt 
Hello World!
vulcan@vulcan:~/Documents/rust/100days/day4/target/debug$ ./day4 test.txt "abcd!"
Error: Custom { kind: Other, error: "This file already exists" }
vulcan@vulcan:~/Documents/rust/100days/day4/target/debug$ ./day4 test.txt
Error: Custom { kind: Other, error: "Wrong amount of arguments!" }
```

We can see that there is an error handling implemented, and program works as intended.

## Notes

I must admit, I struggled with this one a lot. I still cannot change my mindset to statically typed language like Rust. I got a bunch of compiler errors and if I corrected one, two new appeared ;) In the end I managed to solve this with a bit of help from Gemini. I must admit AI is insane in writing this simple tasks, but I don't want to use it too much, since this is supposed to be a learning journey and not a copy-paste of AI-generated code. But I settled on Gemini explaining to me why my code sux ;)

I will also let Gemini explain few concepts of this code, because it does it better than me.

1. Main function has now a return type: `fn main() -> io::Result<()> ` - this already cause issues for me, because I struggled with returning an error for an incorrect number of arguments. Apparently returned error also has to be of a specific type ;)
2. This is exactly the part that I struggled with and Gemini helped me to figure out:

```rust
if args.len() != 3 {
        return Err(io::Error::new(ErrorKind::Other, "Wrong amount of arguments!"));
    }
```

We can see that a specific `io::Error` needs to be returned in order for this to work. 

3. Later we access arguments directly and we have a function that checks if specific Path exists and returns another io Error if it's true and `Ok(())` if not. 
4. Next 3 lines are the core of this code. There is a lot going on here so I will let Gemini explain it to you ;)

**1. `let mut file = File::create(arg1)?;`**

*   **`File::create(arg1)`:** This part attempts to create a new file. 
    * `arg1` holds the filename or path to the file you want to create.
    * `File::create` returns a `Result<File, io::Error>`. 
        * If the file is created successfully, it returns `Ok(file)`, where `file` is a `File` object representing the newly created file.
        * If an error occurs (e.g., the file already exists, insufficient permissions, disk full), it returns `Err(error)`, where `error` is an `io::Error` object containing information about the error.

*   **`?` operator:** This is a shorthand for error handling.
    * If `File::create` returns `Ok(file)`, the `?` operator unwraps the `Ok` value and assigns the `File` object to the `file` variable.
    * If `File::create` returns `Err(error)`, the `?` operator immediately returns that `error` from the enclosing function ( `main` in this case). This allows for concise error propagation.

**2. `if let Err(e) = writeln!(file, "{arg2}") { ... }`**

*   **`writeln!(file, "{arg2}")`:** This macro attempts to write the value of `arg2` (likely a string) to the file represented by the `file` object.
    * It also adds a newline character to the end of the written content.
    * `writeln!` also returns a `Result<(), io::Error>`. 
        * If the write operation succeeds, it returns `Ok(())`.
        * If an error occurs (e.g., disk full, write permission denied), it returns `Err(error)`.

*   **`if let Err(e) = ... { ... }`:** This is a pattern matching construct for `Result` types.
    * It checks if the result of `writeln!` is an `Err` value. 
    * If it's an `Err`, the variable `e` is bound to the `io::Error` object.

*   **`eprintln!("Couldn't write to file: {}", e);`** 
    * If the write operation failed, this line prints an error message to the standard error stream using the `eprintln!` macro. 
    * The error message includes the specific error encountered during the write operation.

**In summary:**

This code snippet attempts to create a new file and write some data to it. It utilizes the `?` operator for concise error handling during file creation and the `if let` pattern to handle potential errors during the write operation, providing informative error messages to the user.

## Sources

* https://doc.rust-lang.org/book/appendix-02-operators.html
* https://stackoverflow.com/questions/32384594/how-to-check-whether-a-path-exists
* https://doc.rust-lang.org/rust-by-example/flow_control/if_else.html
* https://doc.rust-lang.org/std/path/struct.Path.html
* https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html
* https://doc.rust-lang.org/std/fs/struct.File.html
* https://doc.rust-lang.org/std/io/struct.Error.html