# PURI-Sign engineering concept

## Project overview

PURI-Sign is the rebranded presentation of the existing Verilog-2001 prototype in this repository. It implements ECDSA signing on NIST P-256 for a host-supplied 32-byte SHA-256 digest. A private scalar is staged through a write-only register path, validated, retained in volatile RTL state, and used to derive a public key and produce signatures. Nonces are derived deterministically with RFC 6979 and HMAC-SHA-256.

This is an RTL/IP prototype positioned under PERURI Chip Hackathon 2026's Secure Identity & Security Element topic. It is not an official PERURI system, certified device, or production secure element.

## Problem and motivation

Identity and document systems need ways to check authenticity and detect changes. A digital signature provides a mathematical verification mechanism for a message and public key. PURI-Sign explores how signing and a limited key lifecycle can be represented as hardware IP, with explicit registers and command sequencing. It does not authenticate the host or decide whether a signing request is authorized.

## Why hardware IP

The implementation makes the signing datapath, key state, and host interface explicit in synthesizable RTL and supports simulation at both module and register-interface levels. This gives a concrete basis for later FPGA integration and measurement. The present evidence does not show a fitted FPGA, measured PPA, or physical security evaluation.

## Existing architecture

```text
tinysign_de10nano (Avalon-MM shell; board integration pending)
└── tinysign_core (register decode, command FSM, status/data path)
    ├── key_manager (write-only staging, validation, dG, lock, zeroize)
    │   └── scalar_mult → point_ops → modular arithmetic over p
    └── ecdsa_signer
        ├── rfc6979 → hmac_sha256 → sha256_hash → sha256_compress
        ├── scalar_mult → point_ops → modular arithmetic over p
        └── mod_inv/mod_mul/mod_add over subgroup order n for signature s
```

These are the unchanged RTL module names in the maintained source tree. `tinysign_core` and `tinysign_de10nano` remain legacy technical identifiers referenced by existing testbenches, Quartus inputs, and recorded reports. Rebranding did not change module hierarchy, ports, state machines, parameters, constants, or RTL behavior.

### Block roles

- **Register core:** decodes the 32-bit host register map, sequences provisioning/lock/sign/zeroize commands, and exposes status, public key, and signature words.
- **Key manager:** collects all eight words of the private scalar, validates its range, derives `Q=dG`, tracks valid/locked state, and clears volatile key state on zeroization/reset.
- **ECDSA signer:** consumes the private scalar and digest, obtains a deterministic nonce through the RFC 6979 path, and computes `(r,s)`.
- **Arithmetic and ECC:** modular arithmetic supports the field prime `p` and subgroup order `n`; point operations use Jacobian coordinates and the scalar multiplier uses a fixed 256-bit schedule. Multiplication is serialized through the existing Montgomery-based implementation.
- **Avalon-MM shell:** adapts the core bus to an Avalon-MM style interface. Its simulation is not proof of a complete DE10-Nano/Platform Designer integration.

## Data and control flow

The host writes all `KEY_D[0..7]` words. `PROVISION_KEY` checks completeness and `1 ≤ d < n`, then calculates the public key. `LOCK_KEY` prevents further writes. The host stages a digest in `DIGEST[0..7]`, issues `SIGN_DIGEST`, then polls status and reads `SIG_R` and `SIG_S`. `ZEROIZE` can abort an active signing operation and clears sensitive and result state. `KEY_D` reads return zero in the register read path. The provisioning bus itself is not hidden from the host or other bus observers.

The top-level host input is a digest, not arbitrary message bytes. The internal SHA-256 engine is used by HMAC-SHA-256 for deterministic nonce generation.

## RTL and repository implementation

The maintained implementation is `.v` Verilog-2001 under `rtl/`. `tb/` contains testbenches, `python/` and `tests/` contain a reference model/test suite, `scripts/` contains runners, and `quartus/` contains project inputs for a DE10-Nano Cyclone V target. `docs/architecture.md` and `docs/interface.md` retain lower-level design details.

## Security scope

The prototype demonstrates RTL-level key write staging, range validation, lock behavior, zero-valued normal readback for the private-key register window, and zeroization in covered simulation paths. It does not provide or claim:

- protection of the provisioning bus from a privileged or observing host;
- host authentication or authorization for signing;
- physical tamper resistance or invasive attack protection;
- power/EM side-channel countermeasures or fault-injection defenses;
- persistent secure storage, secure boot, certification, or production readiness.

## Verification and evidence

The repository includes testbenches for modular arithmetic, Montgomery multiplication, point operations, scalar multiplication, HMAC, RFC 6979, SHA-256, ECDSA signing, key management, the core, Avalon wrapper, UART report, and board self-test logic. The captured Icarus regression records 14 passing jobs (Python reference, arithmetic comparison, and RTL simulations); the comparison records 254 exact arithmetic matches. The captured RFC 6979 P-256 vector has exact RTL `(r,s)` matches and passes independent Python signature verification. The Python reference suite has a recorded current-session run of 9 passing tests.

These simulation and Yosys results are timestamped evidence from earlier runs except for the Python suite noted above. They cover selected vectors and do not constitute a formal proof. Recorded Yosys structural checks passed for selected tops, while the experimental Cyclone V wrapper mapping exited `-9`. Quartus build, fitting, timing closure, `.sof` generation, and physical board testing are not complete. Full provenance and known coverage gaps are in `docs/verification/verification_report.md`.

## Prototype and implementation status

- **RTL:** implemented as described in the hierarchy above.
- **Selected simulation checks:** recorded PASS; distinguish these from fresh execution.
- **Python reference tests:** recorded 9 tests passed.
- **FPGA target:** DE10-Nano / Cyclone V project inputs exist; hardware build and board integration are pending.
- **Resource use, timing, power, board latency:** TBD / not measured in a completed FPGA implementation.

## Hackathon fit

The implementation most directly fits **Secure Identity & Security Element**, because its actual function is protected-key handling in tested RTL paths plus ECDSA signing for authenticity verification. This connection is conceptual and does not imply integration with PERURI. The current RTL does not demonstrate a complete secure communication product or an AI/edge accelerator. Security-by-design is a development objective; the limited protections and untested threat areas above must remain explicit.

## Future development

Future work can complete the existing Quartus/board path, record implementation measurements, broaden verification coverage, and review the bus and physical threat model. These are open tasks and do not describe current capabilities.

## References

- NIST SP 800-186, P-256 domain parameters.
- NIST FIPS 180-4, SHA-256.
- NIST FIPS 186-5, ECDSA.
- RFC 6979, deterministic DSA/ECDSA nonce generation.
- PERURI Chip Hackathon 2026 guide and supplied proposal materials, used as context rather than evidence of implementation.
