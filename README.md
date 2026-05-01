# STT-RAM Design Space Exploration

Area-constrained design-space exploration of a **16 MB STT-RAM memory** using **NVSim**.

This project studies how **device-level assumptions** and **memory-organization choices** interact under a hard area budget. The goal is not only to find a fast or low-energy design, but to understand the feasible design region and identify a balanced latency-energy point.

## Project Story

STT-RAM is attractive for large on-chip memories because of its high density and negligible leakage. However, memory quality is not determined by the cell alone. A smaller memory cell may improve density, but it can also change latency and dynamic energy. Similarly, architectural choices such as output muxing and mat partitioning affect peripheral overhead, internal access paths, and simulator validity.

This project treats STT-RAM memory design as a **device-architecture co-exploration problem**. A nominal reference is first defined, then candidate device and organization knobs are screened, and finally the most meaningful parameters are swept under a hard area constraint.

## Methodology

The exploration is organized as a staged workflow:

```text
[1] Nominal Reference Setup
    Define a controlled 16 MB STT-RAM reference point in NVSim.

[2] Parameter Screening
    Rank device-level and organization-level candidates separately.

[3] Parameter Selection
    Select CellArea, OutputMux, and MatOrganization for detailed exploration.

[4] Constrained Sweep
    Sweep the selected parameters under a hard area budget.

[5] Feasibility Filtering
    Separate valid, over-budget, invalid, and incomplete NVSim outputs.

[6] Tradeoff Analysis
    Analyze latency-energy Pareto points and select a balanced design.
```

The nominal reference uses a 16 MB STT-RAM memory macro in NVSim with RAM mode, 128-bit data width, 22 nm HP roadmap, and ReadEDP as the reference optimization target.

The hard area budget is set to:

$$
A_{\mathrm{budget}} = 1.10A_0 = 4.8488~\mathrm{mm}^2
$$

where $A_0$ is the nominal reference area.

## Parameter Screening

Before the final sweep, candidate parameters are screened so the final design space is not chosen arbitrarily.

### Device-Level Sensitivity

Device-level screening evaluates grouped STT-RAM cell parameters such as `CellArea`, `WriteCurrent`, `WritePulse`, `ResistanceLevel`, and `AccessWidth`.

For continuous device-side parameters, the project uses average relative sensitivity:

$$
S_{m,g} = \frac{1}{|\mathcal{S}|} \sum_{s\in\mathcal{S}} \left| \frac{m(s)-m_0}{m_0} \middle/ (s-1) \right|
$$

This measures how strongly metric $m$ responds to a relative perturbation of device parameter group $g$. In simple terms, it answers:

> If this device parameter changes by a certain percentage, how much do area, latency, or energy change in response?

The screening result identifies `CellArea` as the leading device-side variable because it strongly affects area feasibility while also producing measurable latency and energy changes.

### Organization-Level Impact

Organization-level screening evaluates discrete architecture parameters such as `OutputMux`, `MatOrganization`, `MuxSenseAmp`, and `Routing`.

For discrete organization choices, a derivative-like percentage sensitivity is less meaningful, so the project uses average relative impact:

$$
I_{m,p} = \frac{1}{N} \sum_i \left| \frac{m_i-m_0}{m_0} \right|
$$

This measures how much a valid alternative organization moves a metric away from the nominal reference. In simple terms, it answers:

> Across valid architectural alternatives, how much does this organization knob change area, latency, or energy?

The screening result identifies `OutputMux` as the strongest organization-side variable, mainly through dynamic-energy impact. `MatOrganization` is retained as a secondary variable because it provides a clean array-partitioning interpretation.

### Aggregate Screening Score

For both screening stages, area, latency, and energy responses are combined into one aggregate ranking score:

$$
\mathrm{Score}(g)=0.4M_{A,g}+0.3M_{L,g}+0.3M_{E,g}
$$

Here, $M_{A,g}$, $M_{L,g}$, and $M_{E,g}$ represent the area, latency, and energy response terms for parameter group $g$. For device-level parameters, these terms come from sensitivity values $S$; for organization-level parameters, they come from impact values $I$.

The area term is given a slightly higher weight because this project is explicitly area-constrained. The score is used as a practical screening heuristic, not as a universal objective function. It helps identify which parameters deserve detailed exploration before the final constrained sweep.

## Final Sweep

The final sweep focuses on three selected parameters:

- `CellArea`
- `OutputMux`
- `MatOrganization`

<div align="center">

| Parameter | Sweep Range |
|:---:|:---:|
| `CellArea` Scale | 0.2 to 1.3, Step 0.1 |
| `OutputMux` | 1, 2, 4, 8, 16 |
| `MatOrganization` | 1×1, 2×2, 4×4, 8×8, 16×16 |

</div>

After running NVSim and parsing the outputs:

<div align="center">

| Category | Count |
|:---:|:---:|
| Attempted Design Points | 300 |
| Complete NVSim Outputs | 101 |
| Area-Feasible Designs | 68 |
| Pareto-Efficient Designs | 6 |

</div>

Invalid or incomplete points are treated as outside the usable NVSim design space under the forced organization constraints.

## Latency-Energy Analysis

Each valid design is compared using average latency and average dynamic energy:

$$
L_{\mathrm{avg}} = \frac{L_r + L_w}{2}
$$

$$
E_{\mathrm{avg}} = \frac{E_r + E_w}{2}
$$

To select one representative balanced point from the feasible tradeoff region, the project uses an equal-weight normalized latency-energy score:

$$
\mathrm{Score}_{\mathrm{LE}} = 0.5\frac{L_{\mathrm{avg}}}{L_{\mathrm{avg},0}} + 0.5\frac{E_{\mathrm{avg}}}{E_{\mathrm{avg},0}}
$$

Pareto-efficient designs define the latency-energy boundary. The score is used only to choose one representative balanced point from that feasible tradeoff region.

## Selected Balanced Design

The best balanced point is:

<div align="center">

| Item | Value |
|:---:|:---:|
| `CellArea` Scale | 0.5× |
| `OutputMux` | 2 |
| `MatOrganization` | 4×4 |
| Area | 2.603 mm² |
| Average Latency | 7.236 ns |
| Average Dynamic Energy | 182.436 pJ |
| `Score_LE` | 0.949 |

</div>

Compared with the nominal reference, this design improves average latency while staying well below the area budget, with only a moderate energy increase.

## Pareto Boundary

The Pareto-efficient designs show the latency-energy boundary under the hard area constraint.

<div align="center">

| Role | CellArea | OutMux | Mat | Area | L_avg | E_avg |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Min Latency | 0.2× | 4 | 8×8 | 1.314 mm² | 6.640 ns | 290.451 pJ |
| Pareto Point | 0.4× | 2 | 1×1 | 2.687 mm² | 7.229 ns | 205.831 pJ |
| Best Balanced | 0.5× | 2 | 4×4 | 2.603 mm² | 7.236 ns | 182.436 pJ |
| Pareto Point | 1.0× | 1 | 8×8 | 4.137 mm² | 7.726 ns | 173.634 pJ |
| Pareto Point | 0.8× | 1 | 2×2 | 3.693 mm² | 7.796 ns | 172.244 pJ |
| Min Energy | 1.0× | 1 | 4×4 | 4.155 mm² | 8.022 ns | 172.020 pJ |

</div>

The lowest-latency design uses smaller cells and more aggressive valid organization settings, but its energy cost is much higher. The lowest-energy design stays closer to nominal cell area and uses lower output muxing. The selected balanced point lies between these extremes.

## Key Takeaways

- `CellArea` mainly controls the area-feasibility envelope, but smaller cells do not automatically produce the best design.
- `OutputMux` introduces strong validity and energy effects.
- `MatOrganization` shifts where the best feasible latency-energy region appears.
- The best design emerges from device-architecture interaction rather than from a single independent knob.

## Report

The full project report is available here:

[`STT_RAM_Area_Constrained_Design_Space_Exploration.pdf`](./STT_RAM_Area_Constrained_Design_Space_Exploration.pdf)

## Tool

- [NVSim](https://github.com/SEAL-UCSB/NVSim)
