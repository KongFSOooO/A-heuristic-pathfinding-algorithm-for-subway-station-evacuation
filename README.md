# A-heuristic-pathfinding-algorithm-for-subway-station-evacuation
This repository illustrates the organization of data for a subway-station evacuation study.
The study develops a time-based A* routing method that uses pedestrian density and environmental information to select evacuation routes. A network model generates routes, and an AnyLogic pedestrian simulation evaluates the resulting evacuation durations. This README explains how to inspect the supplied datasets, carry out the corresponding analyses, and identify the inputs needed to rerun the full workflow.

1. Repository contents
| File | Contents |
| --- | --- |
| `network/nodes.csv` | IDs, floors, and types of 236 example nodes. |
| `network/edges.csv` | Connections and lengths of 275 undirected edges. |
| `scenario1_blockage/route_comparison.csv` | Baseline and proposed routes, hop counts, and change flags across 60 observations. |
| `scenario1_blockage/route_changes.csv` | Number of changed-route origins and their mean time reduction at each observation. |
| `scenario1_blockage/evacuation_times.csv` | Baseline and proposed mean evacuation durations for each evaluated origin. |
| `scenario1_blockage/density_records.csv` | Node-level pedestrian densities across 60 observations. |
| `scenario2_fire/route_comparison.csv` | Baseline and proposed routes, hop counts, and change flags across 104 observations. |
| `scenario2_fire/evacuation_times.csv` | Baseline and proposed mean evacuation durations for each evaluated origin. |
| `scenario2_fire/density_records.csv` | Node-level pedestrian densities across 104 observations. |
| `scenario3_sensitivity/sensitivity_results.csv` | Mean differences and standard deviations for 13 temperature or water-depth settings. |
| `scenario3_sensitivity/origin_results.csv` | Origin-level baseline and proposed durations underlying the sensitivity summaries. |

2. Study configuration
The case study represents Guryong Station using 236 nodes and 275 edges, with circulation levels B6, B5, B3, B2, and B1. Ordinary circulation areas are 5 m × 5 m, corresponding to 25 m² per area. The nearer exit is the single routing target in the modeled configuration.
The simulation contains 1,600 pedestrians: 1,400 distributed across non-exit node areas and an additional 200 on B6 to represent alighting passengers. Comfortable walking speeds are drawn uniformly from 1.2–1.5 m/s. Escalators are assumed stopped and treated as stairs. For network cost calculation, stairs and stopped escalators inherit the effective speeds of their associated level-ground approach nodes.
The manuscript reports Python 3.12 for network calculations and AnyLogic Personal Learning Edition 8.9 for pedestrian simulation. These are the reported study versions; the data-loading examples below are utilities for inspecting the files.
Source: manuscript Sections 3.2 and 4.1–4.2.

3. Open and inspect the data
The workbooks can be opened in a spreadsheet application, and the CSV files can be imported as tables. For Python inspection, install:
python -m pip install pandas openpyxl
from pathlib import Path
import pandas as pd

data_dir = Path(".")

for filename in ("D1_1.xlsx", "D1_2.xlsx"):
    with pd.ExcelFile(data_dir / filename, engine="openpyxl") as book:
        print(f"\n{filename}: {book.sheet_names}")
        for sheet in book.sheet_names:
            preview = pd.read_excel(
                book, sheet_name=sheet, header=None, nrows=6
            )
            print(f"\nWorksheet: {sheet}")
            print(preview.to_string(index=False, header=False))

for filename in (
    "Disaster1_comparison_results.csv",
    "Disaster2_comparison_results.csv",
):
    preview = pd.read_csv(data_dir / filename, header=None, nrows=6)
    print(f"\n{filename}")
    print(preview.to_string(index=False, header=False))
    The CSV example assumes comma-separated UTF-8 text. Adjust the delimiter or encoding if the file uses another format. After inspecting the layout, load the required sheet or table with the appropriate header row.

4. Analyze pedestrian density
For an ordinary 25 m² circulation area, calculate density from occupancy as:
\[
D_{i,k}=\frac{N_{i,k}}{25},
\]
where $N_{i,k}$ is the number of pedestrians in area $i$ at observation $k$, and $D_{i,k}$ is measured in persons/m². If a field already contains density, use it directly.
To reproduce the analysis underlying Figure 9, select the structural-blockage records, align observations by time, and plot density histories for the monitored nodes. Compare peak density, the duration of congestion, and the locations of crowded approach areas. Use the actual timestamps; a sample index alone does not establish the first observation time.
For Scenario 2, the route comparison uses the node-level density snapshot at 530 s, when the baseline B3 floor-average density reaches its recorded maximum. Use each node's contemporaneous density in the route calculation.
Source: manuscript Sections 4.3.1, 4.4.1, and 4.4.2.

5. Compare routes and evacuation outcomes
5.1. Identify origins with different routes
At each observation, compare the complete ordered node sequences returned by the proposed time-based A* method and the distance-based Dijkstra implementation for the same origin and target exit.
Use the same evaluated accessible origin set for both methods, and record origins without an available route separately.
Let $C_k$ contain the origins with different sequences, and let $M_k=|C_k|$. Plotting $M_k$ over the structural-blockage observations gives the analysis underlying Figure 10(a). At the 530 s fire snapshot, the same comparison supports Figure 11.
A route's hop count is its number of transitions, normally len(route) - 1. Compare proposed and Dijkstra hop counts to obtain the distribution underlying Figure 12(a). Equal hop counts can occur for different routes.
5.2. Calculate origin-node evacuation times
Simulation evacuation time is the elapsed time from the beginning of evacuation movement to crossing the exit line. For each strategy, first average pedestrian durations within their initial origin node. Denote the resulting mean by $\overline{T}{i,B}$ for the default AnyLogic shortest-path baseline and $\overline{T}{i,H}$ for the proposed-guidance simulation.
Subsequent comparisons give equal weight to each evaluated origin mean. If the data already contain origin-node means, use those means directly. Floor summaries group origins by their initial floor and represent their complete journeys to the exit.
The Dijkstra network route comparator and the AnyLogic simulation baseline serve different parts of the evaluation. Network traversal-cost estimates and simulated departure-to-exit durations must be kept as separate quantities.
5.3. Use explicit signs for time comparisons
Quantity	Calculation	Interpretation
Evacuation-time reduction	Baseline mean − proposed mean	Positive values favor the proposed guidance.
Evacuation-time difference	Proposed mean − baseline mean	Negative values favor the proposed guidance.


Check the definition of any precomputed CSV difference column before interpreting it.
For Figure 10(b), average the reductions only over the changed-route origins at each observation:
\[
SC_k=\frac{1}{|C_k|}\sum_{i\in C_k}
\left(\overline{T}_{i,B}-\overline{T}_{i,H}\right),
\qquad |C_k|>0.
\]
The origin means come from completed simulation runs. Variation across observations reflects changes in the membership of $C_k$; 
For Figure 12(b), summarize the distribution of origin-level reductions. State the included origin set and its size when calculating percentages, including the percentage with reductions exceeding 10 s.
Source: manuscript Sections 4.3.2, 4.4.1, and 4.4.2; Equations (4)–(6).
Optional Python utility for paired origin means
Use this function after selecting one scenario, environmental setting, and analysis subset. The input must contain one paired record per origin, with simulated mean evacuation times in seconds. Supply the actual column names from the file.
import pandas as pd

def compare_origin_means(table, origin_col, baseline_col, proposed_col):
    paired = table[[origin_col, baseline_col, proposed_col]].copy()
    paired.columns = ["origin", "baseline_s", "proposed_s"]

    if paired.empty or paired.isna().any().any():
        raise ValueError("Select nonempty records with complete paired means.")
    if paired["origin"].duplicated().any():
        raise ValueError("Filter or aggregate to one paired mean per origin.")

    for column in ("baseline_s", "proposed_s"):
        paired[column] = pd.to_numeric(paired[column], errors="raise")
        if not paired[column].between(0, float("inf"), inclusive="left").all():
            raise ValueError("Evacuation times must be finite and nonnegative.")

    paired["reduction_s"] = paired["baseline_s"] - paired["proposed_s"]
    paired["difference_s"] = -paired["reduction_s"]
    summary = {
        "n_origins": len(paired),
        "equal_origin_mean_reduction_s": paired["reduction_s"].mean(),
    }
    return paired, summary
For Figure 10(b), select the origins in $C_k$ before calling the function at each observation. The function does not infer route changes, determine the file's scenario, or convert network travel costs into simulation times.

6. Relate the available records to the paper's outputs
Reconstruct an output only when its required records are present.
Paper output	Required records
Figure 9: Density histories	Structural-blockage node densities or counts, area definitions, and observation times.
Figure 10(a): Changed-route counts	Proposed and Dijkstra route sequences, or documented change indicators, for each evaluated origin and observation.
Figure 10(b): Mean time reductions	Completed-run simulation origin means for both strategies and the changed-route sets for each observation.
Figure 11: Fire-scenario route differences	Routes at 530 s; node coordinates, floor labels, and connectivity for a spatial visualization.
Figure 12(a): Hop-count differences	Ordered routes or documented hop counts for both network methods.
Figure 12(b): Distribution of time reductions	Paired simulated origin means and an explicitly defined origin subset.
Figure 13: Environmental sensitivity	Paired simulated origin means for each separate temperature or water-depth setting. 


7. Rerun the full routing and simulation workflow
The manuscript describes the following workflow:
1. Obtain reference records. Run the default shortest-path AnyLogic simulation to obtain baseline density observations and evacuation durations under the chosen scenario.
2. Prepare each routing snapshot. Remove unavailable nodes and connections, identify the component connected to the target exit, and match density and environmental inputs to network nodes.
3. Estimate local speeds. Retain the lower of the illuminance-based and density-based candidate speeds, then apply the applicable temperature or water-depth adjustment. Assign each stair or stopped-escalator node its parent-node effective speed. Use Table 1 and Section 3.1 for the adopted rules.
4. Calculate transition costs and the heuristic. For non-exit neighbors, use the center-to-center distance divided by the lower adjacent-node speed. For the final connection to the exit, use the adjacent non-exit node's speed. The heuristic is the shortest available network distance to the exit divided by the maximum effective speed among retained non-exit nodes.
5. Generate and apply routes. Run Algorithms 1–2, map node sequences to AnyLogic movement areas, and apply the routes at the corresponding density-sampling times. The manuscript describes reference density records as the source of the observations used for this workflow.
6. Record and compare evacuation durations. Obtain completed-run departure-to-exit durations, aggregate by initial origin, and apply the comparison definitions in Section 5 of this README.
During each A* search, speeds are fixed and assumed finite and positive. Exact reproduction requires the original speed-rule implementation, route tie-breaking, and stochastic simulation settings. The mathematical minimum-cost claim applies to the prepared network snapshot and its modeled traversal costs.
In the reported simulation, local calculated speeds are applied while pedestrians occupy traversable areas subject to environmental mobility adjustments, and their original comfortable-speed settings are restored after leaving those areas.
    
