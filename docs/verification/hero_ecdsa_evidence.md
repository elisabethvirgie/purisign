# Hero Evidence: Deterministic ECDSA P-256 Signature

**Vector:** RFC 6979 Appendix A.2.5, NIST P-256, SHA-256  
**Message:** `sample`  
**SHA-256(message):** `AF2BDBE1AA9B6EC1E2ADE1D694F41FC71A831D0268E9891562113D8A62ADD1BF`  
**Private key:** REDACTED

| Check | Result |
|---|---|
| Python/reference expected `r` | `EFD48B2AACB6A8FD1140DD9CD45E81D69D2C877B56AAF991C34D0EA84EAF3716` |
| Python/reference expected `s` | `F7CB1C942D657C41D436C7A1B6E29F65F3E900DBB9AFF4064DC4AB2F843ACDA8` |
| RTL `r` compared with expected | PASS, exact match asserted |
| RTL `s` compared with expected | PASS, exact match asserted |
| Independent Python signature verification | PASS |
| Captured RTL simulation result | PASS, 7,437,838 simulation cycles |

The testbench only prints a PASS line on successful signing; it asserts exact equality with the displayed public expected `r` and `s`. The Python reference suite independently verifies the vector. Cycle count is from simulation and does not predict FPGA signing latency.

**Evidence:** [`tb_ecdsa_signer.log`](icarus/tb_ecdsa_signer.log), [`python_reference_current.log`](icarus/python_reference_current.log), `tb/tb_ecdsa_signer.v`, `tests/test_p256.py`.
