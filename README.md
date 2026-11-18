# Cloud-Audit-TPA (release v2.0)

Code, raw results and figures for *"A Trust-Aware Third-Party Auditing Framework with Independent Verification for Cloud Data Integrity"*.

## What is implemented
`audit_scheme.py` implements the RSA-group homomorphic-tag PDP of Section 3.2 **with public verification**:
tags `t_i = (H(F||i)·g^m_i)^d`, proof `(mu, T)`, verification `T^e == prod H(F||i)^v_i · g^mu (mod N)` using only `(N, e, g)`.
It also provides plain-product and small-exponent batch verification, a corruption model, and the exact/with-replacement detection probabilities.
All modular arithmetic is Python's built-in `pow` (no GMP, not constant-time). All randomness is from explicitly seeded `random.Random` instances. **Do not use this code to generate real keys.**

## Layout
| File | Purpose |
|---|---|
| `audit_scheme.py`, `stats_util.py`, `env_info.py` | protocol, statistics helpers, environment recorder |
| `exp1_taggen.py` | Exp. 1 tag-generation cost (independent replicates) |
| `exp2a_audit_round.py` | Exp. 2a audit-round cost vs sample size |
| `exp2b_detection.py` | Exp. 2b detection probability, real crypto, independent seed per trial (resumable) |
| `exp3_batch.py` | Exp. 3 sequential vs batch verification + exponent micro-benchmark |
| `exp4_trust.py` | Exp. 4 trust score, real crypto, 30 runs (seeds 1000-1029) |
| `sim_fast.py`, `exp4b_sensitivity.py` | fast-path simulator and sensitivity / long-horizon sweeps |
| `exp5_baseline.py` | baseline comparison (no accountability, IV every round, committee abstraction of [11]) |
| `analyze.py`, `make_charts.py` | raw results -> `results/summary.json` -> `figures/*.png` |
| `tests/selftest.py` | correctness, detection and batch-soundness (cancelling-errors attack) checks |
| `run_all.py` | whole pipeline |
| `results/` | raw per-replicate / per-trial JSON, `summary.json`, `env_info.json`, logs |

## Reproduce
```
pip install -r requirements.txt          # numpy, matplotlib
python tests/selftest.py
python run_all.py                        # full run, roughly 1.5 h on a 2.6-2.8 GHz single core
```
Timing experiments (1, 2a, 3) must run alone on the machine. Wall-clock timings depend on hardware; `results/env_info.json` records the machine used for the shipped results. Detection and trust-layer results are deterministic given the seeds.
Seeds: Exp1 100+r, Exp2a 200+r, Exp2b `"300|frac|c|trial"`, Exp3 400+r, Exp4 1000-1029, Exp4b numpy seed 7, Exp5 numpy seed 9.

## Known limitations of the prototype
Blocks are small integers; challenge coefficients are 16-bit; no CRT; the small-exponent batch test lacks the safe-prime / QR safeguards of a hardened protocol; the committee baseline abstracts only the audit-redundancy of [6]; no BLS instantiation; no network measurements.
