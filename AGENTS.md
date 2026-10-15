# AGENTS.md

Guidance for AI coding agents working in this repository.

## Project

A static site that catalogs and publishes parametric Seamly2D sewing patterns, built from pattern files committed to this repo — no server, database, or user accounts. Full product spec, data model, build pipeline, and open questions: [docs/plans/Seamly mintatár – specifikáció.md](docs/plans/Seamly%20mintat%C3%A1r%20%E2%80%93%20specifik%C3%A1ci%C3%B3.md).

No application code exists yet — this repo currently holds only the spec and planning docs. Check back here and in `docs/plans/` before assuming build/test commands; add them to this file once the project is scaffolded.

## Key decisions already made (see spec for rationale)

- **Stack: .NET.** Pages are rendered statically via Razor components (HtmlRenderer), chosen specifically so the same components can later run interactively in Blazor WebAssembly (round 2b, live client-side preview).
- **Custom geometry/formula engine, not Seamly2D itself.** Seamly2D's source needs Qt 6.5+ and a graphical environment, which doesn't run well in CI, and its code is GPL-3.0 (this project's code is MIT, so Seamly2D source must not be incorporated). The engine is validated against reference points exported from Seamly2D, 0.01 cm tolerance.
- **Single source of truth is the git repo.** Each pattern is a folder under `patterns/<slug>/`; sample measurement files live under `measurements/samples/`. See the spec's "Adatmodell és fájlformátumok" section for exact file layout and metadata fields.
- **No personal data in the repo, ever.** `.sm2d`/`.smis` files must not carry a customer name or a path to an individual's measurement file. The build must fail the whole run (not just warn) if it finds any (F-09). This is a hard constraint, not a style preference.
- **Deterministic build output** (no timestamps in generated content) and **graceful version tolerance** (unknown `.sm2d`/`.smis` minor versions warn, not fail) — both are explicit non-functional requirements (NF-04, NF-09).
- **Bilingual from the start** (Hungarian default, English under `/en/`), not a later retrofit — content fields carry both languages in metadata from day one (F-17–F-19).

## Working in this repo

- Treat `docs/plans/*.md` as the design record. When implementation decisions diverge from what's written there, update the spec in the same change rather than letting it drift.
- The spec's open questions section marks unresolved decisions with `[ ]`; don't silently resolve one in code without flagging it to the user.
