# Hints

## Rung 1 · cheap
Grid buffers are sized with `width * height`, and element access goes through
`y * stride + x`. Both width and height originate in the image header. Scan
`lib.rs`, `mutable_subgrid.rs`, and `shared_subgrid.rs` for bare `*` and `+`
on `usize` values that are attacker-influenced. On 32-bit platforms these wrap.

## Rung 2 · narrower
`AlignedGrid::with_alloc_tracker` computes `len = width * height` and then
`buf_len = len + pad` with no overflow check. If the product wraps small, a
tiny buffer is allocated while the decoder later believes the grid is huge.
`MutableSubgrid::from_buf` / `SharedSubgrid::from_buf` perform their area check
with `stride * (height - 1) + width` — the very arithmetic they are supposed to
be validating, also unchecked.

## Rung 3 · narrow
The pattern is the same everywhere: `y * stride + x` (in `get_ptr_unchecked`
in both subgrid files), `row * stride` (in the row-slice getters), and the
group walker's `gy * group_height` / `gx * group_width` / `num_cols *
num_rows` in `into_groups_with_fixed_count`. Any of these can wrap, yielding an
offset that lands inside the allocation for the wrong element — or past the
allocation entirely — which converts later writes into out-of-bounds writes and
makes `ptr.add(offset)` pointer UB.

## Rung 4 · spicy
The vulnerable arithmetic in `lib.rs`:
```rust
let len = width * height;
let buf_len = len + (Self::ALIGN - 1) / std::mem::size_of::<S>();
```
and in both subgrid files:
```rust
let offset = y * self.stride + x;
unsafe { self.ptr.as_ptr().add(offset) }
```
plus the fixed-count group walker's `y = (gy * group_height).min(height)` and
`row_ptr = unsafe { ptr.as_ptr().add(y * stride) }`.

## Rung 5 · giveaway
On a 32-bit build, open an image whose header claims e.g. `width = 2^31`,
`height = 2`. Then `width * height` wraps to `0`, a (near-)empty grid buffer is
allocated, and the decoder's sample writes (`y * width + x`-style indexing)
write past the real allocation. With `into_groups_with_fixed_count`,
overflowing `gy * group_height` or `gx * group_width` can wrap a later group
back onto the first row/column, producing two mutable subgrids over the same
memory (aliasing/UB), and an overflowing `num_cols * num_rows` just panics the
capacity allocation. The fix replaces the unchecked ops with
`checked_mul`/`checked_add`/`saturating_mul` and `wrapping_add` for
empty-boundary pointers.
