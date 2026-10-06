# PMTiles — Decisions

Spec-level decisions. Newest at the bottom. Status is **Accepted** unless noted.

### D-001 — Decompression lives in io, not core
**Status:** Proposed
**Decision:** `decode_directory` takes already-decompressed bytes. The io layer supplies a `Decompressor`; each port declares which codecs it supports (gzip at minimum).
**Why:** Core must be dependency-free in every language, and not every standard library ships gzip (Swift without Foundation; Zig's compression APIs change between releases).
**Affects:** spec §3 `decode_directory`, `get_tile`, `read_metadata`; all ports.
**Revisit if:** every target language gains a stable built-in gzip decoder.

### D-002 — Reject repeated tile ids and a "continue" offset on the first entry
**Status:** Proposed
**Decision:** In `decode_directory`, a tile-id delta of 0 after the first entry, or a stored offset of 0 (which means "continue from the previous entry") on the first entry, is `pmtiles.invalid_directory`. So is any id, length or offset that overflows its type.
**Why:** Binary search assumes strictly increasing ids, and the first entry has nothing to continue from. The oracle accepts both and computes offset −1, which would read before the tile data section. Writers never produce either.
**Affects:** spec §3 `decode_directory`, A3; fixtures `directory.error.duplicate_tile_id`, `directory.error.first_offset_zero`; all ports.
**Revisit if:** a real writer produces either form.

### D-003 — A leaf pointer covers every id up to the next entry
**Status:** Proposed
**Decision:** In `find_entry`, when no entry matches exactly, the last entry below the target matches if it is a leaf pointer (`run_length == 0`), whatever its `length`. This is the oracle's behavior.
**Why:** A leaf directory covers a contiguous id range that ends where the next root entry begins. The outline draft's "whose range contains the id" was looser than what implementations do.
**Affects:** spec §3 `find_entry`, A4; fixture `find.leaf_pointer`; all ports.

### D-004 — The oracle builds the archives; the spec covers what it can't
**Status:** Proposed
**Decision:** Python `pmtiles==3.8.1` writes both test archives and computes the expected output for valid input. Cases are written from the spec, marked `source: "spec"`, when:
- they are errors (the oracle raises generic exceptions without codes);
- they are where we differ from the oracle (D-002, D-005);
- they need archives the writer can't produce: uncompressed, or an unknown internal compression.

The generator cross-checks every oracle result against a transcription of spec §3.
**Why:** The oracle's writer always gzips internal data, and its reader has no error codes or limits.
**Affects:** `conformance/`, spec §6.

### D-005 — Too-deep leaf nesting is an error
**Status:** Proposed
**Decision:** Following more than `max_leaf_depth` leaf pointers (default 4) is `pmtiles.leaf_depth_exceeded`. The root is depth 0.
**Why:** A cycle or a hostile chain should fail loudly. The oracle stops after 4 directory reads and returns "no tile", which hides the problem.
**Affects:** spec §3 `get_tile`, A6; all ports. No fixture yet, because writers only ever nest one level. A hand-built nested archive is a candidate for 0.1.x.
**Revisit if:** the PMTiles spec defines a maximum depth.

