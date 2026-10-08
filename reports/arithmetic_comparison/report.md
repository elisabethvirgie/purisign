# Python-versus-Verilog modular arithmetic

**Status: PASS** — 254/254 vectors emitted; 254/254 exact matches.

Icarus: `Icarus Verilog version 12.0 (stable) ()`; mode `-g2001 -Wall`.
Deterministic random seed: `0x54494e595349474e`.

| Module | p vectors | n vectors | Exact Python/RTL matches |
|---|---:|---:|---:|
| `mod_add` | 57/57 | 57/57 | 114/114 |
| `mod_sub` | 57/57 | 57/57 | 114/114 |
| `mod_inv` | 13/13 | 13/13 | 26/26 |

Each JSON vector records Python’s expected value, the RTL value, error/done status, exact-match flag, and cycle count.
Artifacts: [`results.json`](results.json), [`icarus.log`](icarus.log), [`vectors.txt`](vectors.txt).
