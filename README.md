# Server-Mandatory 2-of-3 Peer-Assisted Passkey Recovery — Tamarin Formal Analysis

Research artifact for the paper:

> **Formally Verified Server-Mandatory 2-of-3 Peer-Assisted Passkey Recovery**
> Murat Sekmen and Kemal Bicakci, Informatics Institute, Istanbul Technical University.

This repository contains the complete [Tamarin prover](https://tamarin-prover.github.io/)
model, the proof logs, and the generated dependency graphs needed to reproduce
every claim in the paper. All 16 properties are proved for **unboundedly many
sessions and agents**.

It extends the baseline
([CODASPY '26](https://doi.org/10.1145/3800506.3803497),
[artifact](https://github.com/congenial-max/FIDO2PasskeyPeerAssistedRecoveryFormalAnalysis))
from one designated peer to any two of three nominated colleagues, removing the
single-peer availability bottleneck while preserving the baseline's
double-key-pair role isolation.

---

## What the protocol does

A FIDO2 passkey cannot be reset like a password, because the server holds no
secret. This protocol lets an employee recover a lost credential with the help
of the server **and** two colleagues, where neither side can act alone.

**Setup** (once, while the user still has a working device). The user `A`
nominates three colleagues, splits the recovery key `k_A` into one server
component `σ_S` and three peer shares `σ_1, σ_2, σ_3`, seals each share for its
nominee, and deposits the passkey at the server as a blob `β` encrypted under
`PK_A`.

**Recovery** (on a replacement device holding no secret). The device asks the
server, which returns `β`, `σ_S` and the sealed shares in the clear. The user
then meets **any two** of the three nominees in person, receives their shares by
QR code, and rebuilds `k = rec(σ_S, σ_x, σ_y)`.

**Separation of powers.** The server alone cannot recover a credential: `σ_S`
reduces to nothing without two peer shares, and this holds even when the
adversary has read the server's entire database. The nominees alone cannot
recover either, and this is structural rather than a matter of counting them —
no equation of the theory fires without the server component, so even all three
colleagues together get nowhere.

---

## Threat model

The adversary is Dolev–Yao, composed with an **eventually compromised server**
(ECS) and up to `t-1 = 1` corrupted nominees.

The ECS grant is **ungated**: `Reveal_Server_Storage` copies the stored server
component of *any* user onto the public network at *any* point in the trace,
including before that user has ever asked to recover. No restriction gates it
and no lemma excludes it. This is deliberate — a real database is read by backup
theft, an injected query or an insider export, none of which wait for a
recovery request.

### Assumptions

| | Assumption |
|---|---|
| **A1** | Client–server channels are mutually authenticated and confidential in both phases; recovery-phase server *responses* are public (ECS). |
| **A2** | Nominees are pairwise distinct and distinct from `A`. |
| **A3** | The out-of-band channel (in-person QR scanning) is confidential, mutually authentic, and un-redirectable. |
| **A4** | Peer corruption is two-dimensional: identity-key reveal (counted against the collusion budget) and unconditional device-storage dumps. |
| **A5** | Each user performs one setup, one nomination, and one dealing. |
| **A6** | Escrowed shares carry a type tag and the owner identity inside the encryption. |
| **A7** | The replacement device obtains `A`'s registered `PK_A` and nominee list authentically and out of band (enterprise directory, MDM enrollment, or similar) — **not** from the recovery server. |

### Goals

| | Goal |
|---|---|
| **G1** | Secrecy of recovery key and credential against the ECS adversary colluding with any `t-1` nominees. |
| **G2** | Server necessity — no reconstruction without the server. |
| **G3** | Liveness under attrition — any one nominee absent or corrupted. |
| **G4** | Role isolation, preserving the double-key-pair defence against the domino effect. |

---


## Reproducing the results

**Requirements.** Tamarin prover 1.12.0, Maude 3.5.1 and a
large-memory host (the measured peak is 59.1 GiB).

Flag notes:

- `--auto-sources` generates the `AUTO_typing` invariant that closes the
  residual partial deconstructions on the server's opaquely forwarded setup
  package. Without it the non-source lemmas are not valid.
- `-c=50 -s=50` raise the open-chain and saturation bounds above Tamarin's
  defaults (10 and 5); the invariant does not close under the defaults on this
  theory.
- `--derivcheck-timeout=0` disables the message-derivation check *during the
  proof run only*, for speed. Run it separately with a positive bound to confirm
  no honest rule inverts a constructor:
- To inspect the source precomputation without proving anything:
  ```bash
  tamarin-prover 3_ECS_Threshold_2of3_v6.spthy --precompute-only --auto-sources -c=50 -s=50
  ```

The model is **self-contained**: the three in-file tactics
(`reconstruction_secrecy`, `tightness_witness`, `executability`) replace the
external oracle script the baseline artifact needed, so no `--heuristic=O` and
no oracle file are required.

---

## Verification results

Measured on an Intel Xeon Gold 6530 (32 vCPUs), Tamarin 1.12.0 / Maude 3.5.1,
under a 400 GiB cgroup ceiling with swap disabled.

| Wall clock | Solver time | CPU | Peak resident set | Proof steps |
|---|---|---|---|---|
| 45 min 22 s | 2716.34 s | 237 % (≈2.4 cores) | 59.1 GiB | 39,594 |

All 16 lemmas and the generated invariant **verified**, exit status 0.

| Lemma | Form | Goal | Steps |
|---|---|---|---|
| `protocol_is_executable_strong` | ∃ | witness | 13 |
| `secrecy_share` | ∀ | G1 | 425 |
| `secrecy_main_key_collusion` | ∀ | G1 | 5,043 |
| `secrecy_fido_key_collusion` | ∀ | G1 | 16,937 |
| `collusion_premise_reachable` | ∃ | witness | 23 |
| `collusion_bound_tight` | ∃ | G1 tight | 15 |
| `peers_alone_insufficient` | ∀ | G2 | 54 |
| `peers_alone_premise_reachable` | ∃ | witness | 6 |
| `no_reconstruction_without_server` | ∀ | G2 | 93 |
| `recovery_session_binding` | ∀ | consistency | 23 |
| `fido_blob_integrity` | ∀ | integrity | 12,491 |
| `isolation_own_peer_key_compromise` | ∀ | G4 | 4,376 |
| `isolation_guardian_user_key_compromise` | ∀ | G4 | 53 |
| `distinct_helpers_enforced` | ∀ | consistency | 5 |
| `recovery_survives_one_absent_peer` | ∃ | G3 | 25 |
| `consistency_fido_recovery` | ∀ | consistency | 8 |
| `AUTO_typing` (generated) | ∀ | sources | 4 |

The suite is 8 security properties, 5 `exists-trace` witnesses, and 3
consistency checks. The witnesses certify non-vacuity rather than asserting
security, so a bare count of sixteen would overstate the adversarial content.

Cost concentrates where the ECS view meets the collusion guard: credential
secrecy and blob integrity are the only proofs above 10⁴ steps, and together
with recovery-key secrecy and initiator-side isolation they account for 98 % of
all steps, because every goal mentioning `σ_S` must consider both the served
response and the database dump as its source. The witnesses that exercise the
algebra itself remain trivial by comparison — the tightness witness closes in 15
steps. The threshold structure is carried by the equational theory, not by the
proof search.

---

## The equational abstraction

Shamir's scheme is **not** modeled directly, and the obstacle is its arithmetic
rather than the scheme itself. A Shamir share is a point on a random polynomial
over a finite field and recovery is Lagrange interpolation, so shares can be
added, scaled, and mixed. Tamarin accepts only equational theories that are
subterm convergent and have the finite variant property; field arithmetic is not
one of them.

What is modeled instead is the **access structure** the scheme is used for:

```
functions: srv/2, shr1/2, shr2/2, shr3/2, rec/3

equations:
  rec(srv(k, r), shr1(k, r), shr2(k, r)) = k,
  rec(srv(k, r), shr1(k, r), shr3(k, r)) = k,
  rec(srv(k, r), shr2(k, r), shr3(k, r)) = k
```

- **Opacity.** No destructor exists for `srv` or `shr_i`, so a share in
  isolation reveals nothing of `k`. Per-share secrecy becomes expressible — and
  is proved — where the common `Share(k,p)` fact abstraction cannot even state
  it.
- **Server-mandatoriness.** Every equation requires the `srv(k,r)` component. No
  combination of peer shares reduces.
- **Threshold.** Every equation requires two *distinct* share indices. One share
  plus the server component is irreducible.
- **Dealing coherence.** The shared randomness `r` ties one dealing's components
  together; terms from different dealings never mix.

The price is that linearity disappears from the model, so attacks exploiting it
are invisible here. The guarantees are those of an **idealized, opaque,
non-malleable authenticated** sharing scheme; a deployment must supply
verifiable secret sharing or per-share authentication before the symbolic
results transfer.

## Notation

| Paper | Model |
|---|---|
| `k_A` (recovery key) | `ltk_user` |
| `ik`, `IK` (identity pair) | `ltk_peer`, `pk(ltk_peer)` |
| `σ_S`, `σ_i` | `srv_share`, `shr_i(ltk,r)` |
| `β` (credential blob) | `enc_fido` |
| `f_A`, `ν` | `~fido_key`, `~nonce` |
| `Rec(A,k)` | `A_Reconstructs_Key` |
| `Nom(A,B⃗)` | `Nominated_All` |
| `RevU`, `RevP` | `LtkRevealUser`, `LtkRevealPeer` |
| `Srv(A,σ_S,β)` (served) | `ServerSendsRecoveryData` |
| `Dump(A)` (server dump) | `ServerStorageRevealed` |
| `Deal(A,k)`, `Help(A,H_1,H_2)` | `SharesDealt`, `HelpersSelected` |

---

## Citing

Baseline paper:

```bibtex
@inproceedings{sekmen2026peer,
  author    = {Sekmen, Murat and Bicakci, Kemal},
  title     = {Formal Verification of Peer-Assisted {FIDO2} Passkey Recovery Protocol with {Tamarin}},
  booktitle = {Proc. ACM CODASPY},
  pages     = {349--360},
  year      = {2026},
  doi       = {10.1145/3800506.3803497}
}
```

## Acknowledgment

This work was supported by the Scientific Research Projects Coordination Unit of
Istanbul Technical University (Grant No. 2025-46134).
