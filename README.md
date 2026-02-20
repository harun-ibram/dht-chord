# CHORD (MPI) Implementation

This repository contains an MPI-based implementation of a static CHORD-style Distributed Hash Table (DHT), using finger-table routing.

## Project contents

- [src/tema2.c](src/tema2.c) - main implementation
- [src/Makefile](src/Makefile) - build rules for the `tema2` executable
- [checker/checker.sh](checker/checker.sh) - local automated test runner
- [checker/check_lookup.py](checker/check_lookup.py) - validates lookup correctness and hop count
- [checker/tests](checker/tests) - test inputs and expected outputs
- [local.sh](local.sh) - helper script for local checker and Docker workflows
- [Dockerfile](Dockerfile) - container image with OpenMPI dependencies

## Implemented model

- Fixed CHORD identifier space: $m = 4$, so $2^m = 16$ possible IDs.
- Static topology: all nodes are known at startup (no dynamic `join`/`leave`).
- Each MPI process represents one CHORD node.
- Lookup routing is built around:
  - `build_finger_table()`
  - `closest_preceding_finger()`
  - `handle_lookup_request()`
- Communication is done using MPI messages with these tags:
  - `TAG_LOOKUP_REQ`
  - `TAG_LOOKUP_REP`
  - `TAG_DONE`

The intended routing complexity is $O(\log N)$.

## Input format

Each process with rank `R` reads `inR.txt` from the current working directory.

Format:
1. `node_id`
2. `nr_lookups`
3. `key_1`
4. `key_2`
5. ...

## Output format

For each locally initiated lookup, the program prints:

`Lookup K: n0 -> n1 -> ... -> target`

where:
- `K` is the queried key
- `target` is the responsible successor node

## Build

From [src](src):

- `make build` - builds `tema2`
- `make clean` - removes build artifacts

## Manual run

1. Build in [src](src).
2. Place `in0.txt`, `in1.txt`, ... in the directory where you run the binary.
3. Run with MPI using a process count equal to the number of input files.

Generic command example:

`mpirun -np <N> ./tema2`

## Run tests locally

From [checker](checker):

- `./checker.sh`

The checker:
- builds the project,
- runs tests `test1 ... test8`,
- validates the final successor (correctness),
- validates hop thresholds (efficiency).

## Docker utilities

The [local.sh](local.sh) script supports:

- image build,
- checker run in container,
- interactive container session,
- image push.

Help:

`./local.sh --help`

## Notes

- [src/backup](src/backup) is auxiliary and not required for running the checker.
- The executable expected by the checker is `tema2`.