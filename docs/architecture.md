# PURI-Sign architecture

## Project scope and source alignment

PURI-Sign is the public project name for this existing protected-key ECDSA P-256 signing prototype, positioned under Secure Identity & Security Element. It accepts a 32-byte SHA-256 digest and returns an ECDSA `(r,s)` signature. The private scalar is provisioned through a write-only register path and retained in volatile internal state; normal register readback does not expose it. The public key may be read.

The maintained RTL identifiers `tinysign_core` and `tinysign_de10nano` are retained as legacy technical names so existing testbench, synthesis, and project references remain intact. The rebrand changes documentation only; implementation and behavior are unchanged.

The supplied competition proposal and the later engineering implementation are not fully aligned. The proposal describes verification-only ECDSA, while this repository implements protected-key signing. Treat the RTL and test evidence as the source for implemented functionality; proposal text and estimates are not proof of behavior or measured PPA.

## Module hierarchy

```text
tinysign_de10nano (Avalon-MM shell; board integration pending)
└── tinysign_core (register decode, command FSM, status and data path)
    ├── key_manager (write-only key staging, validation, dG, lock and zeroize)
    │   └── scalar_mult -> point_ops -> modular arithmetic modulo p
    └── ecdsa_signer
        ├── rfc6979 -> hmac_sha256 -> sha256_hash -> sha256_compress
        ├── scalar_mult -> point_ops -> modular arithmetic modulo p
        └── mod_inv/mod_mul/mod_add modulo n for s
```

The pure core register interface and command sequencing are implemented. The Avalon-MM shell and Quartus project skeleton exist; synthesis and board integration remain incomplete.

All maintained RTL uses Verilog-2001. Modular multiplication uses `mod_mul -> montgomery_mul`, with conversion inside `mod_mul` so the rest of the design retains ordinary residues. Both field `p` and subgroup `n` operations use this path. The serialized design uses Jacobian point arithmetic and a fixed 256-bit scalar schedule.

## Security lifecycle

1. Host writes eight 32-bit words to the write-only `KEY_D` staging window.
2. `PROVISION_KEY` requires every word, checks `1 <= d < n`, and computes/stores `Q=dG` internally.
3. `LOCK_KEY` prevents further provisioning writes. Signing is accepted only for a valid, locked key.
4. Host writes a digest and issues `SIGN_DIGEST`; RFC 6979 derives a deterministic nonce internally, and only `(r,s)` is exposed as signature output.
5. `ZEROIZE` clears the private key, derived public key, signature, and crypto intermediates in the RTL path; reset also clears volatile key state.

This is register-level readback protection. It does not prevent privileged host software from observing the provisioning bus, power/EM analysis, or invasive silicon access. Any command source attached to the bus can request signing once the key is locked; host identity/authorization is not implemented.

## P-256 parameters

NIST SP 800-186 is the reference for the P-256 field prime `p`, subgroup order `n`, curve coefficient `a=-3`, and generator `G`. See [`interface.md`](interface.md) for host-visible registers and commands and [`verification/verification_report.md`](verification/verification_report.md) for evidence limits.
