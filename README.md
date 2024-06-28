# Substreams Challenge

📚 This challenge is meant for developers with a basic understanding Rust 🦀 and Web3.

Do you want to use substreams but don't know Rust? You only need a subset of Rust!

- [The Book](https://doc.rust-lang.org/book/index.html)
  Just read and understand chapters 1-9, maybe 10, plus chapters 13 and 18.
- [Rustlings](https://rustlings.cool/) is great to apply what you've learned!

🌊 Substreams are a powerful way to process blockchain data efficiently, allowing developers
to stream, transform, and analyze large volumes of on-chain data in real-time.

For a basic introduction to Substreams, watch this [video](https://www.youtube.com/watch?v=fogh2D-vpzg&t=2122s)

For a quicker, more applicable overview watch [this](https://www.youtube.com/watch?v=vWYuOczDiAA&t=27s)

---

Create a simple Substreams:

You'll be using a template Substreams to filter through blockchain data and indexing the transaction volume of NFTs.

Then you'll be outputting the target data into a subgraph.

Finally you'll query the subgraph in a template frontend to display the data with swag. 😎

## Checkpoint 0: 📦 Environment 📚

Before you begin, you need to install the following tools:

(for Scaffold-ETH)

- [Node (>= v18.17)](https://nodejs.org/en/download/)
- Yarn ([v1](https://classic.yarnpkg.com/en/docs/install/) or [v2+](https://yarnpkg.com/getting-started/install))
- [Git](https://git-scm.com/downloads)

(for Substreams)

- [Rust, Buf, and Substreams CLI](https://substreams.streamingfast.io/documentation/consume/installing-the-cli#dependency-installation)
- [Authentication and API key](https://substreams.streamingfast.io/documentation/consume/installing-the-cli#dependency-installation)
  We recommend using the "All-in-one bash function" to make life easier 🤙

  ***

  > You'll need to install Scaffold-ETH with

  ```sh
  yarn install
  ```

Complete the challenge however you want, as long
as you populate the pb with the required data.
Just follow our steps if you want a more guided experience.

# Checkpoint 1: map_module

### 1.1 Making a Protobuf

- Protobufs are a language-agnostic way to serialize structured data.
- Substreams use protobufs to carry data through their modules, so we need to define our protobufs in accordance to the data we want.
- [Protobufs](https://substreams.streamingfast.io/documentation/develop/creating-protobuf-schemas#protobuf-definition-for-substreams) from the Streamingfast docs.
- In substreams > proto > contract.proto, make sure your file looks like this:

```proto
syntax = "proto3";

import "google/protobuf/timestamp.proto";

package contract.v1;

message Transfer {
string address = 1;
string name = 2;
string symbol = 3;
}

message Transfers {
repeated Transfer transfers = 1;
}
```

In this challenge your first map_module will return a protobuf called `Transfers`.

Your `Transfers` protobuf is a vector of `Transfer` protobufs.

When returning a protobuf, you always need to return a `Protobuf` that contains a vector of `Protobufs`.

Because substreams index entire blocks at a time before moving to the next block, you need to be able to return multiple protobufs.

### 1.2 Updating the Yaml

In substreams_challenge > substreams.yaml, you'll find the outline of the project structure.
When adding new modules, you'll need to specify its structure in the `substream.yaml`.
There are two kinds of modules you can read about [here](https://substreams.streamingfast.io/documentation/develop/manifest-modules#module-kinds)

The map_module has mostly been filled out.

- [ ] For the name field put `map_events`
- [ ] For the kind field put `map`.

- Your first map_module will always take in blocks, so the `inputs` field needs `- source: sf.ethereum.type.v2.Block`.

Downstream, your map_module's `input` field can take in `-map: (name of map)` or `-store: (name of store)`.
It is best practice to only take in `sf.ethereum.type.v2.Block` in your initial map_module.

### 1.3 Building the map_module 🏗️

- [ ] Go to `substreams_challenge > src > lib.rs`.

> Every module needs a handler above it: `#[substreams::handlers::(map or store)]` so that your yaml finds the module.

##### What is filled out:

- Your `map_events` module takes in `blk: eth::Block` (blocks).
- The module returns : `Result<Transfers, substreams::errors::Error>`.
  > map_modules always return [Result Types](https://doc.rust-lang.org/rust-by-example/error/result.html).
- `token_meta` is a helper that makes RPC calls to fetch token `name` and `symbol`.
  > Take a look at rpc.rs if you're curious about how RPC calls work.
- The `Transfer` protobuf is instantiated for you with `name` and `symbol` populated from `token_meta`. In the `address` field `Hex::Encode()` is provided to conveinently convert the address (most likely a `Vec<u8>`) to a hexadecimal string.
- The `Transfers` protobuf (what the module returns) has also been instantiated.
- At the top of file we have imported the `TransferEvent` type for you to use.

##### Goal of the module

The module should search the block for all ERC721 transfer events, populate the `Transfer` protobuf with the event address, and populate the `Transfers` protobuf with a vector of `Transfer` protobufs.

---

- [ ] Look at the [available methods](https://docs.rs/substreams-ethereum/latest/substreams_ethereum/pb/eth/v2/struct.Block.html#implementations) on the Block Struct

  > `transactions()`, `reciepts()`, `logs()`, `calls()`, and `events()`, these will allow you to iterate over the block's data.

- [ ] Look at the [Event Trait](https://docs.rs/substreams-ethereum/latest/substreams_ethereum/trait.Event.html) for helpful methods to deal with events.

- [ ] TODO 1: Search the block to find all events that match the `TransferEvent`.

- [ ] TODO 2: Pass the address that emmitted the event into `token_meta` so it can make the calls to the correct address.

- [ ] TODO 3: Assign the `address` field on the protobuf the event address.

- [ ] TODO 4: Assign the `transfers` field on the `Transfers` protobuf the vector of `transfer` protobufs.
