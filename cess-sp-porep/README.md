# CESS Storage Proofs PoRep

The `cess-sp-porep` is the reference implementation of _**Proof-of-Replication**_ (**PoRep**) and performs all the heavy lifting for `cess-proofs`.

_**Proof-of-Replication**_ proves that a Storage Miner is dedicating unique storage for each sector. The miners collect new client's data in a sector, run a slow encoding process (called `Seal`) and generate a proof (`SealProof`) that the encoding was generatd correctly.

PoRep provides two guarantees:
1. Space-hardness: Storage Miners cannot lie about tha amount of space they are dedicating to CESS Network in order to gain more power.
2. Replication: Storage Miners are dedicating unique storage for each copy of teir clients data. 

The _**Proof-of-Replication**_ uses Stacked DRG (SDR) designed by [**Ben Fisch at EUROCRYPT19**](https://eprint.iacr.org/2018/702.pdf). SDR uses Depth Robust Graph to ensure the sector has been encoded with a slow and non-parallelizable sequential process.

The proof size in SRD is too large to store it in blockchain this is mostly due to large amount of Merkle tree proofs required to achieve security. SDR verification algorithm is build using arithemetic circuit and uses SNARKs to prove that SDR proof was evaluated correctly.

## PoRep Circuit

### Overview of entire PoRep Circuit 
![1_8ngv4D_fB5WzauWAY3b2fA](https://user-images.githubusercontent.com/15224865/146317251-22961983-8b0f-49d9-8974-6c4a8de15995.png)

### StackedCircuit
StackedCircuit is the over all circuit of PoRep, defined in [proof.rs](./src/stacked/circuit/proof.rs#L28)
```rust
pub struct StackedCircuit<'a, Tree: 'static + MerkleTreeTrait, G: 'static + Hasher> {
    public_params: <StackedDrg<'a, Tree, G> as ProofScheme<'a>>::PublicParams,
    replica_id: Option<<Tree::Hasher as Hasher>::Domain>,
    comm_d: Option<G::Domain>,
    comm_r: Option<<Tree::Hasher as Hasher>::Domain>,
    comm_r_last: Option<<Tree::Hasher as Hasher>::Domain>,
    comm_c: Option<<Tree::Hasher as Hasher>::Domain>,

    // one proof per challenge
    proofs: Vec<Proof<Tree, G>>,
}
```

This includes
- public_params: StackedDrg (deep robust graph) related parameters, including the parameters of the graph itself and the number of challenges.
- replica_id: Sector copy id
- comm_d: the root of the binary tree of the original data
- comm_r: hash result of comm_r_last and comm_c
- comm_r_last: the root of the octree of the data after encoding
- comm_c: Root of the octree of column hash result
- proofs: Challenge the corresponding proof circuit



## License

MIT or Apache 2.0
