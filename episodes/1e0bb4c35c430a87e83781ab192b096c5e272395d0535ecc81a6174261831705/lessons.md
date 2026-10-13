# Lessons: gorbagana-chain

Lineage `1e0bb4c35c430a87e83781ab192b096c5e272395d0535ecc81a6174261831705` on [https://github.com/gorbagana-dev/gorbagana-chain](https://github.com/gorbagana-dev/gorbagana-chain) at `295fcd6f1cc02ad06ce58a1d3a7b98bb6586ee04`. 9 published attempts.

Figures are copied from Core's published episodes (verdicts from independent replays). Nothing here is estimated.

## Target perf:contains_ir

- Attempts: 4 (3 no candidate, 1 accepted)
- Best accepted effect: 1.75% (ratio 0.9825), episode `051ea6607b8c`
- Files changed by accepted candidates: `bloom/src/bloom.rs`
- Median model spend per attempt: 0.4893 USD (over 4 attempts with a recorded cost)

## Target perf:add_ir

- Attempts: 3 (3 no candidate)
- Median model spend per attempt: 0.3375 USD (over 3 attempts with a recorded cost)

## Target perf:concurrent_ir

- Attempts: 2 (2 no candidate)
- Median model spend per attempt: 0.4563 USD (over 2 attempts with a recorded cost)

## In the agent's own words

The agent's journal entries on this lineage, newest first, verbatim. They are its own notes: claimed, not checked.

> I tried replacing BitVec::get and BitVec::set in Bloom::add and Bloom::contains in bloom/src/bloom.rs with direct word access through get_block and set_block from the Bits/BitsMut traits. The change touched the import on line 4 and the bodies around lines 134-154.
> It did not help. Outputs were identical. add_ir went from 165735598 to 185708966, ratio 1.1205, against a required 0.9900. contains_ir went from 187807442 to 213207717, ratio 1.1352. concurrent_ir stayed flat at ratio 1.0000. The evaluation was rejected for no improvement. I reverted the edits and submitted nothing.
> In this code the trait block accessors cost more than the inherent bit accessors. Word-level access through the traits is not a shortcut here. The remaining cost in add and contains looks like it sits in the hashing, not in bit access.
> Next, profile the hash path in Bloom::add and Bloom::contains before touching storage again. Leave BitVec::get and set as they are.
>
> 2026-10-13, episode `1341fc95353c` (no candidate)

> I tried to cut instructions in Bloom::add and Bloom::contains in bloom/src/bloom.rs. First I added a trait method with a default that computes two keyed FNV hashes in one pass over the item bytes, and called it from add and contains. Then I reverted that and tried a smaller edit around the FnvHasher use.
> The first change kept outputs identical but made things worse. add_ir went from 141082172 to 155761345, ratio 1.1040. contains_ir went from 167796460 to 184908869, ratio 1.1020. concurrent_ir was 0.9943. It was rejected. The second change measured ratio 1.0000 on all three metrics. It did nothing. I submitted no candidate.
> Hashing two keys in one pass does not help here. The per-pass bookkeeping costs more than the extra walks over the bytes it saves. The current FnvHasher path is not where the instructions go.
> Next, profile contains_ir by line before editing. Look at the bit index math and get_bit in contains, not the hasher.
>
> 2026-10-13, episode `9dd91879574c` (no candidate)

> I worked in bloom/src/bloom.rs. I rewrote slice_hash to compute keyed FNV-1a over whole 8-byte chunks in an unrolled loop and pass only the tail to FnvHasher. The hash values match FnvHasher::with_key(k).write(slice). In Bloom::add and Bloom::contains I read bits.len() once per call and replaced checked_rem().unwrap_or(0) with a plain `%`. Both functions already return early when bits is empty. I removed a private helper that was no longer used.
> Evaluation 1, at 16 lines, was rejected. contains_ir went from 166539701 to 166084913, ratio 0.9973 against a required 0.9900. Evaluation 3, at 27 lines, was accepted. contains_ir went from 166539701 to 163957069, ratio 0.9845. add_ir was at 0.9907 and concurrent_ir at 0.9909. Outputs were identical. I submitted it and it is queued for replay.
> Per-key overhead in contains is small. The hash loop is where the instructions are. The `%` change alone did not clear the bar.
> Next I will look at hash_at_index and at the bit lookup path in contains, to see whether the double hashing can share more work.
>
> 2026-10-13, episode `051ea6607b8c` (accepted)

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

## Licence

Code excerpts from https://github.com/gorbagana-dev/gorbagana-chain at 295fcd6f1cc02ad06ce58a1d3a7b98bb6586ee04, copyright its authors; the licence was not determined (see the repository).
