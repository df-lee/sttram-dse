# STT-RAM Design Space Exploration

Area-constrained design space exploration of a 16 MB STT-RAM memory using NVSim.

This project studies how device-level assumptions and memory-organization choices jointly affect area feasibility, latency, energy, and balanced design quality under a hard area constraint.

## Project Overview

STT-RAM is attractive for large on-chip memories because of its density advantage and negligible leakage. However, the final memory-macro quality is not determined by device parameters alone. Organization-level choices such as muxing and mat partitioning also affect peripheral overhead, wiring, validity, latency, and energy.

This project uses NVSim to explore a fixed-capacity 16 MB STT-RAM memory. The workflow first defines a nominal reference point, then screens device- and organization-level parameters, and finally performs a constrained design-space sweep to identify feasible regions and latency-energy tradeoffs.

## Workflow

```text
Nominal 16 MB STT-RAM reference
        ↓
Device / organization parameter screening
        ↓
Selection of CellArea, OutputMux, and MatOrganization
        ↓
Constrained design-space sweep
        ↓
Validity and area-feasibility filtering
        ↓
Pareto and balanced-design analysis
```

## Reference Setup

| Item | Setting |
|---|---|
| Memory target | 16 MB STT-RAM macro |
| Modeling tool | NVSim |
| Mode | RAM target |
| Data width | 128 bits |
| Process / roadmap | 22 nm HP |
| Reference objective | ReadEDP |
| Reference organization | 8×8 banks, 2×2 mats |
| Subarray size | 1024×512 |
| Muxing | SenseAmpMux = 16, OutputMux = 1/1 |
| Area budget | 1.10× nominal reference area = 4.8488 mm² |

## Screening Stage

The first stage screens candidate parameters before the final sweep.

### Device-side candidates

| Parameter group | Interpretation |
|---|---|
| `CellArea` | STT-RAM cell-area / density assumption |
| `WriteCurrent` | Reset/set current scaling |
| `WritePulse` | Reset/set pulse-width scaling |
| `ResistanceLevel` | On/off resistance scaling |
| `AccessWidth` | Access transistor width scaling |

The device-level screening identifies `CellArea` as the leading device-side variable. It mainly controls the area-feasibility envelope, but also produces measurable latency and energy changes.

### Organization-side candidates

| Parameter group | Interpretation |
|---|---|
| `OutputMux` | Grouped output-path muxing levels |
| `MatOrganization` | Mat-level array partitioning |
| `MuxSenseAmp` | Sense-amplifier muxing |
| `Routing` | Interconnect style |

The organization-level screening identifies `OutputMux` as the strongest organization-side variable, mainly through dynamic-energy impact. `MatOrganization` is retained as a secondary variable because it provides a clear array-partitioning interpretation.

## Final Sweep Space

| Parameter | Values |
|---|---|
| `CellArea` scale | 0.2 to 1.3, step 0.1 |
| `OutputMux` | 1, 2, 4, 8, 16 |
| `MatOrganization` | 1×1, 2×2, 4×4, 8×8, 16×16 |

## Sweep Summary

| Category | Count |
|---|---:|
| Attempted design points | 300 |
| Complete NVSim outputs | 101 |
| Area-feasible designs | 68 |
| Pareto-efficient designs | 6 |

Invalid or incomplete points indicate configurations for which NVSim did not generate complete area, latency, and energy outputs under the forced organization constraints.

## Evaluation Metrics

Average latency and average dynamic energy are defined as:

```text
L_avg = (L_read + L_write) / 2
E_avg = (E_read + E_write) / 2
```

The balanced design is selected using an equal-weight normalized latency-energy score:

```text
Score_LE = 0.5 × L_avg / L_avg,0 + 0.5 × E_avg / E_avg,0
```

This score is used only to select one representative balanced design from the feasible tradeoff region. Pareto-efficient designs define the latency-energy boundary.

## Key Results

### Selected balanced design

| Item | Value |
|---|---:|
| `CellArea` scale | 0.5× |
| `OutputMux` | 2 |
| `MatOrganization` | 4×4 |
| Area | 2.603 mm² |
| Average latency | 7.236 ns |
| Average dynamic energy | 182.436 pJ |
| `Score_LE` | 0.949 |

This design remains below the hard area budget while improving average latency relative to the nominal reference, with only a moderate energy increase.

### Pareto-efficient designs

| Role | CellArea | OutMux | Mat | Area (mm²) | L_avg (ns) | E_avg (pJ) |
|---|---:|---:|---:|---:|---:|---:|
| Min Latency | 0.2× | 4 | 8×8 | 1.314 | 6.640 | 290.451 |
| Pareto Point | 0.4× | 2 | 1×1 | 2.687 | 7.229 | 205.831 |
| Best Balanced | 0.5× | 2 | 4×4 | 2.603 | 7.236 | 182.436 |
| Pareto Point | 1.0× | 1 | 8×8 | 4.137 | 7.726 | 173.634 |
| Pareto Point | 0.8× | 1 | 2×2 | 3.693 | 7.796 | 172.244 |
| Min Energy | 1.0× | 1 | 4×4 | 4.155 | 8.022 | 172.020 |

## Main Takeaways

- `CellArea` controls the area-feasibility envelope, but better density does not uniformly improve all metrics.
- `OutputMux` introduces strong validity and energy effects.
- `MatOrganization` shifts where the best feasible latency-energy points appear.
- The best design is determined by device-architecture interaction, not by a single independent knob.
- The lowest-latency point is not selected as the balanced design because its energy cost is much higher.

## Repository Contents

```text
.
├── README.md
└── sttram-area-constrained-dse-report.pdf
```

## Tool

- [NVSim](https://github.com/SEAL-UCSB/NVSim)
