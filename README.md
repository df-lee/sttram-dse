# STT-RAM Design Space Exploration

Area-constrained design space exploration of a 16 MB STT-RAM memory using NVSim.

This project studies how device-level assumptions and memory-organization choices jointly affect area feasibility, latency, energy, and balanced design quality.

## Highlights

- Built an NVSim-based flow for area-constrained STT-RAM memory exploration.
- Screened device-level and organization-level parameters using normalized response metrics.
- Performed a constrained sweep over `CellArea`, `OutputMux`, and `MatOrganization`.
- Analyzed validity boundaries, area feasibility, latency-energy tradeoffs, and Pareto-efficient designs.

## Methodology

```text
Nominal STT-RAM reference
        ↓
Sensitivity / impact screening
        ↓
Key parameter selection
        ↓
Constrained design-space sweep
        ↓
Feasibility filtering
        ↓
Pareto and balanced-design analysis
```

## Sweep Space

| Parameter | Values |
|---|---|
| `CellArea` scale | 0.2 to 1.3, step 0.1 |
| `OutputMux` | 1, 2, 4, 8, 16 |
| `MatOrganization` | 1×1, 2×2, 4×4, 8×8, 16×16 |

## Summary

| Category | Count |
|---|---:|
| Attempted design points | 300 |
| Complete NVSim outputs | 101 |
| Area-feasible designs | 68 |
| Pareto-efficient designs | 6 |

## Selected Balanced Design

| Item | Value |
|---|---:|
| `CellArea` scale | 0.5× |
| `OutputMux` | 2 |
| `MatOrganization` | 4×4 |
| Area | 2.603 mm² |
| Average latency | 7.236 ns |
| Average dynamic energy | 182.436 pJ |
| `Score_LE` | 0.949 |

## Key Takeaways

- `CellArea` mainly controls the area-feasibility envelope.
- `OutputMux` introduces strong validity and energy effects.
- `MatOrganization` shifts where the best latency-energy region appears.
- The best design is determined by device-architecture interaction, not by a single independent knob.

## Report

The full report is included in this repository:

```text
sttram-area-constrained-dse-report.pdf
```

## Tool

- [NVSim](https://github.com/SEAL-UCSB/NVSim)

## Author

Zefu Li  
M.S.E. Electrical Engineering  
University of Pennsylvania
