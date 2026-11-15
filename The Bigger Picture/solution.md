# Vulnerability solution

## Finding

**Severity: High** — integer overflow can lead to out-of-bounds writes in `jxl-grid` on 32-bit platforms (CWE-190, CWE-131, CWE-122).

When decoding a crafted JPEG XL image, attacker-controlled dimensions can make the grid's `width * height` length calculation overflow `usize`. The grid may then allocate a buffer smaller than its logical dimensions. Later mutable subgrid access can use those dimensions and unchecked pointer arithmetic to write outside the allocation. This can cause memory corruption and, according to the advisory, may allow arbitrary code execution.

The vulnerable allocation calculation is in [`lib.rs`](files/crates/jxl-grid/src/lib.rs), and mutable subgrid access is in [`mutable_subgrid.rs`](files/crates/jxl-grid/src/mutable_subgrid.rs). The issue affects `jxl-grid` versions `0.6.1` and earlier; version `0.6.2` is patched.

## Fix

Use checked arithmetic for dimension, allocation-length, bounds, and pointer-offset calculations. Reject or safely handle dimensions that cannot be represented, and avoid constructing or dereferencing out-of-allocation pointers. The upstream fix is in [commit 3986dadd](https://github.com/tirr-c/jxl-oxide/commit/3986dadd3926a95656a642e74a0702d52f5c92e2).

## References

- [GitHub Security Advisory GHSA-5pmv-rx8r-wmv5](https://github.com/tirr-c/jxl-oxide/security/advisories/GHSA-5pmv-rx8r-wmv5)
- [Upstream fixing commit](https://github.com/tirr-c/jxl-oxide/commit/3986dadd3926a95656a642e74a0702d52f5c92e2)

The vulnerable source files have not been modified.
