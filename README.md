# Server-Mandatory 2-of-3 Peer-Assisted Passkey Recovery: Tamarin Formal Analysis

Symbolic formal verification model and proof artefacts for a server-mandatory 2-of-3 threshold peer-assisted FIDO2 passkey recovery protocol under the eventually compromised server (ECS) model in the Tamarin prover.

## Repository Contents

- `3_ECS_Threshold_2of3_v6.spthy` — primary 16-lemma Tamarin theory (SHA-256 `89da524f7c0fad9b94a019a51aa4faecf68a3e51f173b2a16bf0b4879bd5c783`).
- `DependencyGraph_4_v6.spthy`, `log.txt`, `log_summary.txt` — machine-checked proof script, full prover output and timing summary for the primary run.
- `3_ECS_Threshold_2of3_v7.spthy` — extended 18-lemma theory adding the two supplementary T2 server-necessity lemmas (`all_peers_without_server_insufficient` and `all_peers_premise_reachable`).
- `DependencyGraph_T2_supplement.spthy`, `log_T2_supplement.txt` — proof script and run log for the two supplementary T2 lemmas.

## Execution Environment

- **Primary run (`v6`):** Tamarin 1.12.0 with Maude 3.5.1 on an Ubuntu 24.04.4 LTS (Linux 7.0.0-28-generic) VMware full-virtualisation guest with 32 virtual cores (Intel Xeon Gold 6530, one socket, one thread per core, 160 MiB shared last-level cache) and 456 GiB RAM (`MemoryMax=400G`, `MemorySwapMax=0`).
- **Supplementary T2 run (`v7`):** Tamarin 1.12.0 with Maude 3.4.
