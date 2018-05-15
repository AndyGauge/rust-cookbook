## Avoid discarding errors during error conversions

[![error-chain-badge]][error-chain] [![cat-rust-patterns-badge]][cat-rust-patterns]

[Matching] on different error types returned by a function is possible
and relatively compact with [error-chain] crate. To do so,
[`ErrorKind`] is used to determine the error type.

To illustrate the error matching, a random integer generator web
service will be queried via [reqwest] and then the response will be
parsed. Errors can be generated by the Rust standard library,
[reqwest] and by the web service. [`foreign_links`] are used for well
defined Rust errors. The `errors` block of the `error_chain!` macro is
used to create an additional [`ErrorKind`] variant for the web service
error. Finally, a regular `match` can be used to react differently
according to the raised error.

```rust
#[macro_use]
extern crate error_chain;
extern crate reqwest;

use std::io::Read;

error_chain! {
    foreign_links {
        Io(std::io::Error);
        Reqwest(reqwest::Error);
        ParseIntError(std::num::ParseIntError);
    }

    errors { RandomResponseError(t: String) }
}

fn parse_response(mut response: reqwest::Response) -> Result<u32> {
    let mut body = String::new();
    response.read_to_string(&mut body)?;
    body.pop();
    body.parse::<u32>()
        .chain_err(|| ErrorKind::RandomResponseError(body))
}

fn run() -> Result<()> {
    let url =
        format!("https://www.random.org/integers/?num=1&min=0&max=10&col=1&base=10&format=plain");
    let response = reqwest::get(&url)?;
    let random_value: u32 = parse_response(response)?;

    println!("a random number between 0 and 10: {}", random_value);

    Ok(())
}

fn main() {
    if let Err(error) = run() {
        match *error.kind() {
            ErrorKind::Io(_) => println!("Standard IO error: {:?}", error),
            ErrorKind::Reqwest(_) => println!("Reqwest error: {:?}", error),
            ErrorKind::ParseIntError(_) => println!("Standard parse int error: {:?}", error),
            ErrorKind::RandomResponseError(_) => println!("User defined error: {:?}", error),
            _ => println!("Other error: {:?}", error),
        }
    }
}
```

[`ErrorKind`]: https://docs.rs/error-chain/*/error_chain/example_generated/enum.ErrorKind.html
[`foreign_links`]: https://docs.rs/error-chain/*/error_chain/#foreign-links

[Matching]:https://docs.rs/error-chain/*/error_chain/#matching-errors