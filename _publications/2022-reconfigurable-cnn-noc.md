---
title: "Reconfigurable Network-on-Chip based Convolutional Neural Network Accelerator"
collection: publications
category: manuscripts
permalink: /publication/2022-reconfigurable-cnn-noc
date: 2022-08-01
venue: "Journal of Systems Architecture, Volume 129, Article 102567"
paperurl: "https://doi.org/10.1016/j.sysarc.2022.102567"
citation: "Arash Firuzan, Mehdi Modarressi, Midia Reshadi, and Ahmad Khademzadeh. Reconfigurable Network-on-Chip based Convolutional Neural Network Accelerator. Journal of Systems Architecture, 129, 102567 (2022)."
---

### Abstract

Convolutional neural networks (CNNs) deliver strong performance in image and pattern classification, but their computational demands and memory-bandwidth requirements make hardware acceleration important. Many-core CNN accelerators commonly connect processing elements (PEs) according to the dataflow of individual layers. Fixed accelerator structures, however, can underutilize resources when layer dimensions differ from the hardware configuration. This paper presents **RC-CNN**, a reconfigurable CNN accelerator that adapts its PE organization and interconnection pattern to the dimensions and dataflow of the layer being executed. A reconfigurable on-chip fabric organizes a subset of PEs into a shape suited to the target layer and customizes communication among them. To limit the overhead of fine-grained accelerators, the design uses simple reconfigurable switches instead of a full packet-switched NoC.

### Reported results

The publisher's abstract reports an average 37% improvement in PE utilization over a baseline and an average 18% improvement over selected state-of-the-art utilization-aware CNN accelerators. It also reports throughput improvements of 9%–41%, and reductions of 28% in network latency and 22% in energy relative to the compared utilization-aware packet-switched NoC methods. These are results reported by the paper, not measurements from the current Muqarnas-NoC project.

**Authors:** Arash Firuzan, Mehdi Modarressi, Midia Reshadi, and Ahmad Khademzadeh.

[DOI / Publisher page](https://doi.org/10.1016/j.sysarc.2022.102567) · [Google Scholar profile](https://scholar.google.com/citations?user=Ezu-7uYAAAAJ&hl=en) · [ScienceDirect abstract](https://www.sciencedirect.com/science/article/pii/S1383762122001175) · [DBLP record](https://dblp.org/rec/journals/jsa/FiruzanMRK22)
