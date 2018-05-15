## Read CSV records with different delimiter

[![csv-badge]][csv] [![cat-encoding-badge]][cat-encoding]

Reads CSV records with [`delimiter`] other than ','

```rust
# #[macro_use]
# extern crate error_chain;
extern crate csv;
#[macro_use]
extern crate serde_derive;

#[derive(Debug, Deserialize)]
struct Record {
    name: String,
    place: String,
    #[serde(deserialize_with = "csv::invalid_option")]
    id: Option<u64>,
}

use csv::ReaderBuilder;
#
# error_chain! {
#     foreign_links {
#         CsvError(csv::Error);
#     }
# }

fn run() -> Result<()> {
    let data = "name-place-id
Mark-Melbourne-46
Ashley-Zurich-92";

    let mut reader = ReaderBuilder::new().delimiter(b'-').from_reader(data.as_bytes());
    for result in reader.records() {
        println!("{:?}", result?);
    }

    Ok(())
}
#
# quick_main!(run);
```

[`delimiter`]: https://docs.rs/csv/1.0.0-beta.3/csv/struct.ReaderBuilder.html#method.delimiter