# Day 5 - cp program and user input

Now we know how to write to files, we can also learn how to read from them and implement a simple implementation of `cp`. Another super-important idea for any programming language is interacting with user input, so we will add a simple confirmation prompt for existing files. 

## Code

```rust
use std::io::Read;
use std::io::ErrorKind;
use std::env;
use std::io::Write;
use std::fs::File;
use std::io;
use std::path::Path;

fn check_file_exists(path: &Path) -> io::Result<&str> {
    if path.exists() {
        Ok("File exists")
    } else {
        Ok("File does not exist")
    }
}

fn main() -> io::Result<()> {
    let args: Vec<String> = env::args().collect();
    
    if args.len() != 3 {
        return Err(io::Error::new(ErrorKind::Other, "Wrong amount of arguments!"));
    }

    let arg1 = &args[1]; // Access the first argument directly
    let arg2 = &args[2]; // Access the second argument directly

    let path1 = Path::new(arg1);
    let path2 = Path::new(arg2);

    if check_file_exists(path2).unwrap() == "File exists" {
        println!("File {arg2} seems to already exist. Do you want to overwrite it? [y/N]");
        let mut input = String::new();
        io::stdin()
        .read_line(&mut input)
        .expect("Failed to read line");
        if input.trim() != "y" && input.trim() != "Y" {
            println!("Exiting");
            return Ok(());
        }
    }
    let mut file1 = File::open(path1)?;
    let mut file2 = File::create(path2)?;
    let mut buffer = Vec::new();
    if let Err(e) = file1.read_to_end(&mut buffer) {
        eprintln!("Couldn't write to file: {}", e);
    }
    else {
        if let Err(e) = file2.write(&buffer) {
            eprintln!("Couldn't write to file: {}", e);
        }   
    }
    Ok(())
}
```

## Execution

We can now copy files between folders:

```
vulcan@vulcan:~/Documents/rust/100days/day5/target/debug$ echo "test1234" > ~/test.txt
vulcan@vulcan:~/Documents/rust/100days/day5/target/debug$ cat ~/test.txt
test1234
vulcan@vulcan:~/Documents/rust/100days/day5/target/debug$ cat test1234.txt
cat: test1234.txt: No such file or directory
vulcan@vulcan:~/Documents/rust/100days/day5/target/debug$ ./day5 ~/test.txt test1234.txt
vulcan@vulcan:~/Documents/rust/100days/day5/target/debug$ cat test1234.txt
test1234
```

If the file already exists it will ask for the confirmation, with "N" being the default option:

```
vulcan@vulcan:~/Documents/rust/100days/day5/target/debug$ echo "test_1000" > ~/test2.txt
vulcan@vulcan:~/Documents/rust/100days/day5/target/debug$ ./day5 ~/test2.txt test1234.txt
File test1234.txt seems to already exist. Do you want to overwrite it? [y/N]
n
Exiting
vulcan@vulcan:~/Documents/rust/100days/day5/target/debug$ cat test1234.txt 
test1234
vulcan@vulcan:~/Documents/rust/100days/day5/target/debug$ ./day5 ~/test2.txt test1234.txt
File test1234.txt seems to already exist. Do you want to overwrite it? [y/N]
y
vulcan@vulcan:~/Documents/rust/100days/day5/target/debug$ cat test1234.txt 
test_1000
```

## Notes

1. This notation was new to me: `if let Err(e) = file1.read_to_end(&mut buffer)` - Gemini explained it well though:

`if let Err(e) = ...:` This is a pattern matching construct in Rust. It checks if the result of `read_to_end` is an `Err` variant. If it is, the code extracts the error value and assigns it to the variable e.

2. This notation I saw before in some examples but I only used it for the first time: `io::stdin().read_line(&mut input).expect("Failed to read line");`. Another great explanation from Gemini:

**`io::stdin()`:** This gets a handle to the standard input stream (stdin).

**`.read_line(&mut input)`:** This reads a line of input from stdin and appends it to the `input` string. The `&mut input` passes a mutable reference to the `input` string, allowing the `read_line` function to modify it.

**`.expect("Failed to read line");`:** This handles potential errors that might occur during the reading process. If an error occurs, the program will panic and display the provided error message.

Otherwise it was pretty straightforward, I was even surprised that it compiled and ran on a first try!


## Sources

* https://doc.rust-lang.org/beta/std/fs/fn.read_to_string.html
* https://doc.rust-lang.org/std/io/trait.Read.html#method.read_to_end
* https://doc.rust-lang.org/std/io/trait.Write.html#tymethod.write
* https://users.rust-lang.org/t/how-to-get-user-input/5176/2
* https://www.tutorialspoint.com/rust/rust_input_output.htm