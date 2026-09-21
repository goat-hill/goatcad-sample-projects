---
goal: Bitclock (canary board) — an ESP32-S3 e-ink desk clock that measures indoor air quality (CO2, temperature, humidity, VOC/NOx); kept as an open-source reference design
stage: production
scale: { quantity: null, anticipated: null }
budget: { bom_target_usd: null }
assembly:
  method: pcba
  house: JLCPCB
  min_passive: "0402"
  fine_pitch_ok: true
  bga_ok: true
sourcing:
  packaging: either
  distributors: [digikey, mouser]
  second_source: preferred
  lifecycle: active-only
  origin: dont-care
  rohs: required
environment:
  temp_range_c: [0, 40]
  supply: USB-C 5V in, TLV76733DRVT LDO down to a single +3.3V rail; no battery
datasheets:
  path: datasheets/
  committed: false        # PDFs are gitignored for licensing; datasheets/index.json (URL + SHA-256 per file) is committed
  extra_paths: []
models:
  path: canary.3dshapes/
  committed: true
footprints:
  density: nominal
  pin1_marker: chevron
committed_parts:
  - mpn: ESP32-S3-WROOM-1-N8
    reason: Wi-Fi and BLE with 8MB flash for user-written clockface apps. A pre-certified module, so the board needs no RF layout or antenna tuning.
  - mpn: SCD40-D-R2
    reason: Photoacoustic NDIR CO2, a real measurement rather than an eCO2 estimate derived from VOC. The CO2 reading is the headline feature.
  - mpn: SGP41-D-R4
    reason: VOC and NOx index from one part; uses the SHT40 reading for humidity compensation.
  - mpn: SHT40-AD1B-R2
    reason: Temperature and humidity for display and for SGP41 compensation. AD1B is the 0x44 address variant, clear of the SCD40 at 0x62.
  - mpn: TLV76733DRVT
    reason: 5V to 3.3V at 1A in a 2x2mm WSON — headroom for ESP32-S3 TX bursts and the e-ink panel, without a switcher's noise near the sensors.
assumed:
  - stage                     # the design shipped as a product; kept here as a reference, no build planned
  - assembly.min_passive      # read off the board: passives are 0402 and 0603
  - assembly.fine_pitch_ok    # read off the board: WSON-6 P0.65mm, SHT40 DFN 1.5x1.5mm, 24-pin 0.5mm FPC
  - assembly.bga_ok           # nothing on the board needs one; JLCPCB can place them
  - sourcing.packaging        # no build quantity, so either
  - sourcing.second_source
  - sourcing.lifecycle
  - sourcing.origin
  - sourcing.rohs
  - environment.temp_range_c  # an indoor desk gadget
  - datasheets.committed      # taken from .gitignore and THIRD-PARTY.md, not asked
  - models.path
  - models.committed
  - footprints.density        # nominal because the board is machine-assembled
  - footprints.pin1_marker
open_questions:
  - No build is planned (reference project), so there is no quantity to price or check stock against. Part picks are judged at a nominal ~100-piece run until that changes.
  - JLCPCB assembles from LCSC stock, which DigiKey and Mouser searches cannot confirm. A part that is in stock at DigiKey or Mouser is not necessarily in stock at LCSC.
---

## Goal

Bitclock (https://bitclock.io) is open-source hardware: an e-ink desk clock that shows the time,
the weather and user-written clockface apps, and measures indoor air quality (CO2, temperature,
humidity, VOC/NOx). `canary` is its main board: 4 layers, 78 × 48 mm, ESP32-S3-WROOM-1, powered
from USB-C. It was designed in KiCad for JLCPCB assembly and is being migrated to GoatCAD.
In the user's words: "No plans to order at the moment. This is just a reference project."

## Schematic conventions

Taken from the existing sheets where they already had a style; standard practice otherwise.

- **Values.** Value carries the value alone, written the way these sheets write it: with the unit
  and the proper symbols — `1µF`, `100nF`, `22pF`, `10kΩ`, `5.1kΩ`, `0Ω`, `10µH`. Tolerance,
  voltage rating, dielectric and package go in their own fields (`Tolerance`, `Voltage`,
  `Footprint`) or are implied by the MPN — never in Value, which is what prints on the sheet.
  An IC or connector's Value is its MPN.
- **References.** KiCad's letters (R, C, L, D, Q, U, J, Y, SW, TP, FB), numbered straight through
  the whole design (C1, C2, … across every sheet), as this design already does — not per sheet.
- **Parts.** Each placed part carries a `JLCPCB Part #` field (the LCSC number) alongside its MPN,
  as the existing sheets do.
- **Nets.** UPPER_SNAKE, named for function (`I2C_SDA`, `SPI_MOSI`, `USB_D+`); active-low ends
  in `_N` (`RST_N`, `BUSY_N`). Nets local to a block may carry a block prefix (`D_VGH` for the
  display). Rails are spelled as this design spells them: `+3.3V`, `+5V`, `VBUS`, `GND`.
- **Wires and labels.** Wires within a block; local labels to tidy a sheet; hierarchical labels
  across sheets; power symbols for rails. A label sits on the pin's free side with its text
  reading away from the part.
- **Layout.** Signal flows left to right; power at the top, ground at the bottom; connectors at the
  edges. A decoupling capacitor sits one grid step from the pin it serves. Every pin left open on
  purpose carries a no-connect. A `PWR_FLAG` where a rail enters from a connector.
- **Sheets.** A root sheet (`canary.kicad_sch`) that holds the MCU and power and reads as the
  block diagram, and one child sheet per functional block: `usb`, `sensors`, `display_port`. Each
  has its own power symbols. A4 landscape; grow the page rather than crowd it.
- **Text.** Each block gets a title and one line saying what it is for. Notes are concise;
  datasheet rules under a block only when asked.

## Footprint conventions

- A footprint exists in the standard library for nearly every IC package: search by package name
  and pitch (`search_footprints`) before drawing one.
- A drawn footprint (`create_library_footprint`) is transcribed from the datasheet's recommended
  land pattern, read as an image, with the datasheet cached first so the revision is on record.
- It is named the way the library names its neighbors — family, manufacturer, part, pins, pitch,
  orientation — so a reader can tell what it is without opening it.
- Its report says what was cut, what was derived, and what could not be read.
- Every footprint carries a 3D model whose file exists, so the finished board renders whole. A
  model is never needed to lay out or order a board: a part still without one is listed as open,
  never treated as a blocker.
- A model comes from, in order: the model a matching standard-library footprint already links
  (read it with `get_library_footprint`, write the same record with `items_create`); the
  manufacturer's own STEP file for that part number; anything else is a third-party model, which
  is not a reviewed source — ask the user before using one, and say where it came from. Every
  downloaded model is saved with `cache_model` so it is committed with the project, and gets a
  row in the repository's THIRD-PARTY.md.
- A standard footprint whose model file is missing gets a project copy
  (`copy_library_footprint`), the model attached to the copy, and the part's Footprint pointed
  at it.
- Look at a model before attaching it (`view_render_model`): one part, the right part, upright,
  in millimetres. A file over about 5 MB, or one that shows an assembly, is unusual: look for the
  vendor's simplified version or tell the user. After attaching, read the answer's bounds against
  the body and the datasheet's height, and render the footprint with its model
  (`view_render_footprint`, view model): pin 1 over pad 1, leads on pads, the body on the board.

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
- **What makes the pick.** In stock at the build quantity; lifecycle active; a datasheet the tool
  can fetch; stocked by more than one distributor when the project asks for it; price at the
  build quantity, never at one; cut tape when the project builds in cut tape. This board is
  assembled by JLCPCB, so a pick also needs an LCSC number — say when that could not be checked.
- **Every claim from a tool.** A part number comes from a search result, never from memory. A
  specification comes from the distributor's parametric data or the datasheet; anything else is
  marked unverified with what would confirm it. Say what was not checked.
- **A shortlist, not a winner.** Two to four candidates, one recommendation, the trade-off that
  separates them, and what would change the answer. Say which of them the symbol libraries
  already carry; a missing symbol costs one `create_library_symbol`, it is not a reason to pick
  a worse part.
- **Record it.** Bind the MPN on the symbol and cache the datasheet before placing. A part chosen
  for a reason worth remembering goes under `committed_parts` with the reason.
