
# PQCVPN — TRL-6 test bench evidence pack (run of 11-Oct-2026)

# Author: Dr. Mohit Saxena m22aie240@alumni.iitj.ac.in

The filled data sheet is `TRL_6_Data_1_filled.docx`. Every number in blue in that sheet is generated
from `metrics.json`, which `tools/metrics.py` builds from the raw files in this folder.

## Software under test
- pqcvpn, commit 28a52a3 plus the uncommitted working-tree changes present on 11-Oct-2026 (`source-under-test.tar.gz` in the parent folder)
- Built with go 1.24.7 for linux/amd64 using `CGO_ENABLED=0`; the binary hash is in `tools/binaries.sha256`
- Logs state the suite on every handshake: X25519 + ML-KEM-768 (Kyber) hybrid key exchange, ML-DSA-65 (Dilithium) peer identity verified, ChaCha20-Poly1305 transport

## Bench
- Linux network namespaces with real TUN devices: clients `c*`, gateway `gw`, data-centre host `dc` (10.50.0.0/24), attacker / load generator `atk`, access router `rtr` (second bearer)
- Underlay 172.30.0.0/16 (second bearer 172.31.0.0/16); overlay 10.8.0.0/16
- Cloud VM: 2 vCPU Xeon 2.3 GHz, 8 GB, Linux 6.18. All roles share the 2 vCPUs, so treat throughput and CPU figures as conservative

## Folders
| ID | Folder | Contents |
|---|---|---|
| E-01 | 00-build | go vet + 77 unit tests (all pass) |
| E-02..E-05 | 01-functional-dataplane | establishment pcap and logs, RTT, iperf3, CPU/RSS, DC HTTP access, plaintext check |
| E-06 | 02-voice-rekey | RTP voice unprotected / protected / under load, rekey timing. The pcap is filtered to handshakes plus the voice client; the bulk-TCP frames were removed to save space |
| E-07 | 03-mobility-interruption, 03b-gateway-restart | address change, Wi-Fi↔LTE switch, 5/30/200 s outages, gateway restart |
| E-08..E-10 | 04-scale-security | 100 real clients, 400-identity handshake load (c=1/8/32), replay, floods. gateway.log, the pcap and the timing csv are gzipped |
| E-10 | 04b-reassembly-dos | first-fragment flood (reassembly-table exhaustion) |
| E-11 | 05-soak | 60 min, 20 clients, voice, 1 ping/s per client, DC fetch every 30 s |
| E-12 | tools | bench scripts, hsload (Go handshake load generator), rtpvoice.py, hsanalyse.py, metrics.py |

## Re-running
```
sudo bash tools/phase1-functional-dataplane.sh   # then phase2, phase3, phase3b, phase4, phase4b, phase5
python3 tools/metrics.py                          # consolidates metrics.json
```
`hsload` is built from `tools/hsload/main.go` placed under `tools/hsload/` in the pqcvpn module:
`CGO_ENABLED=0 go build -mod=vendor ./tools/hsload`.

## Notes
- The soak's `ping` outputs stop at about 3,535 of 3,600 samples because teardown stopped ping before its summary line. `tools/soak_analyse.py` counts from the per-packet lines instead.
- In `04-scale-security`, gateway "peer re-handshaked from a different address" warnings are an artefact of the load tool. Each hsload run opens new UDP ports for the same identities.
- Findings D1–D5 and O1 are listed in Annex C of the data sheet.
