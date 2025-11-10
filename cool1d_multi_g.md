```mermaid
---
config:
  theme: neo-dark
---
flowchart TD
START([Start cool1d_multi_g])
INITV["Init views and buffers"]
CONST["Set constants, flags and units"]
EDOT0["Zero edot for masked cells"]
PRES["Compute pressure p2d"]
PRIM{"primordial_chemistry = 0"}
TEMP0["Compute rhoH then T and mmw via cloudy"]
TEMP1["Compute mmw and rhoH from species then set tgas"]
H2GAMMA["Optional H2 gamma correction"]
TFLOOR{"Temperature floor active and T <= floor"}
TFLOOR_SET["Set edot tiny and mask cell"]
ZMET["Set metallicity array"]
NH["Set n_H = rhoH * dom"]
ITER1{"iter == 1"}
SETOLD["Set tgasold = tgas"]
LOGS["Build logs for T, rho, species, dvdr, lshield, Tcmb"]
INTERP{"interp flag"}
IDX["Compute table indices"]
START --> INITV --> CONST --> EDOT0 --> PRES --> PRIM
PRIM -- yes --> TEMP0 --> TFLOOR
PRIM -- no  --> TEMP1 --> H2GAMMA --> TFLOOR
TFLOOR -- yes --> TFLOOR_SET --> ZMET
TFLOOR -- no  --> ZMET
ZMET --> NH --> ITER1
ITER1 -- yes --> SETOLD --> LOGS
ITER1 -- no  --> LOGS
LOGS --> INTERP --> IDX
PRI_ON{"primordial_chemistry > 0"}
PRI6["Interpolate primordial tables and add L_pri(edot)"]
IDX --> PRI_ON
PRI_ON -- yes --> PRI6 --> H2SW
PRI_ON -- no  --> H2SW
H2SW{"primordial_chemistry > 1"}
H2RATE{"h2_cooling_rate 0 1 2 3"}
H2_LS["H2 Lepp & Shull"]
H2_GP["H2 Galli & Palla"]
H2_GA["H2 Glover & Abel"]
H2_CW["H2 Chiaki & Wise"]
CIE{"cie_cooling 0 1 2"}
CIE1["Apply CIE Ripamonti & Abel 2003"]
CIE2["Apply CIE Yoshida et al. 2006"]
H2SW -- no --> HD_SW
H2SW -- yes --> H2RATE
H2RATE -- 0 --> H2_LS --> CIE
H2RATE -- 1 --> H2_GP --> CIE
H2RATE -- 2 --> H2_GA --> CIE
H2RATE -- 3 --> H2_CW --> CIE
CIE -- 1 --> CIE1 --> HD_SW
CIE -- 2 --> CIE2 --> HD_SW
CIE -- 0 --> HD_SW
HD_SW{"primordial_chemistry > 2"}
HD_RATE{"hd_cooling_rate 0 1"}
HD_CW["HD Chiaki & Wise 2019"]
HD_OLD["HD Coppola et al 2011 and Wrathmall, Gusdorf, & Flower 2007"]
HD_SW -- no --> METMASK
HD_SW -- yes --> HD_RATE
HD_RATE -- 1 --> HD_CW --> METMASK
HD_RATE -- 0 --> HD_OLD --> METMASK
METMASK["Build metal mask by Z threshold"]
GRAIN_STEP{"use_dust_density_field > 0 and dust_species > 0"}
GRAININC["Compute grain size increment"]
D2G["Compute dust to gas ratio"]
ISRF["Compute instellar radiation field"]
METMASK --> GRAIN_STEP
GRAIN_STEP -- yes --> GRAININC --> D2G
GRAIN_STEP -- no  --> D2G --> ISRF
ANYDUST{"If anydust"}
TDUST["Compute tdust gasgr kappa"]
Ldust["Calculate and add dust cooling to edot"]
OPAC["Proceed to opacity"]
ISRF --> ANYDUST
ANYDUST -- yes --> TDUST --> Ldust --> OPAC
ANYDUST -- no  --> OPAC
OPAC_PRIM{"use_primordial_continuum_opacity == 1"}
ALPHA0["alpha from tables"]
ALPHAZ["alpha equals zero"]
ALPHAD["Add contribution from dust opacity (alphad)"]
TAU["tau_con = alpha * lshield"]
OPAC --> OPAC_PRIM
OPAC_PRIM -- yes --> ALPHA0 --> ALPHAD
OPAC_PRIM -- no  --> ALPHAZ --> ALPHAD --> TAU
HEAT_RT{"primordial_chemistry > 0"}
SSM{"self_shielding_method 0 1 2 3"}
UV0["Add photo heating, thin (no shielding)"]
UV1["Add photo heating HI shielded, HeI and HeII thin"]
UV2["Add photo heating HI shielded, approx. shielding HeI and HeII"]
UV3["Add photo heating HI and HeI shielded, ignore HeII"]
TAU --> HEAT_RT
HEAT_RT -- no --> CLOUDY_PRIM
HEAT_RT -- yes --> SSM
SSM -- 0 --> UV0 --> CLOUDY_PRIM
SSM -- 1 --> UV1 --> CLOUDY_PRIM
SSM -- 2 --> UV2 --> CLOUDY_PRIM
SSM -- 3 --> UV3 --> CLOUDY_PRIM
CLOUDY_PRIM{"primordial_chemistry == 0"}
CLOUDY0["Cloudy primordial tables use mmw for n_e"]
PE["Photoelectric heating stage"]
CLOUDY_PRIM -- yes --> CLOUDY0 --> PE
CLOUDY_PRIM -- no  --> PE
PEMODE{"photoelectric_heating 0 1 2 3"}
PE1["Set gamma_eff constant (gammah)"]
PE2["gamma_eff = 0.05* ISRF* gammah"]
PE3["Compute epsilon and set gamma_eff"]
PEADD["Add photoelectric_heating"]
PE --> PEMODE
PEMODE -- 1 --> PE1 --> PEADD --> DUSTREC
PEMODE -- 2 --> PE2 --> PEADD
PEMODE -- 3 --> PE3 --> PEADD
PEMODE -- 0 --> DUSTREC
DUSTREC{"dust_chemistry > 0 or dust_recombination_cooling > 0"}
REGR["Interpolate recombination on dust"]
REGRADD["Add recombination cooling"]
DUSTREC -- yes --> REGR --> REGRADD --> COMPTON
DUSTREC -- no  --> COMPTON
COMPTON["Add Compton and X ray Compton"] --> RTRAD
RTRAD{"use_radiative_transfer == 1"}
RTADD["Add radiative_transfer"]
RTRAD -- yes --> RTADD --> METAL
RTRAD -- no  --> METAL
METAL{"metal_cooling == 1"}
TABGATE["Mask cells below table Tmin"]
TABNEW{"cloudy_data_new == 1"}
CLOUDY_Z["Cloudy metal tables new"]
CLOUDY_Z_OLD["Cloudy metal tables old"]
LOWT{"metal_chemistry == 1"}
LOWT_COOL[" C/O fine-structure, metal molecular rotational cooling for low_temperatures"]
METAL -- no --> USERH
METAL -- yes --> TABGATE --> TABNEW
TABNEW -- yes --> CLOUDY_Z --> LOWT
TABNEW -- no  --> CLOUDY_Z_OLD --> LOWT
LOWT -- yes --> LOWT_COOL --> USERH
LOWT -- no  --> USERH
USERH{"use_volumetric_heating_rate == 1"}
USERM{"use_specific_heating_rate == 1"}
VADD["Add volumetric heating"]
MADD["Add specific heating"]
USERH -- yes --> VADD --> USERM
USERH -- no  --> USERM
USERM -- yes --> MADD --> OPAC_APPLY
USERM -- no  --> OPAC_APPLY
OPAC_APPLY{"tau_con > 1"}
TAU_SCALE{"tau_con < 1e2"}
SCALE["Scale edot by tau_con"]
ZERO["Set edot to zero"]
OPAC_APPLY -- no --> TOLD
OPAC_APPLY -- yes --> TAU_SCALE
TAU_SCALE -- yes --> SCALE --> TOLD
TAU_SCALE -- no  --> ZERO --> TOLD
TOLD["Set tgasold = tgas"]
FREE["Free dust buffers"]
END([Return])
TOLD --> FREE --> END
