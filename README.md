# STT-RAM Design Space Exploration

Area-constrained design space exploration of a **16 MB STT-RAM memory** using **NVSim**.

This project studies how device-level assumptions and memory-organization choices interact under a hard area budget. The goal is not just to find a fast or low-energy point, but to understand the feasible design region and identify a balanced latency-energy design.

## Project Story

STT-RAM is attractive for large on-chip memory because of its high density and negligible leakage. However, memory quality is shaped by both device-side assumptions and architecture-side organization. A smaller memory cell may improve density, but it can also change latency and energy. Similarly, muxing and mat partitioning can shift the tradeoff by changing peripheral overhead and internal access paths.

This project follows a staged exploration flow:

1. Define a nominal 16 MB STT-RAM reference in NVSim.
2. Screen device-level and organization-level parameter groups.
3. Select the most meaningful parameters for a final constrained sweep.
4. Filter invalid and over-budget designs.
5. Analyze latency-energy Pareto tradeoffs and select a balanced design.

## Methodology

```text
Nominal 16 MB STT-RAM reference
        ↓
Sensitivity / impact screening
        ↓
Select CellArea, OutputMux, and MatOrganization
        ↓
Constrained parameter sweep
        ↓
Validity and area-feasibility filtering
        ↓
Pareto frontier and balanced-design selection
```

The nominal reference uses a 16 MB STT-RAM memory macro in NVSim with RAM mode, 128-bit data width, 22 nm HP roadmap, and ReadEDP as the reference optimization target.

The hard area budget is set to:

$$
A_{budget} = 1.10 A_0 = 4.8488\ \mathrm{mm}^2
$$

where \(A_0\) is the nominal reference area.

## Parameter Screening

Before the final sweep, the project screens candidate parameters so that the design space is not chosen arbitrarily.

**Device-level screening** evaluates grouped STT-RAM cell parameters such as `CellArea`, `WriteCurrent`, `WritePulse`, `ResistanceLevel`, and `AccessWidth`. The result identifies `CellArea` as the leading device-side variable because it strongly affects area feasibility while also changing latency and energy.

**Organization-level screening** evaluates architecture parameters such as `OutputMux`, `MatOrganization`, `MuxSenseAmp`, and `Routing`. The result identifies `OutputMux` as the strongest organization-side variable, mainly through its dynamic-energy impact. `MatOrganization` is also retained because it provides a clean array-partitioning interpretation.

The final sweep therefore focuses on:

- `CellArea`
- `OutputMux`
- `MatOrganization`

## Final Sweep

The final sweep covers 300 attempted design points:

| Parameter | Sweep Range |
|---|---|
| `CellArea` scale | 0.2 to 1.3, step 0.1 |
| `OutputMux` | 1, 2, 4, 8, 16 |
| `MatOrganization` | 1×1, 2×2, 4×4, 8×8, 16×16 |

After running NVSim and parsing the outputs:

- **101** points produced complete NVSim area-latency-energy outputs.
- **68** points satisfied the hard area budget.
- **6** points were Pareto-efficient among the area-feasible designs.

Invalid or incomplete points are treated as outside the usable NVSim design space under the forced organization constraints.

## Latency-Energy Analysis

Each valid design is compared using average latency and average dynamic energy:

$$
L_{avg} = \frac{L_r + L_w}{2}
$$

$$
E_{avg} = \frac{E_r + E_w}{2}
$$

To select one representative balanced point from the feasible tradeoff region, the project uses an equal-weight normalized latency-energy score:

$$
Score_{LE}
=
0.5\frac{L_{avg}}{L_{avg,0}}
+
0.5\frac{E_{avg}}{E_{avg,0}}
$$

Pareto-efficient designs define the latency-energy boundary. The score is used only to pick one balanced representative from that feasible tradeoff region.

## Selected Balanced Design

The best balanced point is:

| Item | Value |
|---|---:|
| `CellArea` scale | 0.5× |
| `OutputMux` | 2 |
| `MatOrganization` | 4×4 |
| Area | 2.603 mm² |
| Average latency | 7.236 ns |
| Average dynamic energy | 182.436 pJ |
| `Score_LE` | 0.949 |

Compared with the nominal reference, this design improves average latency while staying well below the area budget, with only a moderate energy increase.

## Pareto Boundary

The Pareto-efficient designs show the latency-energy boundary under the hard area constraint:

| Role | CellArea | OutMux | Mat | Area | L_avg | E_avg |
|---|---:|---:|---:|---:|---:|---:|
| Min latency | 0.2× | 4 | 8×8 | 1.314 mm² | 6.640 ns | 290.451 pJ |
| Pareto point | 0.4× | 2 | 1×1 | 2.687 mm² | 7.229 ns | 205.831 pJ |
| Best balanced | 0.5× | 2 | 4×4 | 2.603 mm² | 7.236 ns | 182.436 pJ |
| Pareto point | 1.0× | 1 | 8×8 | 4.137 mm² | 7.726 ns | 173.634 pJ |
| Pareto point | 0.8× | 1 | 2×2 | 3.693 mm² | 7.796 ns | 172.244 pJ |
| Min energy | 1.0× | 1 | 4×4 | 4.155 mm² | 8.022 ns | 172.020 pJ |

The lowest-latency design uses smaller cells and more aggressive valid organization settings, but its energy cost is much higher. The lowest-energy design stays closer to nominal cell area and uses lower output muxing. The selected balanced point lies between these extremes.

## Key Takeaways

- `CellArea` mainly controls the area-feasibility envelope, but smaller cells do not automatically produce the best design.
- `OutputMux` introduces strong validity and energy effects.
- `MatOrganization` shifts where the best feasible latency-energy region appears.
- The best design emerges from device-architecture interaction rather than from a single independent knob.

## Report

The full project report is available here:

[`sttram-area-constrained-dse-report.pdf`](./sttram-area-constrained-dse-report.pdf)

## Tool

- [NVSim](https://github.com/SEAL-UCSB/NVSim)
