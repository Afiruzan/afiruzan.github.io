---
title: "Muqarnas-NoC"
collection: portfolio
permalink: /portfolio/muqarnas-noc/
excerpt: "A hierarchical, reconfigurable Network-on-Chip architecture inspired by Muqarnas geometry."
date: 2026-10-30
---

## Overview

Muqarnas-NoC explores a structured, hierarchical interconnect for many-core accelerator systems. Its organization is inspired by the layered geometry of Muqarnas and is intended to support configurable communication paths between processing elements.

## Design direction

- A 16×16 processing-element organization (256 PEs) in the current RTL-oriented research baseline.
- Hierarchical grouping of processing elements into communication domains.
- Configurable lateral connections and express communication paths.
- Routing and switching logic intended to support isolation and flexible traffic movement.

## Implementation and evaluation

The current engineering workflow includes SystemVerilog RTL, Icarus Verilog smoke and unit tests, Verilator linting, and traffic-driven simulation development. Architectural comparisons must use matched traffic patterns, packet formats, topology assumptions, and measurement definitions. No performance advantage is claimed on this page without completed, reproducible measurements.

## Code and updates

Source code and development updates are available from the [Afiruzan GitHub profile](https://github.com/Afiruzan/).
