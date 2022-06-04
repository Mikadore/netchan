# NetChan

NetChan is a Rust library implementing a custom protocol for sending 
structs over the network. It is influenced by Golang's Channels, and 
the various channel APIs in the stdlib and tokio.

It builds ontop the async tokio runtime, using Serde/Bincode for serialization, 
and aims to provide an easy to use and secure API. Sending data between clients
should be only marginally harder than between threads.

## Specification

Read the spec [here](docs/SPEC.md). The specification describes a protocol
designed specifically for the rust use case, with a specific API in mind.
However, nothing stops you from implementing the protocol yourself, and maybe 
even expand on it!

## Disclaimer

This is a school project not intended to be used in production.