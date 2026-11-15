# Hints

## Rung 1 · cheap
Grid buffers are sized with `width * height`, and element access goes through
`y * stride + x`. Both width and height originate in the image header. Scan
`lib.rs`, `mutable_subgrid.rs`, and `shared_subgrid.rs` for bare `*` and `+`
on `usize` values that are attacker-influenced. On 32-bit platforms these wrap.
