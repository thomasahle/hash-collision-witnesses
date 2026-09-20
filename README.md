# Collision witnesses

Reproduction programs for the collision pairs in the write-up
[Adversarial examples for fast hash functions](https://thomasahle.com/blog/adversarial-examples-for-hashes/).
The post defines the terms used here (the key model, what a *pair* is, how a *rate* is
measured and turned into a score) and discusses each hash; this repository only holds the
code that lets you check the numbers yourself.

Each directory covers one hash (or one separately tested version or output width) and holds a
single C11 program plus its own README. The program embeds or re-implements the hash, checks
itself at startup against the hash's published verification value or test vectors (and exits
non-zero on any mismatch), prints the colliding messages, hashes them under millions of random
seeds from a fixed-seed RNG, and prints the collision count and rate together with explicit
colliding seeds and both hash values. The per-directory READMEs give the exact source version,
the build line and the expected output.

```sh
make            # build every program in place
make check      # run each with 2^20 seeds and compare against CHECKS.md
make clean
```

A C11 compiler and `libm` are enough; two programs use POSIX threads and three have an optional
hardware-AES path. Programs write nothing but stdout/stderr. The expected counts that
`make check` compares against are in [CHECKS.md](CHECKS.md).
