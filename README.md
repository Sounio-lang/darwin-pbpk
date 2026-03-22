# Darwin PBPK — Epistemic Pharmacokinetic Modeling

**Physiologically-Based Pharmacokinetic modeling with GUM-compliant uncertainty propagation.**

Part of the [Sounio](https://github.com/Sounio-lang/sounio) ecosystem — the L0 systems + scientific programming language for epistemic computing.

## What is this?

A PBPK modeling framework where **every value carries its uncertainty, confidence, and provenance** as intrinsic properties — not as an afterthought.

Three capabilities that no existing PK/PD tool offers:

1. **GUM-through-ODE** — First application of the ISO metrology standard (JCGM 100:2008) to PBPK. Linearized uncertainty propagation through ODE integration via finite-difference Jacobian. 100-1000× faster than Monte Carlo with equivalent accuracy for CV < 40-50%.

2. **Compile-time confidence gates** — The type system enforces minimum epistemic confidence thresholds. If a computation chain erodes confidence below the threshold, the program refuses to compile.

3. **Biomaterial → PBPK → uncertainty pipeline** — Controlled-release kinetics (Higuchi, zero-order, first-order) integrated with full 14-compartment PBPK via operator splitting, with uncertainty tracked at every step.

## Case Study: Sirolimus-Eluting Stent

The framework is validated with **rapamycin (sirolimus)** — the canonical biomaterial+drug combination (Cypher coronary stent, Cordis 2003):

| Metric | Value | Interpretation |
|---|---|---|
| AUC(0-168h) | 3.26 ± 0.88 mg·h/L | CV=27% — CYP3A4 variability propagated |
| Brain/blood ratio | 0.137 | < 0.15 — P-gp efflux at BBB confirmed |
| s_CL (sensitivity) | 0.84 (84%) | CL dominates AUC uncertainty |
| DES/IV AUC ratio | 1:103 | Implant release correctly orders of magnitude below IV |
| Mass conservation | 5.0 → 3.07 mg | 39% eliminated in 7 days |

## Structure

```
stdlib/
├── darwin_pbpk/
│   ├── tsit5_pbpk14.sio          # 14-compartment PBPK, adaptive Tsit5 solver
│   ├── epistemic_pbpk14.sio      # GUM uncertainty bridge (finite-difference Jacobian)
│   ├── epistemic_sim.sio         # 4-compartment epistemic PBPK (analytical GUM)
│   ├── release/
│   │   └── biomaterial_release.sio  # Higuchi, zero-order, first-order release models
│   ├── drugs/
│   │   └── rapamycin.sio         # Sirolimus parameters (Ferron 1997, Lampen 1998)
│   ├── core/
│   │   ├── rodgers_rowland.sio   # Tissue partition coefficient prediction
│   │   └── fractal_blood.sio    # CTRW anomalous transport
│   └── validation/
│       └── metrics.sio           # FDA/EMA metrics (GMFE, AAFE, percent-within-2-fold)
├── epistemic/
│   ├── knowledge.sio             # Knowledge<T> type
│   ├── gum.sio                   # GUM Type A/B, Welch-Satterthwaite
│   ├── propagate.sio             # Variance propagation rules (GUM §13.3)
│   └── ...                       # ODE, PDE, MCMC, Sobol, optimization
tests/
└── run-pass/
    └── rapamycin_epistemic_pbpk.sio  # 3-compartment rapamycin validation (6/6 PASS)
```

## Running

Requires the Sounio JIT compiler:

```bash
SOUC=./artifacts/omega/souc-bin/souc-linux-x86_64-jit

# Epistemic 14-compartment PBPK (8 tests)
$SOUC run stdlib/darwin_pbpk/epistemic_pbpk14.sio

# Biomaterial release models (8 tests)
$SOUC run stdlib/darwin_pbpk/release/biomaterial_release.sio

# Rapamycin 3-compartment epistemic test (6 tests)
$SOUC run tests/run-pass/rapamycin_epistemic_pbpk.sio
```

## References

- JCGM 100:2008 — Guide to the Expression of Uncertainty in Measurement (GUM)
- Ferron GM et al. (1997) *Clin Pharmacol Ther* 61:696-708 — Sirolimus population PK
- Lampen A et al. (1998) *Biochem Pharmacol* — P-gp substrate, BBB transport
- Higuchi T (1963) *J Pharm Sci* 52:1145 — Matrix diffusion release model
- Sousa JE et al. (2003) *Circulation* 107:2274 — Cypher stent clinical data
- FDA (2018) — Physiologically Based Pharmacokinetic Analyses Guidance
- Tsitouras Ch (2011) *Comput Math Appl* 62:770 — RK 5(4) pairs

## License

MIT

## Author

Demetrios Chiuratto Agourakis — ORCID [0009-0001-8671-8878](https://orcid.org/0009-0001-8671-8878)
