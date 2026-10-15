# Seamly pattern catalog – specification

Oct 15, 2026 · @Beáta Dudás

## Goal and scope

The pattern catalog is a static website that presents parametric sewing patterns made in Seamly2D and makes them downloadable. The site is built automatically from the pattern files, with no server, database or user accounts.

The problem it solves: patterns from the Seamly community are scattered across forum threads, and the shared pattern site (my-pattern.cloud / seamly.cloud) is gone. There is no place where documented, tested parametric patterns can be searched.

**In scope for the first round:**

- a catalog page listing the patterns, with filtering,
- a page per pattern with a description, previews for several sample sizes, and lists of the required body measurements and the variables,
- a downloadable package (pattern file and sample measurement file),
- automatic build and publishing from the files committed to the repository.

**Out of scope for the first round:**

- online pattern editing,
- previews or downloads computed from the visitor's own measurements,
- user accounts, uploads by others, comments,
- more than two languages (the site launches in Hungarian and English).

## Terms

| Term | Meaning |
| --- | --- |
| Pattern | One catalog entry: a .sm2d file and its metadata. |
| Block | A basic draft without styling (e.g. straight skirt block, close-fitting bodice block). |
| Design | A pattern for a specific garment, developed from a block. |
| Drafting system | The pattern-drafting method the pattern is based on (e.g. Aldrich, Müller, own). |
| Measurement file | A SeamlyMe individual measurement file (.smis, formerly .vit) holding body measurements. |
| Sample size | A SeamlyMe measurement file stored in the repository for the previews, not belonging to a real person (e.g. 3–5 sizes from a size chart). Its name and order come from the file. |
| Body measurement | A standard Seamly measurement name, e.g. waist\_circ (G07), hip\_circ (G09). |
| Variable | A name starting with # defined in the pattern, with a formula (e.g. #skirt\_length = 60). |
| Pattern piece | An outline (piece) defined in the pattern, e.g. back, front. |
| Preview | A true-to-scale SVG drawing of the pattern pieces for one sample size. |
| Build | The process that produces the static site from the repository contents. |

## Actors and use cases

There are two actors: the visitor, who browses and downloads, and the editor, who adds patterns through the repository. The site has no sign-in; editor rights are write access to the repository.

**Visitor** (a sewist, pattern maker or hobbyist who uses Seamly2D)

1. UC-V1 Find a pattern: opens the catalog, filters by garment, looks through the cards.
2. UC-V2 Evaluate a pattern: on the pattern page, switches between sizes in the preview, reads the description, the required body measurements and the variables.
3. UC-V3 Download a pattern: downloads the package, opens it in Seamly2D and swaps the measurement file for their own.

**Editor** (Betti)

1. UC-E1 Add a new pattern: creates the pattern folder, copies in the .sm2d file, fills in the metadata, commits.
2. UC-E2 Update a pattern: saves a new version in Seamly2D or fixes the description, commits.
3. UC-E3 Check locally: runs the build on her own machine before publishing and reviews the result.
4. UC-E4 Fix an error: if the build stops (missing metadata, unsupported tool, personal data in the file), the error message shows what to fix.

## Functional requirements

The first round has 19 requirements. There is no public launch without the “Must” items; the “Should” items can come later.

| ID | Requirement | Priority |
| --- | --- | --- |
| F-01 | The build creates a pattern page from every pattern folder (pattern file + metadata). | Must |
| F-02 | The build creates a catalog page of all patterns, one card each with preview, title, drafting system and garment. | Must |
| F-03 | The catalog can be filtered by garment and type (block / design). | Must |
| F-04 | The build creates an SVG preview for every pattern and every sample size, true to scale, with a 1 cm grid and a 10 cm ruler. | Must |
| F-05 | On the pattern page the visitor can switch between sample sizes without reloading the page. | Must |
| F-06 | The build collects the body measurements used in the pattern's formulas and shows them in a table with name, Seamly code and value per sample size. | Must |
| F-07 | The pattern page lists the variables with their formula, the explanation from the metadata, and an “adjustable” / “from measurements” label. | Must |
| F-08 | The build creates a zip package per pattern: the pattern file, one sample measurement file and a short readme. In the package the pattern file refers to the enclosed measurement file. | Must |
| F-09 | The build stops with a clear error if the pattern file contains personal data (a filled-in customer name, a reference to an individual measurement file). | Must |
| F-10 | The build stops if required metadata is missing or the pattern file uses an unsupported tool. The error message names the pattern, the tool and the point. | Must |
| F-11 | The pattern page shows the list of pattern pieces and the drafting facts (ease, darts). | Must |
| F-12 | The pattern page shows the format version, the Seamly2D version it was saved with, and the license. | Must |
| F-13 | A “Test garments” section on the pattern page: descriptions and photos of sewn pieces, if any. | Should |
| F-14 | The build also writes a machine-readable catalog file (JSON) listing the patterns and the body measurements they use. | Should |
| F-15 | Pattern piece names from the metadata appear in the preview in the page language. | Should |
| F-16 | The preview marks the fold line on pieces cut on the fold. | Should |
| F-17 | Every page is produced in both Hungarian and English. Proposed URL structure: Hungarian at the root, English under /en/ (e.g. /en/patterns/\<slug>/). | Must |
| F-18 | Every page has a language switch that leads to the same content in the other language; pages carry hreflang markup. | Must |
| F-19 | Interface texts live in a resource file; pattern texts (title, description, piece names, variable explanations) live in the metadata in both languages. Body measurement names come from Seamly's own translations. If a translation is missing, the build warns and uses the Hungarian text. | Must |

## Data model and file formats

The git repository is the single source of truth. Each pattern is a folder; the folder name is the pattern's URL identifier (slug).

```
patterns/<slug>/
  pattern.sm2d          # the Seamly2D pattern, without personal data
  meta.yaml             # catalog data
  photos/               # optional photos of test garments
measurements/samples/
  *.smis                # sample sizes for the previews
measurements/labels.yaml  # own overrides for the Seamly names
```

**Metadata fields**

| Field | Type | Required | Note |
| --- | --- | --- | --- |
| title | text | yes | Pattern name in Hungarian. |
| slug | text | yes | Matches the folder name. |
| garment | text | yes | Garment, for filtering (skirt, bodice, sleeve …). |
| kind | block / design | yes | For filtering. |
| system | text | yes | Drafting system and source. |
| based\_on | slug | no | For a design: the block it was developed from. |
| license | text | yes | E.g. CC BY-SA 4.0. |
| seamly\_version | text | yes | The Seamly2D version it was saved with. |
| summary | text | yes | 2–3 sentence description. |
| pieces | name → display name | yes | For the piece names in the .sm2d. |
| variables | name → {label, help, adjustable} | yes | One entry per .sm2d variable. |
| construction | list of text | no | Drafting facts. |
| tested | list {date, note, photo?} | no | Test garments. |
| en | block | yes | English texts: title, summary, pieces, variables (label, help), construction. Same keys as the Hungarian fields. |

**The Seamly files the system relies on**

Both are XML. The schemas are in the Seamly2D source code (src/libs/ifc/schema), and the build can validate against them.

- **.sm2d** (format version 0.7.4–0.7.5): header (patternName, customer, measurements reference), variables, then per draftBlock: calculation (points, lines, curves), modeling (copies of the objects used in pieces), pieces (pieces with their node lists).
- **.smis** (0.3.3–0.3.4): personal (name, birth date, email) and body-measurements. A value can be a number or a formula referring to other measurements.

The system only reads the .sm2d file and, for the download, rewrites its measurement file reference. It makes no other change to it.

**Body measurement names**

Names and descriptions of body measurements come from Seamly2D's own sources; we do not translate them again:

- English name, description and code (e.g. waist\_circ, “Waist circumference”, G07): src/libs/vpatterndb/vtranslatemeasurements.cpp, 247 measurements.
- Hungarian name: share/translations/measurements\_hu\_HU.ts, all 247 measurements translated.
- The Hungarian translation is literal in places (e.g. “Magasság: Nyak Hát”). Where a different professional term is needed, measurements/labels.yaml overrides it; everything else keeps the Seamly name.
- The build extracts these once into its own data file (code, name, hu, en, description) and can be re-run when Seamly is updated.
- For custom measurements (starting with @), the name comes from the measurement file's full\_name field or from the override file.

## Build pipeline

The build is a single command that runs the same way on your own machine and in CI. On an error nothing is published, and the previously published version stays live.

&#91;embedded content: build pipeline · 5 steps, 1 decision\]

Validation checks every pattern and reports all errors at once, not just the first.

**Steps in detail**

1. Read input: pattern folders, sample sizes, body measurement labels.
2. Validate: .sm2d and .smis against their schemas; required metadata fields; every .sm2d variable has an explanation; no personal data; every body measurement used is present in every sample size.
3. Evaluate: points, curves and pattern piece outlines per pattern and per sample size.
4. Generate: SVG previews, pattern pages, catalog page, zip packages, catalog.json, stylesheet.
5. Publish: upload the output folder to GitHub Pages (automatically in CI; locally viewable with a simple web server).

## Preview renderer

Previews are produced by our own formula evaluator and geometry engine, not by Seamly2D. Reason: the current Seamly2D source needs Qt 6.5+, expects a graphical environment and is hard to run in CI. A narrow, well-tested engine for the common tools is simpler. In the prototype this is about 300 lines of Python.

**Formula evaluation**

- Sources, in order: body measurements (from the measurement file; these can themselves be formulas), variables (# prefix), then names created while drafting: Line\_A\_B (segment length), AngleLine\_A\_B, Spl\_A\_B (curve length), CurrentLength (the segment of the current tool).
- Operations: + − \* / ^, parentheses, sqrt, abs, min, max, sin, cos, tan (in degrees), conditional expression (a ? b : c), comparisons.
- Number format: decimal point; scientific notation also occurs (5.50672e-06).
- A dedicated parser is required, not the language's eval: formulas come from external files.

**Geometry conventions (matching Seamly2D)**

- Unit: the pattern's unit (cm); the y axis grows downward.
- Angle: in degrees, counter-clockwise from the +x direction as seen on screen: 90 = up, 270 = down.
- The perpendicular (normal) tool adds +90 degrees to the segment direction, then its own angle.
- Curve (simpleInteractive): the two control points are placed from the end points by angle1/length1 and angle2/length2; cubic Bézier.

**Supported tools**

| Tool (.sm2d type) | First round | Note |
| --- | --- | --- |
| single, endLine, alongLine, normal | yes |  |
| intersectXY, pointOfIntersection, height | yes |  |
| line | yes |  |
| simpleInteractive, cubicBezier | yes |  |
| shoulder, bisector, lineIntersect | yes | Needed for bodice blocks. |
| cubicBezierPath, splinePath | yes | Needed for armholes. |
| cutSpline, cutSplinePath, curveIntersectAxis | later | Cutting curves. |
| arc, cutArc, circle intersections | later |  |
| operations (rotate, mirror, move) | later | Common in designs. |

**Assembling pattern pieces**

- A piece's nodes refer to calculation objects through the modeling elements.
- For a curve node, reverse="1" means reversed direction. If absent, the direction is chosen by proximity to the previous point.
- Internal paths (iPaths) are drawn dashed.
- Seam allowance is not drawn in the first round.

**Verification**

Reference tests prove the engine correct: point coordinates of a few patterns, exported from Seamly2D (or measured by hand), are compared with a tolerance of 0.01 cm.

## User interface

There are two page types, both static HTML. JavaScript is needed only for filtering and size switching; all content is readable without it.

**Catalog (/)**

- Header: site name, one-line description.
- Introduction: 2–3 sentences on what a parametric pattern is and how to use it.
- Filter buttons: garment, type. The active filter is visibly highlighted.
- Card grid: preview (default sample size), type, title, drafting system, number of body measurements and variables.

**Pattern page (/patterns/\<slug>/)**

1. Breadcrumb back to the catalog.
2. Header: type, title, system, description; beside it the download button and a facts box (format, Seamly version, unit, license).
3. Preview: size tabs, the drawing on a cm grid below them, a note on scale and the missing seam allowance.
4. Required measurements table: display name, Seamly name, code, value per sample size.
5. Pattern pieces and drafting facts.
6. Variables table: name, formula, type label, explanation.
7. Test garments: list with date, note, photo; an empty state with text.

**Appearance**

- Light and dark theme following the system setting.
- The preview SVG uses color variables, so it follows the theme.
- On phones, tables and the drawing scroll within their own containers; the page never scrolls sideways.

## Non-functional requirements

| ID | Requirement |
| --- | --- |
| NF-01 | Hosting: GitHub Pages, monthly cost 0. No running server code, no database. |
| NF-02 | Build: runs and publishes automatically after a push to the `main` branch; finishes within 2 minutes for 50 patterns. |
| NF-03 | Local run: can be built and viewed on one's own machine with a single command, on Windows too. |
| NF-04 | Deterministic output: byte-identical files from the same input (no timestamps in generated content). |
| NF-05 | Page size: one pattern page with its previews under 500 kB; the only external dependency is fonts. |
| NF-06 | Accessibility: tabs and filters usable by keyboard, visible focus, drawings have a text label. |
| NF-07 | Maintainability: the drawing engine and the page generator are separate modules; the engine can be tested and reused independently of the website (e.g. later in the browser or on a server). |
| NF-08 | Extensibility: support for a new Seamly tool is added by adding one handler, without changing other code. |
| NF-09 | Version tolerance: reads .sm2d 0.7.4 and 0.7.5, and .smis 0.3.3 and 0.3.4. On an unknown version it warns and does not stop. |

## Privacy and licensing

The site collects and stores no personal data, so the first round needs no cookie notice or privacy policy. This changes if analytics, accounts or server-side measurement handling are added.

The risk lies in the files committed to the repository:

- Seamly2D saves the customer's name (customer) and the measurement file path into the pattern file.
- SeamlyMe saves the name, birth date, email address and body measurements into the measurement file.
- Your own or a client's measurement file never goes into the repository. The measurements/individual folder is excluded from version control.
- The build checks (F-09): the customer field is empty, the measurements reference points to a sample size, and the personal section of the sample files holds no real name.
- Optionally a pre-commit hook runs the same check locally.

**License**

- Patterns: CC BY-SA 4.0 suggested (free to use and modify, with attribution, under the same license).
- Code: MIT. If Seamly2D source code were reused, GPL-3.0 would be required. Our own engine avoids this.
- Patterns based on a published drafting system (e.g. Aldrich): the system and the book may be named, but the book's text, figures and trademarks may not appear on the site. Descriptions are our own text.

This is not legal advice; before a public launch the license choice is worth reviewing with an expert.

## Later rounds

Parameter adjustment builds on top in three levels, cheapest first. Each reuses the first round's engine (NF-07).

| Round | Content | New technology | Personal data |
| --- | --- | --- | --- |
| 2a | Create your own measurement file: a form with the body measurements used by the selected patterns, .smis download. | browser JS | never leaves the browser |
| 2b | Live preview: the drawing recomputed with the visitor's measurements and the “adjustable” variables. | the engine in the browser (JS/TS or WebAssembly) | never leaves the browser |
| 2c | Printable PDF (tiled A4) with the visitor's measurements. | PDF generation, in the browser or on a server | on a server, a privacy policy is needed |

2b is the biggest step. If the engine is written in .NET, the same code can run in the browser with Blazor WebAssembly; this is worth considering when choosing the first round's technology.

Further ideas, unprioritized: seam allowance in the preview, showing the block–design relationship, RSS for new patterns.

## Risks and open questions

**Risks**

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Our engine computes differently from Seamly2D. | The preview does not match what the downloader sees. | Reference tests per pattern, 0.01 cm tolerance. |
| A new pattern uses an unsupported tool. | The build stops. | Clear error (F-10); extend the tool list as needed. |
| The Seamly file format changes. | The build draws incorrectly or stops. | Version check (NF-09), schemas from the Seamly repository. |
| Curve handles are given as fixed numbers. | Distorted curves at small and large sizes. | Drafting guideline: handle lengths as formulas; previews at several sizes reveal the problem. |
| Personal data ends up in the public repository. | Privacy incident, hard to remove from git history. | F-09 check, pre-commit hook, excluded folder. |

**Open questions for detailed design**

- [x] Which language for the engine and the generator? Decision: .NET, with Blazor WebAssembly for round 2b.
- [x] The .NET build renders the pages' HTML itself. Decision: static rendering of Razor components (HtmlRenderer), because the same components can run interactively in Blazor WebAssembly in round 2b.
- [ ] Sample sizes: in the first round based on the Aldrich chart, as SeamlyMe files in the measurements/samples folder; size names come from the file, no S/M/L labels. Open: may the book's chart values be published in a public repository, and how many sizes are needed (3–5, covering both ends of the range)?
- [ ] What is the site's final name? Hosting is GitHub Pages (decision); a custom domain can be added later.
- [ ] For which body measurements is an own professional name needed in the override file instead of Seamly's Hungarian translation?
- [x] English version: decision, the site is available in Hungarian and English from launch (F-17–F-19).
- [ ] How is reference test data produced if Seamly2D's command-line export is not available?
