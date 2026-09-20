# `make check` reference

The Makefile parses the table below: directory in column 1, the command (starting with `./`) in column 2, and the space-separated expected collision counts at 2^20 seeds in column 3, in the order the program prints them. `make check` runs each command inside its directory, extracts the integers that follow "collisions" on the sampling lines, and compares them element by element; a program that exits non-zero fails even if its counts agree. Counts are exact because every program seeds its RNG from a fixed constant. `make check CHECK_TOL=n` allows a difference of up to `n` per count; `DIRS=<subset>` restricts any target.

| directory | command (run inside the directory) | expected collision counts |
|---|---|---|
| `a5hash` | `./a5hash_verify 20` | `0 156800 0 7291 1048576` |
| `cityhash-64` | `./cityhash64_verify 2^20` | `1048576 1048576 1048576 1048576 0` |
| `farmhash-64` | `./farmhash64_pairs 20` | `1048576 1048576 1048576 1048576 1048576 1048576` |
| `gxhash-64` | `./gxhash64_verify 20` | `1048576 1048576 1048576 1048576` |
| `highwayhash` | `./highwayhash_verify 20` | `0 0 1 1` |
| `komihash` | `./komihash_pair 20` | `954642 674611 280031 0` |
| `murmurhash3-128` | `./murmurhash3_128_verify 20` | `1048576 0 1048576 0 1048576` |
| `museair` | `./museair_verify 20` | `1048576 1048576 1048576 1048576 1048576 1048576 1048576 1048576 0` |
| `rust-ahash` | `./ahash_pairs 20` | `0 2 0 2 1 0 1 2 0 0` |
| `spookyhash2-64` | `./spookyhash2_64_pair 20` | `524207 524207 524369 524369 1048576 524103 524103 524103` |
| `t1ha2-64` | `./t1ha2_64_verify 20` | `0 57150 0 65785 0 65084` |
| `pengyhash` | `./pengyhash_verify 20` | `1048576 1048576` |
| `nmhash32` | `./nmhash32_verify 20` | `262968` |
| `nmhash32x` | `./nmhash32x_verify 20` | `1048576` |
| `mx3` | `./mx3_verify 20` | `1048576 1048576` |
| `mir` | `./mir_verify 20` | `1048576 1048576 1048576 1048576` |
| `fasthash` | `./fasthash_verify 20` | `1048576 1048576 1048576 1048576 1048576 1048576` |
| `mum` | `./mum_verify 20` | `1048576 1048576` |
| `rapidhash-v3` | `./rapidhash_v3_verify 20` | `0 0 0 0 0 0` |
| `wyhash` | `./wyhash_verify 20` | `0` |
| `rapidhash-v1` | `./rapidhash_v1_verify 20` | `0` |
| `xxh3-64` | `./xxh3_64_pair_check 20` | `0 725 725` |
| `museair-v2` | `./museair_v2_verify 20` | `3 3 0 0` |
| `xxh3-128` | `./xxh3_128_verify 20` | `1` |
| `go-maphash` | `./go_maphash_verify 20` | `65536 65536 0 0` |
| `dotnet-marvin` | `./marvin32_verify 20` | `2274 0` |
| `abseil-hash` | `./abseil_hash_verify 20` | `32 1048576 32 1048576 32 1048576` |

What the counts are:

* `a5hash`: pair 1 uniform, pair 1 class members colliding (all 156800), pair 2 uniform,
  pair 2 class members colliding (all 7291), pair 3 uniform.
* `cityhash-64`: pair A 64-bit seeds, pair A `(seed0, seed1)`, pair B 64-bit, pair B
  `(seed0, seed1)`, control pair (must be 0).
* `farmhash-64`: pairs A, B, C, each on `Hash64WithSeed` then `Hash64WithSeeds`.
* `gxhash-64`: pair 1 `gxhash_64`, pair 1 `gxhash` (128-bit), pair 2 `gxhash_64`, pair 2 `gxhash`.
* `highwayhash`: uniform keys (2^-56 is invisible at 2^20), class keys hashed in full (0.05
  expected), E_2 hits in the 2^24-key screen, and how many of those hits are full-state
  collisions (must equal the hits).
* `komihash`: pair 1 collisions, collisions given no carry (all), collisions given a carry
  (about 3/4), pair 2 uniform (0: one weak seed in 2^64).
* `murmurhash3-128`: pair 1, its control, pair 2, its control, the 4-way multicollision.
* `museair`: the 17-byte pair in the four variants, the 24-byte pair in the four variants, the
  control (0 of 4096).
* `rust-ahash`: A_2, A_1, B_0, historical A, fallback B; each in rs4 then correlated smh models.
* `spookyhash2-64`: pair 1 64-bit and 128-bit, pair 2 64-bit and 128-bit, seeds on which the
  bit-63 predictor was right (all); then selected 275-byte pair at 32, 64 and 128 bits.
* `t1ha2-64`: selected F60, historical A, historical B; each uniform then class.

* `wyhash`, `rapidhash-v1`: pair A.
* `xxh3-64`: selected 24/32/128-byte length checks; identical 16-byte prefixes are excluded.
* `museair-v2`: hash, bfast::hash, hash128, bfast::hash128.
* `xxh3-128`: pair F, full 128-bit equality (one hit in this fixed smoke stream).
* `go-maphash`: the constructed key under 65536 random map seeds (all), the same four key bytes with
  the other 124 bytes and the seed random (all), a control key (0), then the random (key, seed)
  sample (2^-24 is invisible at 2^20; the 2^30 run is in its README).
* `dotnet-marvin`: pair A (12-byte strings, L = 2), then pair B (8-byte, L = 1; 2^-22.5 is invisible
  at 2^20).
* `abseil-hash`: for each of pairs 1, 2, 3: the 32 SwissTable seeds (all collide), then the 2^20
  uniform seeds (all collide).

Counts justified by an every-seed identity remain N for any RNG seed. Other reference counts, including zero-hit rare-event samples, can change with the RNG seed. The exact check uses the fixed default stream.

## What every program has in common

* **Validation before anything else.**  The SMHasher3 verification value of the exact
  registered variant is recomputed with the `lib/Hashinfo.cpp` `_ComputedVerifyImpl`
  procedure (keys `{}`, `{0}`, `{0,1}`, … of length 0..255 with seed `256 - i`, the
  encoded outputs concatenated and hashed with seed 0, first four bytes little-endian;
  XXH3 uses canonical big-endian output encoding). Rapidhash-v1 instead checks the
  local harness’s recorded reference vector; the local SMHasher3 registration is v3.
  Where upstream ships test vectors they are checked too (komihash's README vectors, t1ha's 81
  known answers, HighwayHash's 33-byte vector, Peters' FarmHash string).  Any mismatch ends the
  run with a non-zero status before a pair is touched; each README states the code.
* **The published data are asserted, not just printed.**  Explicit seeds or keys from the
  records must reproduce the published hash values, pair recipes are re-derived where there is
  one (CityHash pair B, gxhash, MuseAir), and pairs that should collide on every seed fail the
  run if a single sampled seed does not.
* **Deterministic sampling.**  All programs use their own splitmix64-seeded xoshiro256** with a
  fixed default seed, overridable from the command line; the sub-README says which output lines
  change with it.
* **Nothing is written to disk** and nothing outside the directory is read.  Building per a
  sub-README's `cc … -o` line, or with this Makefile, leaves only the binary next to the source.
* **Arguments are validated**: non-numeric or out-of-range values are rejected with a usage
  message and a non-zero status rather than silently taken as 0.

## Licensing

* **Original driver code is MIT.** The original eleven programs’ driver additions, READMEs
  and this file: Copyright (c) 2026 Thomas Dybdahl Ahle, MIT License.  Each source file carries
  the full MIT text in its header.
* **komihash** (`komihash/komihash_pair.c`) embeds `komihash.h` release 5.34 **verbatim** (MIT,
  Copyright (c) 2021-2026 Aleksey Vaneev; sha256 of the embedded block equals upstream's, see that
  README).  Its license notice is inside the embedded block.
* **HighwayHash** (`highwayhash/highwayhash_verify.c`) embeds the reference `c/highwayhash.c`
  **verbatim** (Apache License 2.0, Copyright 2017 Google Inc.), with its first line (an
  `#include`) replaced by a comment; the Apache notice and the statement of that one
  modification are in the file header.
* **Go map hash** (`go-maphash/`) keeps verbatim copies of four go1.27.1 runtime files under `upstream/`
  (Copyright The Go Authors, BSD 3-Clause, `upstream/LICENSE`); the C transcription of the assembly is a
  derivative under that notice.
* The other hashes are re-implemented from their references and validated against them.  Where
  the code is close enough to count as a derivative the upstream copyright line is reproduced
  next to ours: CityHash (Google, Inc. 2011, MIT), FarmHash (Google, Inc. 2014 and Frank J. T.
  Wojcik 2021-2022, MIT), gxhash (Frank J. T. Wojcik 2025 and Olivier Giniaux 2023, MIT), a5hash
  (Aleksey Vaneev 2025 and Frank J. T. Wojcik 2021-2025, MIT), aHash (Tom Kaitchuck 2018, MIT OR
  Apache-2.0, two constants), and t1ha (Positive Technologies / Leonid Yuriev 2016-2020, zlib:
  the self-check table, test pattern and probe schedule, with the zlib notice reproduced).
  MurmurHash3 and SpookyHash are public domain; MuseAir is CC0 1.0.


## Integration pass 2

The added standalone programs credit the supplied independent implementations
under `experiment/verify-*/` and the validated driver/records under
`paper_rows/`. Per-hash READMEs include complete expected output. The new RNG
uses one stream per case (default seed 1), so these small checks are distinct
from the earlier large, sometimes multithreaded measurements. No large counts
are silently replaced by small-run estimates.

Added-case count order is exactly the order of each sub-README's pairs table.
All new programs use explicit little-endian loads. NMHASH32/32X use unsigned
32-bit intermediates for narrow products. The exact multiply implementations
require the GCC/Clang 128-bit integer extension.

Pengyhash v0.3 code is GPLv3-or-later (the supplied notice is retained), NMHASH
is BSD 2-Clause, mx3 is CC0, and the other added algorithm notices are retained
in their source files. These are not relicensed by the MIT driver additions.
The full supplied pengyhash license is included as `pengyhash/COPYING`.

The selected 275-byte SpookyHash pair has L=35 and estimated cap 6.13; it
collides at all three output widths at the same measured rate. The earlier
286-byte examples remain in the program. Exact half balance of either
predictor class is unproved, even though the older two classes complement
one another. Native Hash32's 32-bit seed domain differs from the scored
SMHasher3 duplicated 64-bit seed model.

## Completed dependency additions

The four formerly blocked directories now contain standalone C programs and full
EXPECTED output: [wyhash](wyhash/README.md), [rapidhash-v1](rapidhash-v1/README.md),
[XXH3-64](xxh3-64/README.md), and [XXH3-128](xxh3-128/README.md).
Wyhash and rapidhash-v1 transcribe the supplied harness; both XXH3 files embed
xxHash 0.8.3 verbatim with its BSD 2-Clause license. All four default to 2^20
seeds and verify a recorded collision separately. This is only a smoke run;
the 2^30 measurements in the index and sub-READMEs remain historical evidence.
The full package check output is recorded in [REPORT.md](../REPORT.md).

## Scope of the polished package

The 23 primary programs reproduce their deterministic README counts. Matching
known answers checks consistency; it does not prove full equivalence to upstream
on every input. The archive also includes a5hash `selected_pairs.c`, MuseAir
`additional_pairs.c`, and the exact carry-state class counter `count_class.py`.
Their per-hash READMEs give commands and limits.

Sampled rates assume the deterministic pseudorandom streams behave like independent
uniform draws. A zero count is not a hard population bound. Under that assumption,
0/N gives an approximate one-sided 95% upper limit of 3/N. Historical large runs
and exact-count claims are identified separately from the small default checks.

Page scores consistently use L = ceil(max(byte lengths)/8), allowing unequal
lengths. Exact caps round upward; empirical estimates round to nearest.

Numbers pass: [XXH3-64 at 32 bytes](xxh3-64/CHECK_32B.md), program [xxh3_64_32B_check.c](xxh3-64/xxh3_64_32B_check.c); [HalftimeHash independent execution harness](halftime/README.md).
