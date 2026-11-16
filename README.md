# Open Cascade Carnot Battery Dataset and Surrogate Model

This directory contains Aspen Plus simulation data and a trained PyTorch surrogate model for an open cascade Carnot battery. The model predicts operating-point validity, power-to-power round-trip efficiency, and volumetric energy storage density.

## Files

| File | Contents |
| --- | --- |
| [`dataset.csv`](dataset.csv) | 7,883 simulated operating points sampled using Latin hypercube sampling, with 36 columns covering inputs, simulation outputs, derived performance metrics, and labels. |
| [`model.pt`](model.pt) | PyTorch checkpoint containing the trained weights, model configuration, feature and target order, standardization parameters, and training history. |
| [`pareto_front.csv`](pareto_front.csv) | 100 operating points on the approximate Pareto front from surrogate-assisted multi-objective optimization. |

## Dataset

Each row represents one attempted steady-state simulation. The CSV retains unsuccessful runs and operating points that fail the feasibility criterion. Some of these rows still contain numeric outputs, so use `reg_label` to identify regression-eligible rows and `label` for feasibility classification. Empty fields represent missing values.

| Simulation outcome | `reg_label` | `label` | Rows |
| --- | ---: | ---: | ---: |
| Converged and meets the feasibility criterion | 1 | 1 | 992 |
| Converged but does not meet the feasibility criterion | 1 | 0 | 1,106 |
| Did not converge normally | 0 | 0 | 5,785 |
| **Total** | | | **7,883** |

`state = 8` identifies the 2,098 normally converged simulations.

### Model inputs

The rows below follow the checkpoint's `input_names` order. `vfrac`, `flow1`, and `pres2` are the three decision variables in the study's optimization. The saved checkpoint expects all seven inputs.

| Column | Meaning | Unit |
| --- | --- | --- |
| `vfrac` | Primary-cycle inlet vapor fraction | Fraction |
| `flow1` | Primary-cycle working-fluid mass flow rate | kg/h |
| `pres2` | Secondary compressor outlet pressure | MPa |
| `TH'` | Temperature setpoint at block P-3 | °C |
| `heatst` | Thermal storage efficiency | Fraction |
| `comp_eff` | Shared isentropic efficiency of the two compressors | Fraction |
| `exp_eff` | Shared isentropic efficiency of the two expanders | Fraction |

### model outputs

| Column(s) | Meaning |
| --- | --- |
| `eff` | Power-to-power round-trip efficiency, stored as a fraction. |
| `rho_st` | Volumetric energy storage density in kWh/m³. |
| `label`, `is_valid`, `reg_label` | Binary feasibility target, corresponding Boolean flag, and regression eligibility flag. |
| `COMP1_EFF`, `COMP2_EFF`, `EXP1_EFF`, `EXP2_EFF` | Equipment-level isentropic efficiencies recorded in the simulation. Each pair equals its corresponding shared model input. |
| `W_IN1`, `W_IN2`, `W_OUT1`, `W_OUT2` | Compressor input and expander output power by stage, using the Aspen Plus sign convention. Expander output is recorded with a negative sign. |
| `W_IN`, `W_OUT` | Total compressor input and total expander output power, using the same sign convention. |
| `Vflow_gas`, `Vflow_liquid`, `Volumefrac` | Gas and liquid volumetric flow rates at the secondary compressor outlet and the liquid fraction derived from them. |
| `Q_st` | Calculated heat duty at block HEAT-H1. |
| `rho_NH3_L`, `rho_NH3_H`, `rho_H2O_L`, `rho_H2O_H` | Ammonia and water mass densities at the low- and high-temperature storage states, in kg/m³. |
| `v1`, `v3` | Ammonia storage volume and phase-change material volume, respectively, in m³. |
| `row_index`, `state`, `elapsed_s`, `error` | Row identifier, Aspen Plus run status code, elapsed simulation time in seconds, and any recorded diagnostic message. |

## Pareto front

[`pareto_front.csv`](pareto_front.csv) contains the operating points returned by multi-objective particle swarm optimization (MOPSO) using the surrogate model. Each row reports the three decision variables (`vfrac`, `flow1`, and `pres2`) and the corresponding surrogate predictions for round-trip efficiency (`eff`, a fraction) and volumetric energy storage density (`rho_st`, in kWh/m³).

To plot the trade-off, use `rho_st` on the horizontal axis and multiply `eff` by 100 for the vertical axis in percent. Across the published points, round-trip efficiency ranges from 71.53% to 73.15%, and energy storage density ranges from 14.02 to 16.36 kWh/m³.

## License

All files in this repository are provided solely for consulting the data supporting the accompanying paper and reproducing its reported results. See [LICENSE](LICENSE) for the full terms.

