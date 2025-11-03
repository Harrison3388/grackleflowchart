```mermaid
---
config:
  theme: neo-dark
---
flowchart TD
  A["solve_rate_cool_g start"] --> B["initialize units, constants, flags (anydust, min_metallicity, dom, chunit, dx_cgs, c_ljeans)"]
  B --> C{"fields in comoving frame?"}
  C -- "yes" --> C1["scale_fields_g(comoving→proper)"]
  C1 --> D
  C -- "no" --> D["ceiling_species_g()"]
  D --> E["build index helper & begin OpenMP parallel"]
  E --> F["allocate scratch buffers (grain_temperatures, logTlininterp_buf, cool1dmulti_buf, coolingheating_buf, SpeciesRateSolverScratchBuf)"]
  F --> G["loop over i-slices (outer indices j,k)"]
  subgraph SUB["Per i-slice loop"]
    direction TB
    G1["set iteration mask itmask = TRUE"] --> G2["coupled_rt_modify_itmask_() if RT coupled"]
    G2 --> G3["init ttot=0; precompute ddom=d*dom"]
    G3 --> H1
    subgraph SUBCYCLE["Subcycle loop"]
      direction TB
      H1["'reset dtit=huge8 for active cells'"]
      H1 --> H2["cool1d_multi_g(): compute edot, Tgas, Tdust, metallicity"]
      H2 --> H3{"primordial_chemistry > 0 ?"}
      H3 -- "yes" --> H4["lookup_cool_rates1d_g(): interpolate reaction rates"]
      H4 --> H5["rate_timestep_g(): compute dedot, HIdot"]
      H5 --> H6["setup_chem_scheme_masks_(): decide Gauss-Seidel vs Newton-Raphson"]
      H6 --> H7["set_subcycle_dt_from_chemistry_scheme_(): choose dtit per zone"]
      H3 -- "no" --> H8["skip chemistry"]
      H7 --> H9["enforce_max_heatcool_subcycle_dt_(): limit dtit by cooling/heating"]
      H9 --> H10{"with_radiative_cooling == 1 ?"}
      H10 -- "yes" --> H11["update e(i,j,k) += edot/d * dtit"]
      H10 -- "no" --> H12["skip energy update"]
      H11 --> H13{"primordial_chemistry > 0 ?"}
      H13 -- "yes" --> H14["step_rate_g(): Gauss-Seidel solver"]
      H14 --> H15["step_rate_newton_raphson(): Newton-Raphson solver"]
      H13 -- "no" --> H16["skip species solver"]
      H15 --> H17["update ttot += dtit; deactivate finished cells"]
      H17 --> H18{"all cells done?"}
      H18 -- "yes" --> H19["break subcycle loop"]
      H18 -- "no" --> H1
    end
    H19 --> I["if iter > max_iterations: print error, maybe set ierr=GR_FAIL"]
  end
  I -- "outer cycle done" --> J["cleanup temporary buffers"]
  J --> K{"ierr != GR_SUCCESS ?"}
  K -- "yes" --> L["return ierr"]
  K -- "no" --> M{"check fields comoving"}
  M -- "yes" --> N["scale_fields_g(proper→comoving)"]
  N --> O{"primordial_chemistry > 0 ?"}
  O -- "yes" --> P["make_consistent_g(): enforce mass conservation"]
  O -- "no" --> R
  P --> R["return ierr"]

