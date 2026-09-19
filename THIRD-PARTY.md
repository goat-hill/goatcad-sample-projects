# Third-Party Material

Everything in this repository is MIT licensed (see `LICENSE`) **except** the
items below. These are redistributed, or referenced, for the convenience of
anyone building this board. They remain the property of their respective
owners and are not covered by the MIT grant.


## Footprints from the KiCad official library

Two footprints in `bitclock/canary.pretty/` are taken from the KiCad official
footprint library, modified only to point at this project's local 3D model
directory:


| File | Upstream library |
|---|---|
| `Sensirion_SCD4x-1EP_10.1x10.1mm_P1.25mm_EP4.8x4.8mm.kicad_mod` | `Sensor.pretty` |
| `SW_Push_1P1T_XKB_TS-1187A.kicad_mod` | `Button_Switch_SMD.pretty` |

Copyright the KiCad Libraries Contributors. Licensed under
[CC-BY-SA-4.0](https://creativecommons.org/licenses/by-sa/4.0/) with the
[KiCad Library Exception](https://www.kicad.org/libraries/license/). Source:
<https://gitlab.com/kicad/libraries/kicad-footprints>.


The exception means a board designed using these footprints is not a derivative
work; the ShareAlike term applies to the footprint files themselves, which is
why they are called out here rather than folded into the MIT grant.


The remaining five footprints in `canary.pretty/` are original work and are MIT.


## 3D models

The STEP files in `bitclock/canary.3dshapes/` are vendor-supplied models,
redistributed unmodified. Each is the file published by the manufacturer or
distributor named below, and no license to redistribute them is claimed or
granted here -- they are included so the board's 3D view is complete.

| Model | Part | Source | Origin |
|---|---|---|---|
| `FH34SRJ-24S-0.5SH_99_.step` | FH34SRJ-24S-0.5SH(99) | manufacturer | <https://www.hirose.com/product/p/CL0580-1255-6-99> |
| `FPC-SMD_ECT818001568.step` | C711417 | distributor | <https://modules.easyeda.com/qAxj6KHrDKw4blvCG8QJPs7Y/9783fbfc731c4de487b4cd24a5374cd0> |
| `Sensirion_SCD4x-1EP_10.1x10.1mm_P1.25mm_EP4.8x4.8mm.step` | SCD40-D-R2 | manufacturer | <https://sensirion.com/media/documents/260AFF2D/616531AE/Sensirion_CO2_Sensors_SCD4x_STEP_file.step> |
| `Sensirion_SGP40_DFN_2.44x2.44mm.step` | SGP41-D-R4 | manufacturer | <https://sensirion.com/media/documents/1B38BCCF/61644FE8/Sensirion_Gas_Sensors_Software_GAS_CD_SGP40_STEP_File.step> |
| `SW_Push_1P1T_XKB_TS-1187A.step` | TS-1187A-B-A-B | manufacturer | <https://www.helloxkb.com/public/images/zip/TS-1187A-B-A-B.zip> |
| `TYPE-C-TH_16PLC-H10.0.step` | C3151747 | distributor | <https://modules.easyeda.com/qAxj6KHrDKw4blvCG8QJPs7Y/38b3c9d170164529b6980414f1c5dbe9> |
| `USB-C-SMD_TYPE-C16PIN.step` | C393939 | distributor | <https://modules.easyeda.com/qAxj6KHrDKw4blvCG8QJPs7Y/99e30ad731ee487a8d60b7518cb54538> |

`canary.3dshapes/index.json` records the size and SHA-256 of every model as
shipped, so any file can be checked against what is documented here.
