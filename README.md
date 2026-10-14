# ProVerif Models for E-LMFA-WSN

Formal verification models (ProVerif / applied-π calculus) accompanying the
manuscript **"E-LMFA-WSN: …"** (resubmission to *Computers & Security*).

These models reproduce the two-layer analysis in the paper:

1. A **Dolev–Yao symbolic analysis** of the original LMFA-WSN protocol and of
   our repaired protocol E-LMFA-WSN (authentication, secrecy, final-message
   integrity, and User–Sensor session-key agreement — paper queries Q1–Q9).
2. A **sensor-node-capture analysis** demonstrating the forward-secrecy /
   node-capture weakness of the original protocol (paper query Q10) and showing
   that the repair resists it.

---

## Files

| File | Protocol | Sessions | Node capture? | Purpose |
|------|----------|----------|---------------|---------|
| `LMFA_original_DY_aligned.pv` | Original LMFA-WSN | replicated (`!`) | No | Baseline DY model; the authentication / integrity queries **fail** here. |
| `E-LMFA_DY_aligned.pv` | E-LMFA-WSN (proposed) | replicated (`!`) | No | Repaired DY model; the **same** query set is **proved**. |
| `LMFA_forward_secrecy.pv` | Original LMFA-WSN | single | Yes (`phase 1`) | Shows Q10: past session secrets leak after a sensor node is captured. |
| `E-LMFA_forward.pv` | E-LMFA-WSN (proposed) | single | Yes (`phase 1`) | Shows the repair resists Q10 (forward secrecy under node capture). |

The two `*_DY_aligned.pv` files share an **identical query set** so that the
"before/after" comparison is apples-to-apples. The two `*_forward.pv` files add
the `phase 1; out(pub, XSGj)` block that models an adversary capturing a sensor
node (and learning its long-term secret `XSGj`) after the session completes.

---

## Modeling notes

- **Channel / attacker:** all messages travel on the public `pub` channel under
  the standard Dolev–Yao attacker.
- **ECC:** `ecmult` is an *idealized* elliptic-curve scalar multiplication. The
  commutativity equation `ecmult(k, ecmult(l, P)) = ecmult(l, ecmult(k, P))`
  captures ECDH without modelling the discrete-log problem.
- **Hash / XOR:** `H` is a symbolic perfect hash; `xor` has left/right
  cancellation. `s2b` converts a scalar to a bitstring.
- **Fuzzy extractor:** abstracted symbolically — `FE` (secret extraction of
  `ω`) and `FEpub` (public helper `θ`); no concrete error-correction modelled.
- **Test secrets:** `SecretU` / `SecretS` are wrapped with `senc`/`sdec` purely
  so that the `attacker(...)` secrecy queries can test whether the *session key*
  is exposed (the key is used to encrypt these dummy secrets).
- **Long-term private values:** `IDUi, PWUi, BIOUi, IDSj, a` (GWN master
  secret), `Ppr` (GWN private key). The registration phase is treated as secure
  (these are initialised private and never disclosed except where the
  node-capture model explicitly releases `XSGj`).
- **Reachability instrumentation:** the `Reach(...)` events and their
  `query event(Reach(...))` lines are debug-only and **disabled by default**
  (commented out). They can be enabled to confirm protocol executability.

### What the repair changes (E-LMFA-WSN)

- The GWN→User final authentication tag `M*13` is bound to the **full**
  transcript via the domain-separated tag `tag3`
  (`H((tag3, M2, M3, M*12, RS, T2, T3, T4))`), closing the missing-binding
  gaps (Q6–Q9).
- The session key uses the **ephemeral** ECDH shared secret `ZUS`
  (`ecmult(N1, RS)` / `ecmult(N3, M1)`) instead of the long-term-derived value
  `M9`, giving forward secrecy under sensor-node capture (Q10).

---

## How to run

Install [ProVerif](https://proverif.inria.fr/) (see version note below), then
run each model from this directory:

```bash
proverif LMFA_original_DY_aligned.pv
proverif E-LMFA_DY_aligned.pv
proverif LMFA_forward_secrecy.pv
proverif E-LMFA_forward.pv
```

ProVerif prints, for each `query`:
- `true` / `false` for `attacker(...)` secrecy queries, or
- `RESULT  … is true` / `is false` for correspondence queries, with a trace
  when a property is violated.

### Expected outcomes (matches the manuscript)

- `LMFA_original_DY_aligned.pv` → the authentication / final-message-integrity
  correspondence queries are **false** (attacks found); this is the basis for
  Q6–Q9.
- `E-LMFA_DY_aligned.pv` → all secrecy and correspondence queries (including
  `UserKey ==> SensorKey`) are **true**.
- `LMFA_forward_secrecy.pv` → the session-secret (`SecretU` / `SecretS`)
  secrecy queries are **false** after `phase 1` node capture (Q10).
- `E-LMFA_forward.pv` → the same secrecy queries remain **true** under node
  capture.

---

## ProVerif version

> **Action required:** the exact ProVerif version used for the reported results
> should be stated here. The models were developed and validated with
> **ProVerif 2.04pl1** (compatible with recent 2.x releases). Please update this
> line with the precise build you ran before archiving / submitting.

```text
proverif -version   # paste the output here
```

---

## Citation

If you use these models, please cite the accompanying manuscript
(E-LMFA-WSN, *Computers & Security* resubmission) and the original protocol:

- Original protocol: LMFA-WSN (ref. [15] in the manuscript).
- Repair & analysis: E-LMFA-WSN (this work).

---

## License

Code is provided for research reproducibility. See the manuscript for the
protocol definitions and security claims.
