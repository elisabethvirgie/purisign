# Repository verification inventory

The workspace contains multiple related snapshots and unrelated reference projects. This evidence package and the PURI-Sign README targets the maintained Verilog-2001 implementation in this project directory. A sibling alternate SystemVerilog snapshot is not the source used by the maintained validation scripts. Other workspace folders contain separate examples and are outside this project scope.

| Area | Location | Contents / evidence |
|---|---|---|
| Arithmetic RTL | `rtl/arithmetic/` | Modular add/subtract/multiply/inverse and Montgomery multiply |
| ECC RTL | `rtl/ec/`, `rtl/ecdsa/` | Point operations, scalar multiplication, ECDSA signer |
| Hash and nonce RTL | `rtl/sha256/`, `rtl/nonce/` | SHA-256, HMAC-SHA-256, RFC 6979 |
| Signing and security | `rtl/security/` | Key manager and register core (legacy module identifier) |
| Board wrapper/support | `rtl/platform/`, `rtl/test_support/` | Avalon-MM wrapper, self-test, UART support |
| Testbenches | `tb/` | Arithmetic, ECC, crypto, register, wrapper and self-test benches |
| Python reference | `python/`, `tests/` | P-256, RFC 6979, signature and packet/reference tests |
| Runners | `scripts/` | Icarus, Python/RTL comparison, validation, Yosys and board scripts |
| Captured output | `reports/` | Timestamped prior-run simulation and preliminary Yosys evidence |
| Quartus inputs | `quartus/` | Cyclone V project inputs; no completed build result |
| Documentation | `README.md`, `docs/` | PURI-Sign overview, interface, architecture, verification and integration notes |

No Quartus fit/timing report, `.sof` file, physical board capture, or signing/key-manager/core waveform was found in the reviewed evidence package.

