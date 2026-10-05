# A-heuristic-pathfinding-algorithm-for-subway-station-evacuation-
the database for paper A heuristic pathfinding algorithm for subway station evacua-tion considering pedestrian density and environmental infor-mation
The study develops a time-based A* routing method that uses pedestrian density and environmental information to select evacuation routes. A network model generates routes, and an AnyLogic pedestrian simulation evaluates the resulting evacuation durations. This README explains how to inspect the supplied datasets, carry out the corresponding analyses, and identify the inputs needed to rerun the full workflow.
Documentation status: This README was prepared from the manuscript and the four filenames. The actual spreadsheets and CSV contents have not been inspected. File-to-scenario assignments, worksheet meanings, column names, units, and stored difference signs therefore require confirmation. The examples below inspect the files or accept user-specified column names; they do not assume an unverified data schema.
1. Repository contents
Place this README alongside the following files, or adjust the paths in the examples to match their locations.
Use the scenario definitions below to establish each file's role. The manuscript does not establish whether the two workbooks represent different strategies, observations, inputs, or outputs.
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
Before combining records, identify the following information wherever it is available:
| Information | Purpose |
| --- | --- |
| Scenario and routing strategy | Keep the prescribed conditions and comparison methods aligned. |
| Observation time or sample index | Associate density snapshots with their route calculations. |
| Node or initial-origin identifier | Match density, routes, and evacuation outcomes to the correct locations. |
| Initial floor | Group complete evacuation journeys by starting floor. |
| Pedestrian count or density | Determine whether conversion to persons/m² is needed. |
| Ordered route sequence | Identify route changes and count transitions. |
| Evacuation duration or origin-node mean duration | Calculate the simulation-based performance measures. |
4. Analyze pedestrian density
For an ordinary 25 m² circulation area, calculate density from occupancy as:
\[
D_{i,k}=\frac{N_{i,k}}{25},
\]
where $N_{i,k}$ is the number of pedestrians in area $i$ at observation $k$, and $D_{i,k}$ is measured in persons/m². If a field already contains density, use it directly.
To reproduce the analysis underlying Figure 9, select the structural-blockage records, align observations by time, and plot density histories for the monitored nodes. Compare peak density, the duration of congestion, and the locations of crowded approach areas. Use the actual timestamps; a sample index alone does not establish the first observation time.
For Scenario 2, the route comparison uses the node-level density snapshot at 530 s, when the baseline B3 floor-average density reaches its recorded maximum. Use each node's contemporaneous density in the route calculation.
Source: manuscript Sections 4.3.1, 4.4.1, and 4.4.2.


    
