# canary (Bitclock) — a GoatCAD project

`canary` is the main board of Bitclock (https://bitclock.io), an open-source ESP32-S3 e-ink
air-quality desk clock, kept here as a reference design.

This board is designed in GoatCAD (https://goatcad.com). Work on it through the `goatcad` MCP
tools, which edit the design open in the app as undoable steps; the `.kicad_*` files are its
on-disk format, not something to edit by hand. If the `goatcad` tools are not available, ask the
user to open `canary.kicad_pro` in GoatCAD.

Design context, constraints and house style live in [board-intent.md](board-intent.md) — call
`get_project_context` before choosing a part, editing a sheet, or drawing a footprint.

- Footprints in `canary.pretty/` and 3D models in `canary.3dshapes/` — both committed.
- Datasheets in `datasheets/`: only `index.json` is committed; the PDFs are gitignored for
  licensing and re-fetched from the URLs in the index.
- Third-party footprints and models are itemized in [../THIRD-PARTY.md](../THIRD-PARTY.md) —
  add a row when you add one.
- When the user corrects a convention, record it in board-intent.md, not here.
