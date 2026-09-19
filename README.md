# GoatCAD sample projects

| Project | Description |
|---|---|
| [`bitclock/`](bitclock/) | `canary` — the 4-layer ESP32-S3 board behind Bitclock, an e-ink air-quality desk clock |

## Opening a project

Clone and open the `.kicad_pro` in GoatCAD (or KiCAD). Each
project is self-contained: its footprint library and 3D models resolve through
`${KIPRJMOD}`, so nothing depends on a global library being installed.

## License

The design work — schematics, layouts, project files, design-intent documents,
and original footprints — is MIT licensed. See [`LICENSE`](LICENSE).

The vendor 3D models and two footprints taken from the KiCad official library
are third-party material under their own terms, and are itemized in
[`THIRD-PARTY.md`](THIRD-PARTY.md).
