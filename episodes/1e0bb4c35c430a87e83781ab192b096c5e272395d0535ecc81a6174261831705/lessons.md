# Lessons: gorbagana-chain

Lineage `1e0bb4c35c430a87e83781ab192b096c5e272395d0535ecc81a6174261831705` on [https://github.com/gorbagana-dev/gorbagana-chain](https://github.com/gorbagana-dev/gorbagana-chain) at `295fcd6f1cc02ad06ce58a1d3a7b98bb6586ee04`. 6 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:contains_ir

- Attempts: 2 (2 no candidate)
- Median model spend per attempt: 0.4800 USD (over 2 attempts with a recorded cost)

## Target perf:add_ir

- Attempts: 2 (2 no candidate)
- Median model spend per attempt: 0.4021 USD (over 2 attempts with a recorded cost)

## Target perf:concurrent_ir

- Attempts: 2 (2 no candidate)
- Median model spend per attempt: 0.4563 USD (over 2 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I read bloom/src/bloom.rs and the bench in bloom/examples/lineage_bench.rs. I searched for the concurrent paths and for the BitVec conversion, truncate and into_boxed_slice functions, looking for overhead outside hashing. I made no edits and ran no sandbox evaluations.
> Nothing was measured this session. Most of the instruction count in the concurrent benchmark is FNV hashing, done once per key per item. The big filter uses 27 keys over 64-byte items. The 24 small filters use 3 to 5 keys over 32-byte items. The FnvHasher byte loop looks close to its floor of three instructions per byte. Earlier measured attempts made every metric worse. I submitted no candidate.
> I now think the cost here is the number of hash passes, not the code that performs each pass. Making each pass a little faster will not move the totals.
> Next time, try computing one hash per item and deriving all the keys from it. That would have to be checked to see whether bloom.rs semantics allow the bit positions to change.
>
> 2026-10-13, episode `4e687659887f` (no candidate)

> I edited bloom/src/bloom.rs in two places. In Bloom::add, Bloom::contains and ConcurrentBloom::pos I replaced checked_rem with unwrap_or by a plain %. I also read bits.len() once per call instead of once per key. All three changes were measured together.
> Build was ok on base and candidate. Outputs were identical. add_ir went from 168218561 to 168218574, ratio 1.0000. contains_ir went from 218634265 to 217902906, ratio 0.9967. concurrent_ir was 0.9997. The required ratio is 0.9900, so the evaluation rejected it as no_improvement. The change did not help enough. I submitted nothing.
> The remainder and the length read are not where the instructions go in this code. The checked_rem path appears to cost almost nothing in add. Earlier attempts at rewriting the hash loop did not reach the limit either.
> Next I will profile add_ir per function before editing anything. hash_at_index and the set_bit and get_block paths need to be looked at first. Small edits to the modulo path are done.
>
> 2026-10-13, episode `525f844e2497` (no candidate)

> I worked on contains_ir in bloom/src/bloom.rs. First I replaced checked_rem/unwrap_or with a plain `%` in Bloom::add and Bloom::contains, since bits is known to be non-empty there. Then I unrolled the FNV byte loop 4 bytes at a time. Then I added #[inline] to slice_hash and hash_at_index. A fourth variant was tried as evaluation 7.
> The `%` change measured contains_ir 173378403 to 172174586, ratio 0.9931, with identical outputs. That is a real gain but under the 0.9900 minimum, so it was rejected. The unrolled FNV loop gave 1.1261 on contains_ir and 1.1866 on add_ir. The #[inline] change gave 1.0638. Evaluation 7 gave 1.2156. None of these helped. I submitted no candidate.
> The remainder in add and contains costs a little and can go. The FNV loop and the hash helpers already compile well. Forcing inlining or unrolling there makes every metric worse.
> Next, start from the plain `%` change and look for a second small cut in Bloom::contains. One option is hoisting bits.len() out of the per-key loop. Measure that before touching hashing again.
>
> 2026-10-13, episode `2f603e0118dc` (no candidate)

> Target was concurrent_ir in bloom/src/bloom.rs. I tried three changes. First, hashing four keys per pass over the item bytes in ConcurrentBloom::add and Bloom::add. Second, the same with two scalar lanes. Third, loading the word before the atomic fetch_or, so the read-modify-write is skipped when the bit is already set.
> All three builds passed with identical outputs. All three were rejected. The four-key pass gave concurrent_ir 1.0359, add_ir 1.0647 and contains_ir 1.0088. Two lanes gave concurrent_ir 1.0855 and add_ir 1.0848. The load-before-fetch_or change left add_ir and contains_ir at 1.0000, but concurrent_ir rose to 1.0162. None of them helped. I submitted nothing.
> The per-key hashing loop is not where the cost is that batching can remove. Fusing it added instructions. The extra load costs more than the skipped atomics save on this bench.
> Next, read lineage_bench.rs for the bit-set hit rate before touching the atomics again. Then look at index computation instead of hashing.
>
> 2026-10-13, episode `d3cf018d058b` (no candidate)

> Target was add_ir in bloom/src/bloom.rs. First I replaced bv get/set with get_block/set_block in Bloom::add and Bloom::contains. Then I tried a provided trait method, hash_at_indexes, to share the index computation between add and contains.
> Evaluation 1, the block access change: add_ir went from 140123731 to 155522234, ratio 1.1099. contains_ir ratio was 1.1250. Outputs were identical. It was rejected. Evaluation 3 did not build, and the stable tests failed, test_add_contains among them. Evaluation 5, the trait method: add_ir ratio was 1.0389, contains_ir 1.0045, concurrent_ir 0.9940. Outputs were identical. It was rejected. Neither change helped. I submitted nothing.
> In bv, block access through the Bits trait costs more than plain get/set. The index loop in Bloom::add is already tight. Adding indirection around it adds instructions.
> Next I would profile where add_ir is actually spent before editing again. The hashing may cost more than the bit writes. I would also check how much of add_ir comes from bench setup in lineage_bench.rs.
>
> 2026-10-13, episode `1327267a4097` (no candidate)

## Licence

Code excerpts from https://github.com/gorbagana-dev/gorbagana-chain at 295fcd6f1cc02ad06ce58a1d3a7b98bb6586ee04, copyright its authors; the licence was not determined (see the repository).
