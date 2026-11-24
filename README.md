# SAPS: Semantics-Aware Path Steering for Multi-Tenant NVMe-over-Fabrics

This repository contains the SAPS prototype. SAPS treats path repair and tenant scheduling as one allocation problem.
It derives a graded path-health signal from NVMe completion behavior, converts
that signal into effective path capacity, and allocates the feasible service
budget with weighted max-min fairness.

The prototype runs the estimator and allocator on NVIDIA BlueField-3 DPA cores.
The host-side SPDK path reads committed decisions and applies them while
submitting I/O.

## Design in one equation

For path `p`, SAPS combines provisioned path capacity `K_p` with estimated
health `h_p`:

```text
e_p = K_p h_p
B   = min(C, sum_p e_p)
```

Here, `C` is the configured service envelope and `B` is the service budget
that can be promised under the current path state. A demand-aware weighted
progressive fill divides `B` among active tenants. Each tenant's rate is then
split across paths in proportion to `e_p`. Excluded paths receive only the
bounded probe traffic needed to test recovery.

## Repository layout

- `src/` contains the DPA program, the host library, and shared data structures.
- `integration/spdk/` contains the SPDK patches used by the prototype; see
  `integration/spdk/README.md`.
- `docs/` describes the control loop, implementation boundary, and runtime
  configuration.

The `sapsq` prefix remains in internal symbols and filenames for compatibility
with recorded experiment manifests. It refers to the SAPS controller described
in the paper.

## Build

The DPA application requires NVIDIA DOCA FlexIO and the SPDK integration used by
the testbed.

```bash
cd src
meson setup build
ninja -C build
```

`integration/spdk/0001-saps-spdk-integration.patch` contains the submit,
completion, path-selection, initialization, and target-side fault-injection
changes used in the evaluation. It applies to SPDK commit
`a83e52f1da18807e21b552a0fe35057f8e9ea586`; `integration/spdk/README.md` gives
the build steps. `integration/spdk/0002-saps-least-loaded-selection.patch`
applies on top of it and adds the least-loaded multipath selection.

## Requirements

- NVIDIA BlueField-3 with DPA and FlexIO support
- NVIDIA DOCA SDK 2025.10 or newer
- SPDK built with the SAPS integration
- An NVMe-oF RDMA initiator and target with multiple paths

## License

Apache License 2.0. See `LICENSE`.

## Citation

If you use this artifact, cite:

> Yong-Xuan Huang, Ming-Hung Chen, I-Hsin Chung, and Jerry Chou.
> "SAPS: Semantics-Aware Path Steering for Multi-Tenant NVMe-over-Fabrics." Future Generation Computer Systems, under review, 2026.

A machine-readable record is available in `CITATION.cff`.
