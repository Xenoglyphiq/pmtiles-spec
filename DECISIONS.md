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
**Reproducibility:** the generator routes the oracle's `gzip.compress` through a gzip writer that uses deflate *stored* blocks. zlib and zlib-ng, which Python builds on different platforms use, compress identical input to different bytes, so archives generated on macOS and on the Linux CI runner differed. Stored blocks are byte-identical everywhere and are valid gzip for every decoder, so ports are still tested on real gzip parsing.
**Affects:** `conformance/`, spec §6.

### D-005 — Too-deep leaf nesting is an error
**Status:** Proposed
**Decision:** Following more than `max_leaf_depth` leaf pointers (default 4) is `pmtiles.leaf_depth_exceeded`. The root is depth 0.
**Why:** A cycle or a hostile chain should fail loudly. The oracle stops after 4 directory reads and returns "no tile", which hides the problem.
**Affects:** spec §3 `get_tile`, A6; all ports. Fixtures since 0.2.0: hand-built chains of 4 leaf pointers (allowed) and 5 (`leaf_depth_exceeded`).
**Revisit if:** the PMTiles spec defines a maximum depth.

### D-006 — Decompression failures get their own code, and limits bound decompressed size
**Status:** Accepted (2026-10-06, owner)
**Decision:** Internal data in a supported compression that doesn't decode (corrupt, cut short, or a gzip CRC-32 or length mismatch) is `pmtiles.decompression_failed`, for directories and metadata alike. `max_directory_bytes` and `max_metadata_bytes` bound the bytes both as stored and as decompressed, for leaf directories as well as the root. Related rules made explicit at the same time: a short read is `pmtiles.truncated`, and an overflowing offset sum is `pmtiles.invalid_directory`.
**Why:** 0.1.x had no code for a bad stream, so the three ports agreed on interim codes (`invalid_directory` for a directory, `truncated` for metadata). Both were misleading, and neither was in a fixture. Neither reference implementation helps: Python raises its gzip exceptions with no code, and `go-pmtiles` reports bad metadata as "unknown compression" and discards gzip errors in directories entirely. Without a decompressed-size bound, a few hundred bytes of gzip can expand to gigabytes.
**Affects:** spec §3 `get_tile` and `read_metadata`, §5, A8; all ports (they already enforced the limits after decompression; they change codes). Found by the Swift, Nim and Zig ports (Xenoglyphiq/pmtiles-spec#3).

### D-007 — Metadata must be well-formed UTF-8
**Status:** Accepted (2026-10-06, owner)
**Decision:** `read_metadata` rejects metadata that isn't well-formed UTF-8 (RFC 3629) with `pmtiles.invalid_metadata`. It checks nothing else: the JSON stays unparsed.
**Why:** The PMTiles spec says the metadata is JSON, and JSON is UTF-8. The ports disagreed (Swift replaced bad bytes with U+FFFD, Nim and Zig returned them as is), and so do the references: the oracle rejects it as a side effect of parsing the JSON, while `go-pmtiles` passes the bytes through, and where it does parse them, replaces bad ones silently. Rejecting means a string is always a faithful copy of the stored bytes and no port has to choose a repair.
**Affects:** spec §3 `read_metadata`, A9; all ports.
**Revisit if:** real archives turn up with non-UTF-8 metadata that readers are expected to accept.
