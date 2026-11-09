---
title: "Muqarnas-NoC: Architecture, RTL & Simulation"
collection: portfolio
permalink: /portfolio/muqarnas-noc/
excerpt: "An evolving research log documenting the design, RTL verification, traffic-driven simulation, and reproducible evaluation of a hierarchical reconfigurable Network-on-Chip."
date: 2026-11-09
---

Muqarnas-NoC is an ongoing research project exploring a hierarchical, reconfigurable Network-on-Chip (NoC) for many-core accelerator systems. The project takes structural inspiration from the layered geometry of Muqarnas and investigates how configurable communication paths can support different traffic patterns and communication domains.

This page is a research and engineering log. It distinguishes implemented components, measured results, diagnostic observations, and planned experiments. Results are not presented as final architectural claims unless the corresponding experiment has completed under a documented methodology.

## 1. Research objectives

The project investigates whether a structured hierarchy with configurable lateral and express links can provide useful communication flexibility for accelerator-oriented processing-element (PE) arrays.

The main objectives are:

- Develop a hierarchical NoC organization with configurable communication paths.
- Implement and verify the switching, routing, buffering, and arbitration logic in RTL.
- Evaluate baseline interconnect topologies using repeatable traffic-driven simulations.
- Compare architectures under matched traffic traces and consistent router and packet assumptions.
- Quantify latency, accepted throughput, hop count, congestion, and reconfiguration overhead.
- Extend the architecture toward larger PE arrays after the current implementation and evaluation stages are validated.

## 2. Architecture and implementation scope

### Current RTL prototype

The current RTL prototype is a **4×4 network with 16 processing elements**, organized into four communication domains. It includes hierarchical communication links and configurable lateral/express paths. The RTL work includes switch configuration, routing, FIFO buffering, round-robin arbitration, and configuration updates coordinated with network traffic.

The configurable domain-level links include horizontal, vertical, and diagonal express connections. The configuration logic prevents simultaneous activation of the two crossing diagonal paths where that combination is disallowed by the design constraints.

### Planned scale-up

A **16×16 array (256 PEs)** is a planned larger-scale research target. It must not be confused with the current 4×4 RTL prototype or with the 4×4 BookSim baseline experiments described below. Results from the smaller network do not establish performance at 256 PEs.

## 3. Simulation and verification environment

The experiments are conducted locally in an Ubuntu virtual machine running under VMware Workstation. The workflow uses the following tools:

| Tool | Role |
|---|---|
| Icarus Verilog | RTL compilation and simulation testbenches |
| Verilator | RTL lint and static checks |
| BookSim 2 | Traffic-driven evaluation of baseline NoC topologies |
| Python 3 | Experiment orchestration, log parsing, aggregation, and plotting |

The RTL and BookSim experiments are treated as separate evaluation tracks. BookSim baseline measurements are not described as Muqarnas RTL measurements. A direct comparison requires a validated Muqarnas BookSim model or a validated trace-driven adapter with compatible timing and traffic semantics.

## 4. Research workflow

The evaluation is organized into the following stages.

### Stage 1 — Architecture specification

Define the PE organization, communication domains, hierarchical links, configurable express paths, routing constraints, and switch configuration behavior.

**Output:** a documented 4×4 RTL prototype specification and a separate plan for the 16×16 scale-up.

### Stage 2 — RTL implementation

Implement the network components, including routers/switches, FIFOs, arbitration, routing decisions, and configuration control.

**Output:** synthesizable RTL and testbench infrastructure.

### Stage 3 — RTL verification

Run smoke tests, unit tests, and network-level testbenches. Use linting to identify structural and width-related issues.

**Recorded result:** the RTL regression reported 17 passing tests, 0 failing tests, and 0 test errors over 12,863 cycles. Verilator lint completed with warnings under the relaxed warning policy; width warnings remain and are not represented as a warning-free lint result.

### Stage 4 — Baseline topology benchmarking

Build BookSim 2 and evaluate three 4×4 baseline topologies:

- 4×4 Mesh
- 4×4 Torus
- 4×4 Flattened Butterfly

The current baseline traffic configuration uses uniform traffic, 5-flit packets, 4 virtual channels, 4-flit VC buffers, and unit input/output speedup. Experiments use multiple random seeds. The BookSim runs are conducted independently of the RTL regression.

### Stage 5 — Multi-rate and repeatability experiments

Run each baseline topology across several injection rates and seeds. Preserve raw logs and CSV results so that unstable runs are not silently discarded or treated as successful measurements.

### Stage 6 — Saturation-region investigation

Investigate the transition from stable operation to unstable or non-convergent operation. A run that aborts because average latency exceeds the configured threshold is classified as unstable; it is not automatically classified as a deadlock.

### Stage 7 — Muqarnas and competitor comparison

Integrate a validated Muqarnas model into the same evaluation methodology. Use identical packet traces (source, destination, and injection time), packet formats, router pipeline assumptions, buffer/VC settings, arbitration assumptions, and measurement definitions wherever the compared mechanisms permit.

SMART and Express Virtual Channels (EVC) require faithful models of their respective mechanisms; they are not to be represented as simple interchangeable topologies. Area and energy comparisons require appropriate synthesis or physical/power models and cannot be inferred from BookSim latency output alone.

## 5. RTL verification status

| Check | Recorded status |
|---|---|
| Icarus Verilog smoke test | Passed (RTL_SIMULATION_OK) |
| RTL regression | 17 passed, 0 failed, 0 errors |
| Regression duration | 12,863 cycles |
| Verilator lint | Completed with warnings under -Wno-fatal |
| Remaining lint issue | Width warnings require review |

These are verification results for the current RTL test environment. They do not, by themselves, establish throughput, latency, area, or energy advantages over another NoC.

## 6. BookSim baseline results

### 6.1 Repeated experiment at injection rate 0.02

The following values are the recorded means from 10 seeds (101–110) for each 4×4 baseline topology.

| Topology | Mean packet latency (cycles) | Mean network latency (cycles) | Mean flit latency (cycles) | Mean hops |
|---|---:|---:|---:|---:|
| Mesh 4×4 | 25.5579 | 25.0685 | 19.6622 | 3.5005 |
| Torus 4×4 | 26.5697 | 26.0781 | 19.4253 | 2.9955 |
| Flattened Butterfly 4×4 | 25.9944 | 25.5052 | 17.5026 | 2.4992 |

At this injection rate, Mesh has the lowest recorded mean packet latency, while Flattened Butterfly has the lowest recorded mean flit latency and hop count. These metrics do not identify a single overall winner.

### 6.2 Multi-rate experiment

A 180-run experiment was configured for three topologies, six injection rates (0.01, 0.02, 0.05, 0.10, 0.15, and 0.20), and ten seeds (101–110).

| Injection rate | Completed runs | Unstable runs |
|---:|---:|---:|
| 0.01 | 30 | 0 |
| 0.02 | 30 | 0 |
| 0.05 | 30 | 0 |
| 0.10 | 30 | 0 |
| 0.15 | 0 | 30 |
| 0.20 | 0 | 30 |
| **Total** | **120** | **60** |

At 0.15 and 0.20, BookSim reported latency-threshold/convergence stops, including messages such as "Average latency for class 0 exceeded 500 cycles" and "Too many sample periods needed to converge". These runs are classified as unstable/non-convergent, not as proven deadlocks. The raw logs retain diagnostic metrics, but they are excluded from stable-run summary statistics.

### 6.3 Diagnostic runs at high injection rates

A follow-up diagnostic used seed 101 at injection rates 0.15 and 0.20. BookSim produced metrics before stopping as unstable.

| Topology | Requested injection | Accepted packet rate | Packet latency (cycles) | In-flight flits at stop |
|---|---:|---:|---:|---:|
| Mesh | 0.15 | 0.128744 | 704.721 | 352 |
| Mesh | 0.20 | 0.128506 | 1803.63 | 373 |
| Torus | 0.15 | 0.128112 | 724.638 | 293 |
| Torus | 0.20 | 0.128275 | 1810.75 | 292 |
| Flattened Butterfly | 0.15 | 0.133550 | 547.919 | 266 |
| Flattened Butterfly | 0.20 | 0.134894 | 1652.93 | 269 |

These are diagnostic observations from unstable runs, not final saturation-throughput measurements. The near-constant accepted rates alongside sharply increasing packet latency are consistent with congestion near or beyond the saturation region. More seeds and stable-region measurements are required before reporting a saturation point.

## 7. Reproducibility and data organization

The benchmark workflow preserves configurations, logs, and tabular outputs under the local project directory:

- results/research_benchmark/raw_results.csv — per-run multi-rate results.
- results/research_benchmark/summary.csv — aggregated baseline summary.
- results/research_benchmark/configs/ — generated BookSim configurations.
- results/saturation_diagnostic/ — high-injection diagnostic configurations, logs, and CSV.
- results/BOOKSIM_BIN_PATH.txt — path to the locally built BookSim executable.

The local Ubuntu VM is the execution environment. The website reports selected results and methodology; it does not claim that the simulation itself runs in the visitor's browser.

## 8. Current status and next experiments

**Completed or recorded**
- 4×4 RTL prototype and testbench workflow.
- RTL regression with 17 passing tests.
- Verilator lint run, with width warnings still open.
- BookSim 2 build.
- 10-seed baseline comparison at injection rate 0.02.
- 180-run multi-rate baseline campaign, with 120 completed and 60 unstable.
- High-injection diagnostic runs for seed 101.

**In progress**
- Refine the saturation-region sweep between injection rates 0.10 and 0.15.
- Repeat the transition-region experiments over multiple seeds.
- Improve run classification and convergence reporting.
- Review remaining RTL lint width warnings.

**Planned**
- Add a validated Muqarnas model to the common benchmark methodology.
- Generate and reuse identical packet traces across compared architectures.
- Add faithful SMART and EVC evaluation models.
- Evaluate reconfiguration overhead and domain isolation.
- Extend the validated design toward a 16×16 (256-PE) architecture.
- Evaluate area and energy using suitable synthesis and power-analysis flows.

## 9. Research integrity

The values on this page are tied to the experiment stages described above. Unstable runs are explicitly identified, diagnostic observations are not promoted to final performance claims, and the current 4×4 prototype is distinguished from the planned 16×16 scale-up. No claim of Muqarnas superiority is made until the Muqarnas model and competing architectures have been evaluated under a fair, reproducible methodology.

## 10. Source code and project updates

The project source and engineering updates are associated with the [Afiruzan GitHub profile](https://github.com/Afiruzan/). This page will be updated as additional experiments complete and their raw data and analysis are validated.


## 11. API connectivity and autonomous workflow update (7 November 2026)

The local API setup has reached the connectivity-check stage. Both OpenAI and DeepSeek API key authentication succeeded, and each service returned its available model list. A minimal text-generation request was then attempted against each provider:

| Provider | Authentication / model listing | Text generation test |
|---|---|---|
| OpenAI | Successful | HTTP 429: no API credits remaining |
| DeepSeek | Successful | HTTP 402: insufficient balance |

Both API keys were replaced after they had been exposed in a screenshot. The replacement credentials are kept private and are not included in this repository. The generation tests have not yet succeeded; the current blocker is account balance, not a demonstrated Python SDK or Ubuntu connectivity problem. The tests should be repeated after the API accounts are funded. API billing is separate from a ChatGPT subscription.

The intended multi-agent workflow will use OpenAI for research direction and analysis, DeepSeek for coding/debugging assistance, and deterministic local checks for validation. LangGraph is installed in the local Python virtual environment as a candidate orchestrator. The agent system itself has **not** yet been implemented or validated, and no autonomous experiment execution is claimed.

**Execution policy:** all code editing, compilation, BookSim runs, RTL simulations, data analysis, and result storage remain on the researcher's own Ubuntu VMware virtual machine. The APIs provide model inference only; they are not used as remote simulation machines. Credentials are stored in a private `.env` file and must never be committed to Git or shown in screenshots.

## 12. Updated next steps

1. Fund the OpenAI and DeepSeek API accounts, then rerun the minimal generation test.
2. Implement the first agent task as a read-only audit of the existing local Muqarnas-NoC project and its scripts.
3. Reproduce the existing RTL regression before permitting code modifications or new benchmark runs.
4. Add command allowlists, backups/Git branches, bounded retries, serial execution, and explicit spending limits before enabling autonomous actions.
5. Continue the saturation-region sweep and review outstanding RTL width warnings.
6. Add a validated Muqarnas model and matched traffic traces before making comparative performance claims.


## 13. RTL open-loop benchmark update (9 November 2026)

A new **RTL-level open-loop benchmark** has been completed on the 4×4, 16-PE prototype. The experiment compares the tree-only configuration with the express-link configuration under matched traffic: for each workload and seed, the same generated packet trace is replayed in both configurations. The runner also checked that each replay used an identical trace.

### Experiment setup

- Three traffic patterns: uniform, transpose, and hotspot (PE 5).
- Two random seeds per pattern (seeds 1 and 2).
- Two network configurations: tree-only and express enabled.
- 500 measurement cycles after 100 warm-up cycles; injection probability 0.20 per PE per cycle.
- Source queue depth 128; overflow policy set to `fail`.
- Receiver-side artificial stalls disabled (`rx_stall_cycles=0`).
- Total: **12 RTL simulation cases** (3 patterns × 2 seeds × 2 configurations).

All 12 cases completed with `OPENLOOP_RESULT: PASS`. Across the cases, queued = accepted = delivered, with zero dropped packets, zero undelivered packets, zero protocol errors, zero policy errors, and completed drain. The generated traffic traces were reused for each tree-only/express pair.

### Mean results across the two seeds

Latency values are in RTL simulation cycles. Throughput is delivered packets per measurement cycle.

| Traffic pattern | Configuration | Network latency | End-to-end latency | Delivered throughput |
|---|---|---:|---:|---:|
| Uniform | Tree-only | 9.5732 | 11.6296 | 3.191 |
| Uniform | Express | 9.0237 | 10.4894 | 3.200 |
| Transpose | Tree-only | 4.0937 | 4.0944 | 3.231 |
| Transpose | Express | 4.0937 | 4.0944 | 3.231 |
| Hotspot | Tree-only | 31.4756 | 204.0272 | 2.098 |
| Hotspot | Express | 30.3791 | 164.1685 | 2.239 |

Relative to tree-only, the express configuration reduced mean end-to-end latency by **9.80%** under uniform traffic and **19.54%** under hotspot traffic. Under hotspot traffic, delivered throughput increased by **6.72%** and mean source-queue waiting time decreased by **22.46%**. Under transpose traffic, the two configurations produced identical measured averages in this experiment.

### Interpretation and limitations

These results show a workload-dependent benefit in this particular 4×4 RTL experiment; they do **not** establish a universal advantage for express links or overall superiority over other NoC architectures. Only two seeds and one injection setting were tested. There was no receiver-side stall injection, no injection-rate sweep or saturation study in this RTL campaign, and no matched RTL comparison against external competitor implementations. The transpose result is reported as measured; no unverified explanation is assigned to the lack of change.

The reported latency and throughput are RTL testbench measurements, separate from the previously reported BookSim 2 baseline results. They must not be mixed or presented as directly comparable measurements. The 16×16 (256-PE) design remains a scale-up target, not the configuration measured here.

### Updated project status

**Completed**
- Open-loop RTL benchmark: 12/12 cases passed.
- Matched tree-only/express traffic traces for all three traffic patterns and both seeds.
- Packet accounting and drain checks passed in every case.

**Still required for broader performance claims**
- Additional seeds and injection-rate sweep, including the congestion/saturation region.
- Controlled receiver-stall/backpressure experiments.
- A validated Muqarnas model and fair, matched comparisons with competitor architectures.
- Continued review of outstanding Verilator width warnings.

The raw run directories, per-run CSV metrics, manifests, simulator logs, and traffic traces are retained locally under `results/v2/20261108T192622Z-e6f0f112/` in the Ubuntu project environment. They are not claimed to be hosted on this website.
