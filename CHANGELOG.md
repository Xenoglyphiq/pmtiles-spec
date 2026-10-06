# Changelog

Spec releases. Ports vendor a tagged release into `.spec/`; the version here is `spec_version` in `spec/capability.yaml`.

## 0.2.0 — 2026-10-06

Behavior changes in the io layer. Ports must re-sync and change two interim codes.

- **New error codes:**
  - `pmtiles.decompression_failed`: corrupt, cut-short or CRC-mismatched internal data. Ports previously reported `invalid_directory` for a directory and `truncated` for metadata (D-006).
  - `pmtiles.invalid_metadata`: metadata that isn't well-formed UTF-8 (D-007).
- **Limits:** `max_directory_bytes` and `max_metadata_bytes` now bound decompressed size too, and `max_directory_bytes` covers leaf directories (D-006).
- **Stated explicitly:** a short read is `pmtiles.truncated`; an overflowing offset sum is `pmtiles.invalid_directory`.
- **13 new io cases (81 total: 55 core, 26 io),** all on hand-built archives. They cover the codes and limits above, the leaf-depth limit (D-005; it had no fixture before), valid multi-byte UTF-8, and a tile past the end of the archive. Inflating archives use a hand-written fixed-Huffman gzip block, so regeneration stays byte-identical everywhere.
- `fetch_one_tile` example pinned to `small.pmtiles`, tile `2/1/3` (what every port already did).

## 0.1.1 — 2026-10-06

No behavior change; conformance cases are identical (only their `spec_version` stamp changed).

- `bench/`: benchmark input (`bench.pmtiles`, `coords.txt`, checksum 998434), the shared method, and the Rust reference (`pmtiles` crate `=0.24.1` over an in-memory backend).
- Reference and port timings recorded (`bench/README.md`); Nim, Swift and Zig ports at M3 in `capability.yaml` and the port table; Nim's dependency tier corrected to T1 (zippy).

## 0.1.0 — 2026-10-06

First release: 5 core operations (`decode_header`, `decode_directory`, `zxy_to_tile_id`, `tile_id_to_zxy`, `find_entry`) and 2 io operations (`get_tile`, `read_metadata`); 12 error codes; 4 limits; 68 conformance cases (55 core, 13 io) built from two oracle-written archives; decisions D-001 to D-005.
