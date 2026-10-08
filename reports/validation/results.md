# RTL validation results

Suite: full. Source: simulation. Board: NOT RUN.

| Test | Result | Wall time (s) |
|---|---|---:|
| python_reference | PASS | 0.391 |
| tb_montgomery_mul | PASS | 22.919 |
| tb_mod_arith | PASS | 9.185 |
| tb_point_ops | PASS | 0.803 |
| tb_scalar_mult | PASS | 347.233 |
| tb_hmac | PASS | 0.495 |
| tb_rfc6979 | PASS | 0.459 |
| tb_sha256_hash | PASS | 0.301 |
| tb_ecdsa_signer | PASS | 107.033 |
| tb_key_manager | PASS | 52.75 |
| tb_tinysign_core | PASS | 267.944 |
| tb_tinysign_de10nano | PASS | 0.351 |
| tb_selftest_uart | PASS | 0.209 |
| tb_board_selftest | PASS | 290.968 |

Wall time is host simulation runtime, not FPGA latency. See per-test logs for cycle counts.

## Supplemental Python-versus-Verilog arithmetic comparison

**PASS:** 254/254 exact matches for `mod_add`, `mod_sub`, and `mod_inv` across P-256 field modulus `p` and group order `n`. This supplemental run includes boundary/equal operands, deterministic random vectors, and rejection of inverse-of-zero. Per-vector Python expected values, RTL outputs, and match flags are in [`arithmetic_comparison/results.json`](../arithmetic_comparison/results.json); the Icarus transcript is [`arithmetic_comparison/icarus.log`](../arithmetic_comparison/icarus.log), with vector inputs in [`arithmetic_comparison/vectors.txt`](../arithmetic_comparison/vectors.txt).
