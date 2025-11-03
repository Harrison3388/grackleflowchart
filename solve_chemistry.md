```mermaid
---
config:
  layout: elk
  theme: neo-dark
---
flowchart TD
    A["solve_chemistry: start"] --> B["call local_solve_chemistry"]
    B --> L1{"use_grackle == true ?"}
    L1 -- no --> L2["local_solve_chemistry returns SUCCESS"]
    L1 -- yes --> L3@{ label: "init photo_rate_storage 'my_uvb_rates' to zeros" }
    L3 --> L4{"UVbackground == 1 ?"}
    L4 -- yes --> L5["update_UVbackground_rates"]
    L5 --> L6{"update ok ?"}
    L6 -- no --> L7@{ label: "'Error in update_UVbackground_rates'" }
    L7 --> L8["local_solve_chemistry returns FAIL"]
    L6 -- yes --> L9["UVB rates ready"]
    L4 -- no --> L10@{ label: "copy rates from 'my_rates' into 'my_uvb_rates'" }
    L9 --> L11{"my_fields->metal_density is NULL ?"}
    L10 --> L11
    L11 -- yes --> L12[metal_field_present = FALSE]
    L11 -- no --> L13["metal_field_present = TRUE'"]
    L12 --> L14@{ label: "new_internalu_ from my_units" }
    L13 --> L14
    L14 --> L15{"self_shielding_err_check ok ?"}
    L15 -- no --> L16["local_solve_chemistry returns FAIL"]
    L15 -- yes --> L17["call solve_rate_cool_g with metal flag, dt, internalu, chemistry, rates, fields, uvb_rates"]
    L17 --> L18{"ierr == GR_FAIL ?"}
    L18 -- yes --> L19@{ label: "'Error in solve_rate_cool_g'" }
    L19 --> L20["local_solve_chemistry returns FAIL"]
    L18 -- no --> L21["local_solve_chemistry returns SUCCESS"]
    L2 --> S1{"did local_solve_chemistry return FAIL ?"}
    L8 --> S1
    L16 --> S1
    L20 --> S1
    L21 --> S1
    S1 -- yes --> S2@{ label: "Error in local_solve_chemistry" }
    S2 --> S3["solve_chemistry returns FAIL"]
    S1 -- no --> S4["solve_chemistry returns SUCCESS"]
    L3@{ shape: rect}
    L7@{ shape: rect}
    L10@{ shape: rect}
    L14@{ shape: rect}
    L19@{ shape: rect}
    S2@{ shape: rect}

