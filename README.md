# PMTiles

Read PMTiles v3 single-file tile archives: header, directories, tile lookup and tile bytes. Implements [PMTiles v3](https://github.com/protomaps/PMTiles/blob/main/spec/v3/spec.md), read only.

The spec and conformance cases for PMTiles. Each language port lives in its own repo and is tested against the same cases.

## Ports

| Language | Repo | Package | Spec pinned | Conformance | Status |
|---|---|---|---|---|---|
| Nim | [`Xenoglyphiq/pmtiles-nim`](https://github.com/Xenoglyphiq/pmtiles-nim) | `pmtiles` | 0.1.1 | core ✓ io ✓ full ✓ (68/68) | feature-complete; first release pending |
| Swift | [`Xenoglyphiq/pmtiles-swift`](https://github.com/Xenoglyphiq/pmtiles-swift) | `PMTiles` | 0.1.1 | core ✓ io ✓ full ✓ (68/68) | feature-complete; first release pending |
| Zig | [`Xenoglyphiq/pmtiles-zig`](https://github.com/Xenoglyphiq/pmtiles-zig) | `pmtiles` | 0.1.1 | core ✓ io ✓ full ✓ (68/68) | feature-complete; first release pending |

Install instructions and examples are in each port's repo.

## What's here

| Path | What |
|---|---|
| `spec/SPEC.md` | Behavior spec |
| `spec/capability.yaml` | Machine-readable contract: types, operations, errors, limits |
| `conformance/` | Test cases every port must pass; regenerate with `uv run conformance/generate/generate.py` |
| `bench/` | Shared benchmark input, method and the Rust reference |
| `.kit/` | Shared conventions, schemas and validator (vendored) |
| `CONTRIBUTING.md` | How changes to the spec are made |
| `DECISIONS.md` | Why the spec is the way it is |
| `CHANGELOG.md` | What each spec release changed |

## Contributing

See `CONTRIBUTING.md`.

## License

MIT OR Apache-2.0
