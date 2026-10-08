# PURI-Sign

PURI-Sign is a Verilog-2001 prototype of a protected-key ECDSA signing element for 32-byte SHA-256 digests. It implements NIST P-256 signing with an internally provisioned private key, deterministic RFC 6979 nonce generation, and a 32-bit register interface. The project is a prototype relevant to PERURI Chip Hackathon 2026's **Secure Identity & Security Element** topic; it is not a PERURI deployment or a production secure element.

## Overview

Digital signatures help a verifier check that data was signed by the holder of a private key and was not altered after signing. In identity and document workflows, that property supports authenticity and verification. PURI-Sign explores one hardware boundary for this function: a host stages a private scalar through a write-only register window, provisions and locks it, then submits a SHA-256 digest and reads the resulting signature. This does not establish who the host is or authorize a signing request.

## Why hardware?

The signing datapath and volatile key state are implemented in RTL so the key lifecycle, arithmetic, command sequence, and register behavior can be exercised as a hardware IP prototype. The implementation provides tested register-level key readback suppression and zeroization behavior. It has not been evaluated for physical tampering, power or electromagnetic leakage, fault injection, or production deployment.

## PURI-Sign architecture

The maintained source is the Verilog-2001 implementation in `rtl/`. The existing hierarchy is:

```text
tinysign_de10nano  (Avalon-MM shell; board integration pending)
└── tinysign_core  (register decode, command FSM, status and data path)
    ├── key_manager (key staging, validation, public-key derivation, lock, zeroize)
    │   └── scalar_mult → point_ops → modular arithmetic modulo p
    └── ecdsa_signer
        ├── rfc6979 → hmac_sha256 → sha256_hash → sha256_compress
        ├── scalar_mult → point_ops → modular arithmetic modulo p
        └── modular arithmetic modulo n for signature s
```

These `tinysign_*` RTL identifiers are retained for compatibility with the existing testbenches and project scripts. They are internal technical names; the project and documentation name is PURI-Sign. The hierarchy and behavior have not been changed for this rebrand. See [the architecture and security lifecycle](docs/PURI-Sign.md) and [the register map](docs/interface.md).

## How it works

1. The host writes all eight 32-bit words of private scalar `d` to `KEY_D`; reads from that window return zero.
2. `PROVISION_KEY` validates `1 ≤ d < n` and derives the public point `Q = dG`.
3. `LOCK_KEY` prevents further key writes. Signing requires a valid, locked key.
4. The host stages a 256-bit SHA-256 digest and issues `SIGN_DIGEST`.
5. RFC 6979 derives the nonce using HMAC-SHA-256. The signer returns ECDSA components `(r,s)` through read-only result registers.
6. `ZEROIZE` clears key and result state and can abort an active signing operation.

The host supplies a digest; arbitrary-message hashing is not the core's host input mode. SHA-256 is present internally as part of HMAC/RFC 6979.

## RTL implementation

The datapath uses serialized, resource-shared operations. Point arithmetic uses Jacobian coordinates; scalar multiplication follows a fixed 256-bit schedule. Modular arithmetic supports the P-256 field modulus `p` and subgroup order `n`. `mod_mul` wraps the Montgomery multiplier so its callers use ordinary residues. These are implementation descriptions, not claims about area, timing closure, or side-channel resistance.

## Security considerations

The RTL and directed simulation cover private-key staging, range checks, lock behavior, normal key-register readback returning zero, and zeroization in tested paths. The private key still enters over the host provisioning bus, which is not protected from observation by this design. Any bus master can request signing once the key is locked; host identity and authorization are not implemented. Volatile state is reset/zeroized, but no physical tamper response, side-channel countermeasure, fault defense, secure boot, or nonvolatile key storage is claimed. See [interface and security details](docs/interface.md) and [verification limits](docs/verification/verification_report.md).

## Verification

Available sources include RTL testbenches under `tb/`, a Python P-256 reference and tests under `python/` and `tests/`, and runners under `scripts/`. Captured evidence reports selected passing tests for arithmetic, ECC, SHA-256, HMAC, RFC 6979, ECDSA, key management, core integration, wrapper, and self-test simulation. The RFC 6979 P-256 signing vector matched the expected `(r,s)` and was independently verified by the Python reference. A direct arithmetic comparison records 254 exact matches.

Evidence provenance matters: these Icarus/Yosys results are recorded prior runs, not fresh runs in this session. The Python suite was previously rerun and passed 9 cases. Quartus compilation, FPGA fit/timing, `.sof` generation, and physical-board testing have not been completed. See the [verification report](docs/verification/verification_report.md) for scope and limitations.

## FPGA prototype status

The project files target a DE10-Nano / Cyclone V (`5CSEBA6U23I7`) and include an Avalon-MM shell and a standalone self-test project. This is a target configuration, not evidence of a working board build. Quartus implementation and physical hardware validation remain unverified.

## Resource utilization

**TBD — not measured in a completed Quartus implementation.** Recorded Yosys structural checks are not FPGA resource or timing results. A recorded experimental Cyclone V wrapper mapping exited with `-9`.

## Repository structure

| Path | Contents |
|---|---|
| `rtl/arithmetic/`, `rtl/ec/`, `rtl/ecdsa/` | Modular arithmetic, point operations, scalar multiplication, signer |
| `rtl/sha256/`, `rtl/nonce/`, `rtl/security/` | Hash/nonce logic, key manager, register core |
| `rtl/platform/`, `rtl/test_support/` | Avalon-MM shell and self-test/UART support |
| `tb/`, `tests/`, `python/` | Verilog testbenches and Python reference tests |
| `scripts/` | Simulation, validation, arithmetic comparison, Yosys, and board scripts |
| `quartus/` | Cyclone V project inputs; no completed build output |
| `reports/` | Recorded simulation and preliminary Yosys evidence |
| `docs/` | Architecture, interface, verification, and integration notes |

## Quick start

From this directory, the existing Python reference tests can be run with:

```sh
python3 -m unittest discover -s tests -v
```

With Bash, Icarus Verilog (`iverilog`, `vvp`), and Python installed, run the existing full RTL testbench set with:

```sh
bash scripts/run_rtl_tests.sh
```

The validation wrapper records Python, arithmetic-comparison, and RTL test results:

```sh
python3 scripts/run_validation.py --quick
python3 scripts/run_validation.py
```

The full run includes multi-million-cycle simulations. Tool availability and platform prerequisites apply; recorded outputs are not a substitute for running the command in the target environment. Yosys and Quartus commands and their limits are documented in [proposal testing](docs/proposal_testing.md).

## Demo

The repository contains simulation testbenches and a Montgomery waveform demo. A board self-test RTL project and capture script are present, but no physical-board demo or board capture is included in the current evidence. Do not present the self-test simulation as a board test.

## Current status

- **Implemented:** P-256 arithmetic and point/scalar operations, SHA-256/HMAC/RFC 6979 nonce path, ECDSA signing controller, key manager, register core, Avalon-MM shell, and self-test support RTL.
- **Verified in recorded simulations:** selected vectors and register/control paths as summarized above; recorded evidence is not a fresh full regression.
- **Freshly rerun in the evidence record:** Python reference suite, 9 tests passed.
- **Not verified:** Quartus compilation/fitting/timing, programming file generation, and physical DE10-Nano operation.
- **Not measured:** FPGA resource use, timing closure, power, and hardware signing latency.

## Future work

Complete the Quartus and board integration flow, capture reproducible hardware results, and evaluate the security boundary and threat model. Any security or performance claim should follow measured evidence. These are future activities, not implemented features.

## PERURI Chip Hackathon 2026

The closest supported area in the guide is **Secure Identity & Security Element**: the prototype explores protected-key signing and authenticity verification at an RTL/IP boundary. It does not implement the guide's separate hardware cryptography accelerator, AI/edge accelerator, or secure communication system as an end-to-end product. The relevance to PERURI's identity and authenticity context is conceptual. No partnership, official deployment, certification, or production readiness is claimed.

## References

- NIST SP 800-186, *Recommendations for Discrete Logarithm-Based Cryptography: Elliptic Curve Domain Parameters* (P-256 parameters).
- NIST FIPS 180-4, *Secure Hash Standard* (SHA-256).
- NIST FIPS 186-5, *Digital Signature Standard* (ECDSA).
- RFC 6979, *Deterministic Usage of the Digital Signature Algorithm (DSA) and Elliptic Curve Digital Signature Algorithm (ECDSA)*.
- PERURI Chip Hackathon 2026 guide and supplied proposal materials (positioning context; proposal statements are not implementation evidence).
