# Contributing to PMTiles

Thanks for helping. This repo holds the spec and conformance cases that every language port of PMTiles implements. The ports live in their own repos (`Xenoglyphiq/pmtiles-<language>`).

## How this repo works

- **`spec/`** defines behavior. `SPEC.md` is the readable version; `capability.yaml` is the machine-readable contract. Ports implement the spec; they never reinterpret it.
- **`conformance/`** holds test cases generated from a pinned reference implementation. Every port must pass them.
- **Ports** live in separate repos and vendor a tagged release of this repo into their `.spec/` folder.
- **`.kit/`** holds shared conventions, schemas and the validator used by every library in this family. Don't edit it here; it's updated from upstream.

Read `.kit/CONVENTIONS.md` before changing anything: it covers data conventions, the error model, layers and canonical types.

## Making a change

| You want to… | Do this |
|---|---|
| Fix a bug in one port | Open the PR in that port's repo. If the bug shows a missing test, add a conformance case here too |
| Change behavior | Open an issue first. Behavior changes need a `DECISIONS.md` entry, a `spec_version` bump, regenerated fixtures and a release tag; each port then updates in its own repo |
| Propose a new port | Open an issue here; a port is listed in the README once it passes all `core` cases |
| Report a mismatch between ports | Open an issue with the input that behaves differently; it becomes a conformance case |

## Checks every PR must pass

1. `python .kit/validate.py .` (needs `pip install pyyaml jsonschema`)
2. Fixture generation is reproducible: re-running `conformance/generate/` with the pinned oracle gives identical files

## Style

- Follow the language's own conventions for names, errors and packaging.
- Errors keep the kinds and codes from the spec; tests assert kind and code, never message text.
- Every public item has a doc comment naming the spec operation it implements.

## License

By contributing you agree your contribution is licensed under MIT OR Apache-2.0, the same as this project.
