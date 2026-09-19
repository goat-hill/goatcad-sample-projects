---
goal: Bitclock — an e-ink desk companion that measures indoor air quality (CO2, temperature, humidity, VOC/NOx) and shows the time, weather and user-written clockface apps
stage: production
scale: { quantity: 100, anticipated: null }
budget: { bom_target_usd: null }
assembly:
  method: pcba
  house: JLCPCB
  min_passive: "0402"
  fine_pitch_ok: true
  bga_ok: true
sourcing:
  packaging: tape-reel
  distributors: [digikey, mouser]
  second_source: preferred
  lifecycle: active-only
  origin: dont-care
  rohs: required
environment:
  temp_range_c: [0, 40]
  supply: USB-C 5V in, TLV76733DRVT LDO down to a single +3V3 rail; no battery
datasheets:
  path: datasheets/
  committed: true
  extra_paths: []
models:
  path: canary.3dshapes/
  committed: true
footprints:
  density: nominal
  pin1_marker: chevron
committed_parts:
  - mpn: ESP32-S3-WROOM-1-N8
    reason: The product is the radio and the apps — Wi-Fi 2.4GHz plus BLE 5 and 8MB flash for user-written clockfaces. A module, not a bare chip, so the RF section is pre-certified and the board needs no antenna tuning.
  - mpn: SCD40-D-R2
    reason: True photoacoustic NDIR CO2, not an eCO2 estimate derived from VOC. The CO2 number is the headline feature; an estimate would not be honest.
  - mpn: SGP41-D-R4
    reason: VOC and NOx index on one part, which is what the UI shows. Pairs with the SHT40 for the humidity compensation its index needs.
  - mpn: SHT40-AD1B-R2
    reason: Temperature and humidity displayed directly and used to compensate the SGP41. AD1B is the 0x44 address variant, which keeps the I2C bus clear of the SCD40 at 0x62.
  - mpn: TLV76733DRVT
    reason: 5V to 3V3 at 1A in a 2x2mm WSON — enough headroom for the ESP32-S3 TX bursts and the e-ink panel's boost rail without a switcher's noise near the sensors.
assumed:
  - scale.quantity            # you said "hundreds"; 100 is the price break I quote at
  - budget.bom_target_usd     # left open rather than invented
  - assembly.min_passive      # read off the board: passives are 0402 and 0603
  - assembly.fine_pitch_ok    # read off the board: WSON-6 P0.65mm, SHT40 DFN 1.5x1.5mm, 24-pin 0.5mm FPC
  - assembly.bga_ok           # nothing on the board proves it; JLCPCB at this quantity can
  - sourcing.packaging        # follows from hundreds per build
  - sourcing.second_source
  - sourcing.lifecycle
  - sourcing.origin
  - sourcing.rohs
  - environment.temp_range_c  # an indoor desk gadget
  - datasheets.path
  - datasheets.committed
  - models.committed          # canary.3dshapes/ exists but is untracked in git today
  - footprints.density        # nominal because the board is assembled, not hand-soldered
  - footprints.pin1_marker
open_questions:
  - 'Every MPN field in the BOM is empty. IC part numbers currently live in Value (TLV76733DRVT, SCD40-D-R2, SGP41-D-R4, SHT40-AD1B-R2, ESP32-S3-WROOM-1-N8) and the LCSC Part / JLCPCB Part # fields carry the assembly-house numbers. Judgement: MPN is the field of record and should be filled from the Value strings; LCSC Part stays beside it as the JLCPCB cross-reference. Not done yet.'
  - 'No datasheet is cached locally. The Datasheet fields hold vendor URLs (LCSC, Espressif, TI) - exactly the link rot a committed datasheets/ folder exists to prevent. Cache all five ICs before the next order.'
  - 'Six footprints do not resolve on this machine. fp-lib-table registers only canary.pretty, but parts point at Babldev (J1, U3), EasyEDA2Kicad (J2, J3, J4) and PCM_Espressif (U2). Judgement: copy them into canary.pretty so the project is self-contained, rather than expecting a global library on every machine.'
  - 'Three footprints have no 3D model file: SW1-SW2 (SW_Push_1P1T_XKB_TS-1187A), U4 (SCD4x) and U5 (SHT4x). Open, not blocking.'
  - 'canary.3dshapes/ and canary.pretty/ are both untracked in git. They are design inputs and should be committed.'
  - 'R2 is written 1k and R3 is 470R, against the µ/Ω symbol style the rest of the sheet uses. Normalize them to 1kΩ and 470Ω.'
  - 'Sourcing catalog of record. DigiKey and Mouser are what is connected and what has parametric data and fetchable datasheets, so parts are selected there. JLCPCB assembles from LCSC stock, so a DigiKey price at 100 is a sanity check, not the build cost.'
---

## Goal

Bitclock is "a 3D-printed air quality monitor, desk clock, and customizable e-ink gadget" — an
e-ink desk companion that measures CO2, temperature, humidity and VOC levels so you can see the
air you are actually sitting in, alongside the time and the weather. Open-source and meant to be
hacked on: users write their own clockface apps against it.

`canary` is its board. A 2.71" 264x176 e-ink panel with no backlight, driven by an ESP32-S3 over
Wi-Fi 2.4GHz and BLE 5, powered from USB-C, in a 2.9" x 3.5" x 2.0" printed enclosure. Sold from
bitclock.io, USA only, in runs of a few hundred assembled by JLCPCB.

The design is four sheets: a root carrying the ESP32-S3 module, the 5V→3V3 LDO, the boot and reset
buttons, the power and status LEDs and thirteen testpoints; and three children — **Sensors**
(SCD40, SGP41, SHT40 on one I2C bus, plus the FPC connector out to the sensor board), **USB**
(the Type-C receptacle, CC resistors and ESD diodes) and **Display Port** (the e-ink FPC
connector and the panel's boost rails).

Two things about this project shape every later decision. It is a **sensor** board, so noise near
the analog parts costs accuracy in the number on the screen — which is why the 3V3 rail is an LDO
and not a switcher. And it is **open hardware people modify**, so the sheets are read by strangers:
a block without a title or a net without a meaningful name is a real cost here, not a style point.

## Schematic conventions

Read off the existing sheets except where marked defaulted.

- **Values.** Value carries the value alone, in the symbol style this project already uses:
  `1µF`, `100nF`, `10pF`, `4.7µF`, `10kΩ`, `5.1kΩ`, `470mΩ`, `0Ω` — µ for micro, Ω on every
  resistance. This is the sheet's own style, not a default. Two parts predate it and should be
  normalized: R2 `1k` → `1kΩ` and R3 `470R` → `470Ω`. For an IC, Value
  carries the ordering code (`TLV76733DRVT`, `SCD40-D-R2`). Tolerance, voltage rating and
  dielectric go in their own fields or are implied by the MPN — never in Value, which is what
  prints on the sheet.
- **References.** KiCad's letters (R, C, L, D, Q, U, J, SW, TP), numbered in reading order —
  left to right, top to bottom. **Flat and continuous across all four sheets**, not per-sheet
  hundreds: C1–C27 already spans root, USB and Display Port. Keep it flat; do not renumber to
  R101/R201.
- **Nets.** UPPER_SNAKE, named for function, active-low ending `_N` — already the practice here
  (`I2C_SDA`, `SPI_MOSI`, `SPI_CS_DISP`, `CHIP_EN`, `BUSY_N`, `RST_N`, `SGP4X_VDD`). A net
  belonging to one block is prefixed with it (`D_VCOM`, `D_VGH`, `D_BUSY_N` for the display).
  Rails are spelled as KiCad's power library spells them (`+3V3`, `+5V`, `GND`).
- **Wires and labels.** Wires within a block; labels only between blocks. **Rails run on power
  symbols and cross-sheet signals on hierarchical labels — this project uses no global labels at
  all.** Keep it that way: a new cross-sheet signal gets a hierarchical label on both sheets, not
  a global one. A label sits on the pin's free side with its text reading away from the part.
- **Layout.** Signal flows left to right; power at the top, ground at the bottom; connectors at the
  edges. A decoupling capacitor sits one grid step from the pin it serves. Every pin left open on
  purpose carries a no-connect. A `PWR_FLAG` where a rail enters from a connector.
- **Sheets.** A root sheet that reads as a block diagram, one child per functional block, each with
  its own power symbols — already the structure. A fifth block gets a fifth sheet rather than
  crowding an existing one. A4 landscape; grow the page rather than crowd it.
- **Text.** Each block gets a title and one line saying what it is for — the sheets already do
  this (`Linear regulator (LDO) 5v -> 3v3`, `Boot button`, `Chip enable delay`, `Testpoints`).
  Notes are concise; datasheet rules under a block only when asked. Where a symbol is wired to
  satisfy ERC rather than the circuit, say so in a note, as the root's `ERC Hacks` block does.

## Footprint conventions

Density `nominal` and a chevron pin-1 marker, both defaulted (see `assumed`).

- A footprint exists in the standard library for nearly every IC package: search by package name
  and pitch (`search_footprints`) before drawing one.
- **This project's own footprints live in `canary.pretty`, which is the only library
  `fp-lib-table` registers.** A part must not depend on a library that happens to be installed
  globally: six footprints here point at `Babldev:`, `EasyEDA2Kicad:` and `PCM_Espressif:` and do
  not resolve on a fresh machine. New work goes in `canary.pretty`, and those six should be copied
  into it.
- A drawn footprint (`create_library_footprint`) is transcribed from the datasheet's recommended
  land pattern, read as an image, with the datasheet cached first so the revision is on record.
- It is named the way the library names its neighbors — family, manufacturer, part, pins, pitch,
  orientation — so a reader can tell what it is without opening it.
- Its report says what was cut, what was derived, and what could not be read.
- Every footprint carries a 3D model whose file exists, so the finished board renders whole. A
  model is never needed to lay out or order a board: a part still without one is listed as open,
  never treated as a blocker. Three are open today (SW1-SW2, U4, U5).
- A model comes from, in order: the model a matching standard-library footprint already links
  (read it with `get_library_footprint`, write the same record with `items_create`); the
  manufacturer's own STEP file for that part number; anything else is a third-party model, which
  is not a reviewed source — ask the user before using one, and say where it came from. Every
  downloaded model is saved with `cache_model` so it is committed with the project.
- A standard footprint whose model file is missing gets a project copy
  (`copy_library_footprint`), the model attached to the copy, and the part's Footprint pointed
  at it.
- Look at a model before attaching it (`view_render_model`): one part, the right part, upright,
  in millimetres. A file over about 5 MB, or one that shows an assembly, is unusual: look for the
  vendor's simplified version or tell the user. After attaching, read the answer's bounds against
  the body and the datasheet's height, and render the footprint with its model
  (`view_render_footprint`, view model): pin 1 over pad 1, leads on pads, the body on the board.
  **Height matters on this board** — it goes in a printed enclosure 2.0" deep behind a panel, so
  check a tall part's height against `MAXIMUM_PACKAGE_HEIGHT` where a part carries it.

## Working on the schematic

- Read the conventions above before the first edit, and follow them without being asked.
- Draw one block at a time. After a block is placed and wired, render it (`view_render` by box)
  and read the picture against the conventions: text through a body, labels overlapping, a passive
  away from its pin, a wire crossing a body, a dangling end, an open pin with no no-connect. Fix
  before starting the next block.
- Before reporting done: render every sheet, at a zoom at which text and overlaps are legible,
  to make sure symbols, labels and other content are not overlapping in an undesirable way; run
  the checks; and show the user the renders with a summary. Findings that remain are listed, not
  hidden.
- Before starting the board: a schematic is ready for layout when every part `bom_get` lists
  has a footprint, and ready to order when each has an MPN and a cached datasheet. A generic
  passive is placed by value first and gets its footprint here. Fill the BOM before layout, list
  what is still open, and list the parts that still have no model as open
  (`bom_get` with the MODEL column) — open, not blocking.
- **This design is not order-ready as it stands**: every MPN field is empty and no datasheet is
  cached. Both are listed under `open_questions`; fill them before the next run rather than
  reading a part number out of Value at order time.
- A new sensor or bus goes on the existing I2C bus where it can. The bus already carries the
  SCD40 at 0x62 and the SHT40 at 0x44 (the AD1B suffix picks that address) — check a new part's
  address against those two, and against the `I2C Address` field the parts carry, before choosing
  the variant.
- When the user corrects a convention, update the section above so it sticks for every later
  session.

## Choosing parts

- **Requirements before the catalog.** Write what the part must do — function, the numbers that
  matter, a package the assembly method allows, the build quantity — before searching. A wrong
  premise is free to fix here and expensive later.
- **Generic before specific.** A passive or a commodity part is found by kind and value (a
  category with filters on DigiKey, a short keyword elsewhere), then chosen from what is in
  stock at the build quantity from a maker you would trust — never from a part number you recall.
- **Family before ordering code.** For an IC, search the family name (`SHT40`, `TPS62`,
  `STM32G0`), not a full ordering code, so the catalog shows every variant, the successors, and
  what is actually stocked. The exact suffix is picked from the results.
- **A full MPN confirms; it does not discover.** Search one only to check that a chosen part is
  real and buyable at quantity.
- **Short queries.** A catalog search matches part and family names, not descriptions. A sentence
  returns nothing; two or three words return the landscape.
- **What makes the pick.** In stock at 100+; lifecycle active; a datasheet the tool can fetch;
  stocked by both DigiKey and Mouser where there is a choice; price quoted at 100, never at one;
  tape-and-reel packaging.
- **Where the price actually comes from.** Parts are selected on DigiKey and Mouser, because that
  is what is connected and what has parametric data and fetchable datasheets. But JLCPCB
  assembles from LCSC stock, so a DigiKey price at 100 is a sanity check on the part, not the
  build cost — and a part LCSC does not stock costs an extra-part fee and a lead time at
  assembly. Check LCSC availability before committing to a part that is only interesting because
  it is cheap, and record the number in `LCSC Part`.
- **Every claim from a tool.** A part number comes from a search result, never from memory. A
  specification comes from the distributor's parametric data or the datasheet; anything else is
  marked unverified with what would confirm it. Say what was not checked.
- **A shortlist, not a winner.** Two to four candidates, one recommendation, the trade-off that
  separates them, and what would change the answer. Say which of them the symbol libraries
  already carry; a missing symbol costs one `create_library_symbol`, it is not a reason to pick
  a worse part.
- **Record it.** Bind the MPN on the symbol and cache the datasheet before placing. A part chosen
  for a reason worth remembering goes under `committed_parts` with the reason.
