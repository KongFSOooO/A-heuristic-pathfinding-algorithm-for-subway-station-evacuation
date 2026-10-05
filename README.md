## Study configuration

Guryong Station, Seoul, modelled as an undirected graph of 236 nodes and 275 edges across levels B6, B5, B3, B2 and B1. Ordinary circulation areas are 5 m × 5 m (25 m²); stairs, escalators and exits are represented as facility nodes. The nearer exit area is the single routing target.

The simulation contains 1,600 pedestrians: 1,400 distributed across non-exit node areas and 200 on B6 to represent platform and train-boarding locations. Comfortable walking speeds are drawn from U(1.2, 1.5) m/s. Escalators are assumed stopped and treated as stairs; for network cost calculation, stair and stopped-escalator nodes inherit the effective speed of their adjacent approach node.

Network calculations were implemented in Python 3.12 and pedestrian simulations in AnyLogic Personal Learning Edition 8.9.

## Files

### `network/`

| File | Columns | Contents |
|---|---|---|
| `nodes.csv` | node_id, floor, node_type | 236 nodes. floor is B1–B3, B5, B6; node_type is area, stair, escalator or exit |
| `edges.csv` | from_node, to_node, length_m | 275 undirected edges with centre-to-centre distance |

### `scenario1_blockage/`

Stair blockage on B2 and B1 (nodes 73, 76, 126, 129, 134). 60 observations at 30 s intervals; 229 evaluated origins.

| File | Columns |
|---|---|
| `route_comparison.csv` | time_step, origin_node, baseline_route, proposed_route, baseline_hops, proposed_hops, route_changed |
| `route_changes.csv` | time_step, changed_count, mean_reduction_s |
| `evacuation_times.csv` | origin_node, baseline_s, proposed_s |
| `density_records.csv` | time_step, node_id, density |

### `scenario2_fire/`

Localized fire on B3 (nodes 39, 40, 41, 43, 44, 45, 46, 61). 104 observations at 10 s intervals from 0 s; 223 evaluated origins. The route comparison reported in the manuscript uses the 530 s record, when the B3 floor-average density reaches its maximum.

| File | Columns |
|---|---|
| `route_comparison.csv` | time_step, origin_node, baseline_route, proposed_route, baseline_hops, proposed_hops, route_changed |
| `evacuation_times.csv` | origin_node, baseline_s, proposed_s |
| `density_records.csv` | time_step, node_id, density |

### `scenario3_sensitivity/`

Temperature from −20 to 30 °C in 10 °C steps, and water depth from 0 to 60 cm in 10 cm steps, applied separately.

| File | Columns |
|---|---|
| `sensitivity_results.csv` | variable, setting, mean_diff_s, std_s |
| `origin_results.csv` | variable, setting, origin_node, baseline_s, proposed_s |

## Conventions

- All files are comma-separated UTF-8. Times are in seconds, densities in persons/m², distances in metres.
- Routes are semicolon-separated node sequences from the origin to the exit; hop count is the number of transitions.
- `route_changed` is 1 when the two methods return different node sequences at that observation.
- `mean_reduction_s` is baseline minus proposed, averaged over the changed-route origins at that observation. Positive values indicate shorter evacuation under the proposed guidance.
- `mean_diff_s` is proposed minus baseline. Negative values indicate shorter evacuation under the proposed guidance.
- `std_s` in `sensitivity_results.csv` is the standard deviation across evaluated origins, not across simulation repetitions.
- nodes count changed in different disaster scenarioes where they become naccessible.

## License

CC BY 4.0
