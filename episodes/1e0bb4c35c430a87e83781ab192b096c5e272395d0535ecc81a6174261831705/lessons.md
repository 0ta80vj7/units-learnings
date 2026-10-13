# Lessons: gorbagana-chain

Lineage `1e0bb4c35c430a87e83781ab192b096c5e272395d0535ecc81a6174261831705` on [https://github.com/gorbagana-dev/gorbagana-chain](https://github.com/gorbagana-dev/gorbagana-chain) at `295fcd6f1cc02ad06ce58a1d3a7b98bb6586ee04`. 2 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:contains_ir

- Attempts: 1 (1 no candidate)
- Median model spend per attempt: 0.3684 USD (over 1 attempt with a recorded cost)

## Target perf:add_ir

- Attempts: 1 (1 no candidate)
- Median model spend per attempt: 0.4668 USD (over 1 attempt with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> Target was add_ir in bloom/src/bloom.rs. First I replaced bv get/set with get_block/set_block in Bloom::add and Bloom::contains. Then I tried a provided trait method, hash_at_indexes, to share the index computation between add and contains.
> Evaluation 1, the block access change: add_ir went from 140123731 to 155522234, ratio 1.1099. contains_ir ratio was 1.1250. Outputs were identical. It was rejected. Evaluation 3 did not build, and the stable tests failed, test_add_contains among them. Evaluation 5, the trait method: add_ir ratio was 1.0389, contains_ir 1.0045, concurrent_ir 0.9940. Outputs were identical. It was rejected. Neither change helped. I submitted nothing.
> In bv, block access through the Bits trait costs more than plain get/set. The index loop in Bloom::add is already tight. Adding indirection around it adds instructions.
> Next I would profile where add_ir is actually spent before editing again. The hashing may cost more than the bit writes. I would also check how much of add_ir comes from bench setup in lineage_bench.rs.
>
> 2026-10-13, episode `1327267a4097` (no candidate)

> I worked on contains_ir in bloom/src/bloom.rs, measured with bloom/examples/lineage_bench.rs. I made three attempts at the key hashing path. First, an FNV loop over 8-byte words. Second, a byte loop chunked by 8. Third, computing key hashes in pairs in Bloom::contains after the first key.
> The word loop took contains_ir from 190048016 to 265932590, ratio 1.3993. add_ir rose to ratio 1.4696. The chunked byte loop gave contains_ir 204906966, ratio 1.0782. The paired hashing gave 192154779, ratio 1.0111, with add_ir and concurrent_ir at 1.0000. All three were rejected. Outputs were identical each time. Nothing was submitted.
> The stock FnvHasher loop is already compiled tightly, likely unrolled for fixed-size arrays. Rewriting the hash loop by hand made it slower every time. I did not see a gain there.
> Next, leave the hash alone. Look at the per-key overhead around it, such as bit indexing in get_bit and modulo work, and profile it before editing.
>
> 2026-10-13, episode `ac01c5581786` (no candidate)

## Licence

Code excerpts from https://github.com/gorbagana-dev/gorbagana-chain at 295fcd6f1cc02ad06ce58a1d3a7b98bb6586ee04, copyright its authors; the licence was not determined (see the repository).
