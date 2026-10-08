# PURI-Sign verification evidence

This directory collects report-ready summaries and verification evidence for the PURI-Sign project. The maintained implementation is Verilog-2001 `.v` RTL in the parent project directory. Legacy RTL module and testbench identifiers are retained in source files and recorded logs for compatibility and traceability.

## Evidence status

- Python reference suite: 9 tests passed in the recorded current-session run.
- Icarus regression and Python/RTL arithmetic comparison: timestamped recorded runs, not fresh runs; 14 jobs and 254 exact arithmetic matches are reported.
- Yosys structural checks: selected recorded checks passed. The experimental Cyclone V wrapper mapping exited `-9`.
- Quartus compile, timing, `.sof`, and physical DE10-Nano test: not run / not available in the evidence package.

See [`verification_report.md`](verification_report.md) for the provenance, coverage, known gaps, and detailed status. Do not describe simulator cycles as FPGA latency or Yosys structural checks as Quartus resource/timing results.
