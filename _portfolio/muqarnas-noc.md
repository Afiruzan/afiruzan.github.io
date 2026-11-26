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


## 14. RTL quality-gate and regression update (14 November 2026)

The 4×4 (16-PE) RTL prototype has undergone an additional quality-control pass on the researcher's Ubuntu VMware environment. This update records the checks completed and keeps the remaining regression-runner issue explicit.

### Static lint and Generic synthesis

- **Verilator 5.032:** the standalone quality-gate lint completed with exit code 0 and **zero warnings** after a narrowly scoped cleanup of width and unused-signal warnings.
- **Yosys 0.52 Generic synthesis:** completed successfully and emitted a generic Verilog netlist. The post-cleanup report lists **64,804 generic cells**, including 14,913 sequential cells and 5,731 generic multiplexers. These are technology-independent synthesis statistics, not standard-cell area, power, timing, or silicon measurements.
- The earlier inferred-latch issue in the round-robin arbiter was removed by assigning a default value to its procedural loop variable. The subsequent Yosys log contains no `Latch inferred` message and reports zero problems in its structural `check` passes.
- Yosys still emits one unique warning (twice in the log): the FIFO array `mem` is replaced with a list of registers. This is recorded rather than suppressed; it means this generic flow did not preserve the array as a memory macro.

### Functional regression

The quick RTL regression produced the following results:

| Test | Recorded status |
|---|---|
| `original_route_tb` | PASS |
| `route_exh_fixed` | PASS |
| `route_exh_original` | EXPECTED_FAIL (known FIX-1 behavior in the original implementation) |
| `unit_fifo` | PASS |
| `unit_arb` | PASS |
| `noc16` | PASS |

These results support the continued function of the FIFO, corrected arbiter, fixed routing test, and 16-PE network testbenches after the arbiter cleanup. The expected failure of the original routing test is not counted as a failure of the corrected test.

### Open issue in the aggregate regression script

The same `run_all.sh quick` invocation recorded Verilator as `WARN_OR_FAIL` because its lint command passes the include path through a shared, space-separated argument variable. This differs from the standalone quality-gate invocation, which passes. Inspection also showed that the aggregate script records the Verilator return code but does not propagate that failure into its overall exit status. Consequently, the aggregate command returned exit code 0 despite the failed lint substage.

This is classified as a **test-runner/reporting defect under investigation**, not as evidence of an RTL functional failure. The script has not yet been corrected, and a clean full regression through the aggregate runner has therefore **not** been claimed.

### Current evidence-based status

- Standalone Verilator lint: passed, zero warnings.
- Yosys Generic synthesis: passed; no inferred latch; one documented memory-to-register warning.
- Quick functional tests: five PASS results and one documented EXPECTED_FAIL for the original buggy routing variant.
- Aggregate regression runner: requires correction so include arguments are passed reliably and any failed stage causes a non-zero overall exit status.
- No new claims are made about physical area, energy, clock frequency, saturation throughput, or superiority over competing NoCs.

All reported checks were run locally by the researcher. The website records the supplied terminal outputs; it does not imply that the simulations were executed by GitHub or by this website.


## 15. Consolidated simulation and verification report (24 November 2026)

This section consolidates the latest available engineering and simulation records. It separates RTL measurements, Python model-only results, BookSim baseline experiments, and the preliminary Muqarnas BookSim integration. These evidence classes are not interchangeable.

### 15.1 Current architecture represented in RTL

The implemented prototype is a 4×4 array of 16 processing elements (PEs), arranged in four domains. The hierarchy contains one root switch, four regional/domain switches, and 16 PE tile interfaces (21 switch/router nodes in the graph abstraction). The fixed hierarchical backbone has 20 bidirectional logical links. The design exposes 24 candidate lateral/express links (six per domain); the configuration constraints prohibit activating both crossing diagonals simultaneously, allowing at most 20 lateral links to be active.

The configuration `24'h7df7df` activates four axial links and one main diagonal in each domain, for 20 active lateral links. The current RTL uses 40-bit flits, single-flit packets, and FIFO depth four. The PE-to-domain mapping is:

| Domain | PE identifiers |
|---|---|
| D0 | 0, 1, 4, 5 |
| D1 | 2, 3, 6, 7 |
| D2 | 8, 9, 12, 13 |
| D3 | 10, 11, 14, 15 |

An important modeling constraint is that the current `mq_tile.sv` has terminal-only behavior: a tile injects toward its domain hierarchy or directly to a connected destination through an enabled lateral link, and incoming packets eject to the PE. Tiles do not forward transit packets. Consequently, a generic shortest-path model that routes packets through PE tiles as intermediate routers would not be semantically equivalent to this RTL.

### 15.2 RTL functional verification

The recorded RTL verification history includes a regression with 17 passing tests, zero failures, and zero test errors (one archived run reported 12,860 cycles). Other quick-regression records identify the individual test outcomes as follows:

| Test | Recorded status |
|---|---|
| `original_route_tb` | PASS |
| `route_exh_fixed` | PASS |
| `route_exh_original` | EXPECTED_FAIL (known original-variant FIX-1 behavior) |
| `unit_fifo` | PASS |
| `unit_arb` | PASS |
| `noc16` | PASS |

A route-exhaustive test explored 368,896 walks without observing an illegal direct-edge traversal. These are functional verification observations, not performance comparisons against external NoCs.

### 15.3 Open-loop RTL measurements

The completed 12-case open-loop campaign compared tree-only and express-enabled configurations for three synthetic traffic patterns (uniform, transpose, and hotspot), two seeds per pattern, and a 20% injection probability. Each express run replayed the same generated traffic trace as its tree-only counterpart. The measurement window was 500 cycles, preceded by 100 warm-up cycles.

All 12 cases completed with packet accounting equality and drain completion. The campaign recorded no dropped or undelivered packets and no protocol/policy errors. Receiver-side artificial stalls were disabled.

| Traffic | Tree-only mean E2E latency | Express mean E2E latency | E2E latency change | Delivered throughput change |
|---|---:|---:|---:|---:|
| Uniform | 11.6296 cycles | 10.4894 cycles | −9.80% | +0.28% |
| Transpose | 4.0944 cycles | 4.0944 cycles | 0.00% | 0.00% |
| Hotspot | 204.0272 cycles | 164.1685 cycles | −19.54% | +6.72% |

For hotspot traffic, mean source-queue waiting time also decreased by 22.46%. These results indicate workload-dependent effects in this specific 4×4 RTL experiment. They do not establish general superiority, and they are not a comparison against external competitor implementations. The campaign used two seeds and one injection setting; it did not include a full RTL injection-rate/saturation sweep or receiver-stall campaign.

A separate 30% injection-rate attempt was incomplete: only eight cases were present, with hotspot cases missing and the summary CSV containing only its header. It is not a valid aggregate result and is excluded from performance claims.

### 15.4 Python model-only experiments

The project contains Python cycle/model and route-audit scripts. Their outputs are explicitly model-only and must not be presented as RTL measurements. A historical model-only report recorded 48 runs without loss or deadlock under its modeled conditions, with illustrative route-delay observations (for example, PE0→PE15 at four cycles and selected nearest-neighbor cases improving from 3.27 to 2.32 cycles with express links). These results depend on the Python model's assumptions and do not demonstrate corresponding behavior in the RTL or BookSim implementation.

### 15.5 Verilator, Yosys, and RTL quality gates

A standalone Verilator lint run passed with zero warnings after cleanup of width-truncation and unused-signal warnings. Yosys Generic synthesis completed successfully after a default assignment was added to the round-robin arbiter's procedural loop variable; subsequent logs showed no inferred latch and zero problems in structural `check` passes.

One archived Generic synthesis report recorded 64,823 cells, 49,561 wires, and 94,613 wire bits. The reported sequential-cell and mux counts are technology-independent synthesis statistics. They are not standard-cell area, physical timing, power, energy, or silicon measurements. Yosys also reported that the FIFO array memory was replaced with a list of registers; this is a documented generic-mapping warning.

The aggregate `run_all.sh quick` script still has a known reporting defect in the recorded version: its Verilator include-path argument is split incorrectly, and the script does not propagate that substage's failure into its overall exit code. Therefore, a zero exit code from that aggregate invocation is not sufficient evidence that every substage passed. The independent Verilator and Yosys quality gates should be considered separately until the runner is repaired and rerun.

### 15.6 BookSim 2 baseline experiments

BookSim 2 baseline experiments evaluated 4×4 Mesh, Torus, and Flattened Butterfly networks. A historical 120-run saturation-region campaign used three topologies, four injection rates (0.11–0.14), and ten seeds. At injection rate 0.11, mean packet latencies were 46.44 cycles (Mesh), 45.35 cycles (Torus), and 40.61 cycles (Flattened Butterfly). At 0.12, the corresponding means were 70.26, approximately 66.20, and 51.51 cycles. At 0.13, Mesh and Torus were unstable while Flattened Butterfly remained stable at a mean of 104.05 cycles; at 0.14, all three were unstable.

In these experiments, “unstable” means BookSim stopped after the configured latency threshold was exceeded. It is not proof of deadlock. These are BookSim baseline-only results, not Muqarnas measurements. Earlier BookSim logs also contained a return-code convention issue (the supplied upstream `main.cpp` returned -1 on successful completion), so any result affected by that runner behavior must be interpreted with its raw log and exit-code handling in view.

### 15.7 Preliminary Muqarnas BookSim integration

A source-aware terminal-only routing prototype was added to a detached BookSim source copy to represent the 21-node Muqarnas graph abstraction with 16 terminals, 20 backbone links, and either zero or 20 active lateral links. The Tree and Express graph-generation smoke workflow printed the expected PE/domain mapping and graph counts.

This integration is preliminary. Its smoke test used BookSim's stock traffic generation rather than byte-for-byte replay of the RTL traffic traces. It is therefore not yet a fair, validated comparison against the RTL or competing NoC architectures.

The most recent user-side smoke attempt stopped with `BookSim exited with 255`. The patch had been installed, but the user had not yet rebuilt the BookSim binary after applying it. The next required diagnostic step is to rebuild the patched source and rerun the smoke test; no successful end-to-end Muqarnas BookSim benchmark is claimed at this stage.

### 15.8 Current evidence status and next steps

**Established by recorded runs**
- A 4×4, 16-PE RTL prototype and its functional testbench infrastructure.
- Historical RTL regression and exhaustive route-test results.
- A 12-case matched-trace RTL open-loop experiment with the workload-specific results above.
- Standalone Verilator lint with zero warnings and successful Yosys Generic synthesis after the arbiter fix.
- BookSim baseline-only experiments for Mesh, Torus, and Flattened Butterfly.
- A preliminary source-aware Muqarnas AnyNet/terminal-only routing prototype and graph-generation smoke workflow.

**Not yet established**
- A completed, validated Muqarnas BookSim benchmark.
- A matched-trace, same-assumption comparison of Muqarnas against external competitors.
- A full RTL injection-rate/saturation study with multiple seeds and backpressure.
- Physical area, power, energy, or maximum clock frequency.
- Performance at 16×16 (256 PEs); this remains a scale-up target.

**Immediate next steps**
1. Rebuild the patched BookSim binary and rerun the Tree/Express smoke tests, preserving complete logs and exit codes.
2. Repair `run_all.sh` so include paths are passed correctly and any failed substage produces a non-zero overall status.
3. Validate terminal-only routing semantics against RTL behavior before using the BookSim model for comparative claims.
4. Implement exact packet-trace replay (source, destination, and injection cycle) across Muqarnas and each competitor, with documented differences in router pipeline and buffering assumptions.
5. Expand seeds and injection-rate sweeps, add receiver backpressure, and report confidence intervals and stable/unstable classifications.
6. Use appropriate synthesis and power-analysis flows for area/energy claims.

The central research-integrity rule remains unchanged: RTL measurements, Python model-only outputs, BookSim baseline results, and preliminary Muqarnas BookSim smoke tests are reported as separate evidence classes. No overall superiority claim is made until a validated, reproducible, same-condition comparison has been completed.


## 16. Latest BookSim smoke and static route audit (26 November 2026)

This addendum supersedes the earlier preliminary status in Section 15.7 where the first post-patch smoke attempt had exited with code 255. A subsequent run on 25 November 2026 completed successfully.

### 16.1 Build and smoke-test outcome

The user reran the build command in:

`~/Downloads/muqarnas_benchmark/external/booksim2/src`

Make reported `Nothing to be done for 'all'` and returned `BUILD_EXIT_CODE=0`. The existing BookSim binary was present (reported size 17 MB; timestamp 25 November 2026, 14:43 local shell time).

The starter smoke script then generated both graphs and completed with:

- `SMOKE_TEST_PASS`
- `SMOKE_EXIT_CODE=0`

The generated result directory was:

`Muqarnas-NoC-RTL-Final/results/booksim_muqarnas_smoke_20261125T141409Z`

The graph generator reported the expected PE/domain mapping:
`{0:[0,1,4,5], 1:[2,3,6,7], 2:[8,9,12,13], 3:[10,11,14,15]}`.

| Graph | Routers | Terminals | Backbone links | Active lateral links |
|---|---:|---:|---:|---:|
| Tree-only | 21 | 16 | 20 | 0 |
| Express | 21 | 16 | 20 | 20 |

The final printed BookSim packet-latency averages were 23.9984 cycles for Tree-only and 23.1920 cycles for Express. This is approximately a 3.36% lower printed average for Express in this particular smoke configuration.

Both configuration files used the same settings except for the network-file path: AnyNet topology, `routing_function = muqarnas`, Uniform traffic, one VC, VC buffer size four, one-flit packets, injection rate 0.01, seed 1, latency simulation, sample period 5000, ten maximum samples, one warm-up period, and the same configured pipeline/allocation delays. The logs contained no matches for the checked warning/error/assertion/unstable/convergence/deadlock keywords.

These numbers remain smoke-test observations only. They are not a validated performance result, do not establish saturation behavior, and must not be used as evidence of superiority over external architectures. The run uses one seed and a low injection rate; it did not replay the exact RTL traffic trace.

### 16.2 Static all-pairs graph audit

A corrected static audit reconstructed the bidirectional Router-to-Router edges from the AnyNet files and checked all 256 source/destination pairs in each graph. It verified graph connectivity, terminal mapping, expected backbone/lateral link counts, and that PE routers are not used as intermediate transit routers in the constructed routes.

| Audit result | Tree-only | Express |
|---|---:|---:|
| Total undirected Router links | 20 | 40 |
| Backbone links | 20 | 20 |
| Lateral links | 0 | 20 |
| Source/destination pairs checked | 256 | 256 |
| Cross-domain hierarchical pairs | 192 | 192 |
| Same-domain hierarchical pairs | 48 | 8 |
| Direct lateral pairs | 0 | 40 |
| Self pairs | 16 | 16 |
| Static graph audit | PASS | PASS |

The audit generated:
- `tree_all_pairs_route_audit.csv`
- `express_all_pairs_route_audit.csv`

This is a static graph and expected-route audit. It does not by itself prove that the compiled BookSim routing callback actually selected the intended output port for every packet.

### 16.3 Routing-function registration discrepancy — unresolved

The inspected configuration files specify:

`routing_function = muqarnas;`

However, the inspected `networks/anynet.cpp` registration line is:

`gRoutingFunctionMap["muqarnas_anynet"] = &muqarnas_anynet;`

The source search output showed this `muqarnas_anynet` registration, while the smoke configuration uses the different string `muqarnas`. The actual callback implementation is located later in the file (reported around line 560), and the source-aware route table is built using both `Flit::src` and `Flit::dest`. However, the supplied diagnostic output did not include the callback body or establish where the `muqarnas` name is registered/resolved.

This discrepancy must be resolved before treating the smoke run as a validated Muqarnas routing experiment. A successful process exit and latency output alone do not prove that the intended callback was selected. The next verification must inspect the full routing-function registration/dispatch path and the callback body, then run a small deterministic source/destination test that confirms the selected output ports.

### 16.4 Updated evidence status

**Now established**
- The patched BookSim executable was available and the Make build stage returned zero.
- The Tree-only and Express smoke workflows both completed with exit code zero.
- The generated graph sizes and link counts match the intended 21-router/16-terminal abstraction.
- Static all-pairs graph audits passed for 256 pairs per graph.
- The smoke logs contained no matches for the checked warning/error/assertion/unstable/convergence/deadlock terms.

**Still not established**
- That `routing_function = muqarnas` resolves to the intended `muqarnas_anynet` callback.
- That the compiled callback chooses the intended output port for every source/destination pair.
- A validated BookSim performance characterization of Muqarnas.
- A fair same-trace comparison against Mesh, Torus, Flattened Butterfly, or other published architectures.
- Saturation throughput, confidence intervals across multiple seeds, or backpressure behavior.

**Next actions**
1. Resolve the routing-function name/registration discrepancy and inspect the complete callback implementation.
2. Add deterministic callback/route-selection checks for representative same-domain, cross-domain, direct-lateral, and self-destination packets.
3. Only after these checks pass, run multiple seeds and an injection-rate sweep for Tree-only and Express.
4. Implement exact trace replay and controlled competitor comparisons, with explicit documentation of packet format, traffic timing, router pipeline, buffering, and measurement windows.

The approximately 3.36% smoke-test latency difference is retained as a preliminary observation only. It is not promoted to a research performance claim.
