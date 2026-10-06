# Changelog

Spec releases. Ports vendor a tagged release into `.spec/`; the version here is `spec_version` in `spec/capability.yaml`.

## 0.1.1 — 2026-10-06

No behavior change; conformance cases are identical (only their `spec_version` stamp changed).

- `bench/`: benchmark input (`bench.pmtiles`, `coords.txt`, checksum 998434), the shared method, and the Rust reference (`pmtiles` crate `=0.24.1` over an in-memory backend).

## 0.1.0 — 2026-10-06

First release: 5 core operations (`decode_header`, `decode_directory`, `zxy_to_tile_id`, `tile_id_to_zxy`, `find_entry`) and 2 io operations (`get_tile`, `read_metadata`); 12 error codes; 4 limits; 68 conformance cases (55 core, 13 io) built from two oracle-written archives; decisions D-001 to D-005.
