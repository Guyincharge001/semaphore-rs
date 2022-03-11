# Semaphore-rs

Rust support library for Semaphore

## Example

```rust
use semaphore::{
    identity::Identity, hash::Hash, poseidon_tree::PoseidonTree,
    protocol::* };
use num_bigint::BigInt;

// generate identity
let id = Identity::new(b"hello");

// generate merkle tree
const LEAF: Hash = Hash::from_bytes_be([0u8; 32]);

let mut tree = PoseidonTree::new(21, LEAF);
let (_, leaf) = id.commitment().to_bytes_be();
tree.set(0, leaf.into());

let root: BigInt = tree.root().into();
dbg!(root);

let merkle_proof = tree.proof(0).expect("proof should exist");
let root = tree.root().into();

// change signal and external_nullifier here
let signal = b"hello";
let external_nullifier = b"123";

let nullifier_hash = generate_nullifier_hash(&id, external_nullifier);

let config = SnarkFileConfig {
    zkey: "./snarkfiles/semaphore.zkey".to_string(),
    wasm: "./snarkfiles/semaphore.wasm".to_string(),
};

let proof = generate_proof(&config, &id, &merkle_proof, external_nullifier, signal).unwrap();
let success = verify_proof(&config, &root, &nullifier_hash, signal, external_nullifier, &proof).unwrap();
```
