# Truss Bridge Science Fair Project: 3D Model Brief

This file is the brief for Claude Code. Read all of it before writing any code.

## Fill these in before starting

| Item | Value |
|---|---|
| 3D printer model | Creality K2 Pro Combo (with CFS multi-filament unit) |
| Usable bed size (X × Y × Z, mm) | 300 × 300 × 300 |
| Nozzle diameter (mm) | 0.4 |
| Filament | PLA, 1.75 mm |
| Bolts we can get | M3 × 16 mm bolts with M3 hex nuts and washers (standard metric hardware, easy to buy in Tashkent); joints are also glued |
| CAD software the students know (if any) | None. All design and modelling is done by Claude Code. Slicing is done in Creality Print (the K2 Pro's slicer) |

If any of these are still blank, ask the user for them first. If the bed size is unknown, assume 220 × 220 mm.

## Who this is for

Three grade 9 students at the Presidential School in Tashkent are doing a physics science fair project:
**"Which Truss Design Is Strongest? Comparing the Strength-to-Weight Ratio of 3D-Printed Warren, Pratt and Howe Truss Bridges."**

The students will print the bridges on the school printer, load each one at the centre until it breaks, and compare the failure load ÷ bridge mass. They will also calculate the force in every member to predict where each bridge fails. They are learning, so explain design decisions in plain language and keep the code readable and well commented.

## What to build

### 1. Three bridge designs that differ only in their truss pattern

| Design | Web members | Notes |
|---|---|---|
| **Warren** | Diagonals only, alternating direction (no verticals) | Top-chord nodes sit halfway between bottom-chord nodes |
| **Pratt** | Verticals + diagonals sloping down *toward the centre* | Under a centre load, diagonals are in tension and verticals in compression |
| **Howe** | Verticals + diagonals sloping down *away from the centre* | The mirror of Pratt: diagonals in compression, verticals in tension |

Everything else must be **identical in all three designs**, because this is a fair test:

| Parameter | Default value |
|---|---|
| Overall length | 500 mm |
| Clear span between supports | 460 mm (20 mm bearing at each end) |
| Truss height (centre-line of bottom chord to centre-line of top chord) | 80 mm |
| Number of panels | 6 (panel length ≈ 76.7 mm on the clear span; adjust so nodes line up neatly) |
| Chord cross-section (in-plane width × thickness) | 8 mm × 6 mm |
| Web member cross-section | 6 mm × 6 mm |
| Distance between the two side trusses | 60 mm |
| Floor beams | One at every bottom-chord node, same size in all designs |
| Top lateral struts | One at each top-chord node (Warren has different top-node positions; keep the count as close as possible) |
| Loading point | The centre floor beam has a 6 mm hole or hook slot for hanging the load bucket |
| Deck | None (no solid deck). The load goes into the centre floor beam |

Put every parameter in **one config file** (e.g. `config.py` or `params.json`) so the students can change the span, height or member size in one place and regenerate everything.

Both supports must be at bottom-chord end nodes, and the bridge must have a bottom-chord node exactly at mid-span so the centre load sits on a node. Use an even number of panels.

### 2. Split each side truss into printable sections

A 500 mm truss will not fit on the bed, so:

- Each side truss is printed **flat** (lying in the XY plane, thickness = Z). This is the strongest orientation, because the layer lines run along the members.
- Split each side truss into the fewest sections that fit the bed with a 5 mm margin (on the K2 Pro's 300 × 300 mm bed, expect **2 sections**). **Do not split exactly at mid-span**, because that is where the load hangs. Use one longer and one shorter section (for example about 290 mm + 210 mm) so the joint sits away from the centre node.
- **Cut only the top and bottom chords. Never cut a web member.** Choose the split positions to make this possible. For example, cut the chords a short distance beside a node, where no web member is attached.
- Put the splits at the **same x-positions in all three designs** wherever the geometry allows. If that is impossible for Warren, explain the difference in `NOTES.md`.
- Join the sections with **printed splice plates**: one plate on each face of the chord, sandwiching the joint, held by 2 bolts on each side of the cut plus glue. Use the same splice plate design and bolt pattern in all three bridges.
- Add bolt holes sized for the bolts in the table above. For M3 bolts, use 3.4 mm clearance holes.
- Floor beams and lateral struts are separate small parts that bolt or press-fit into slots or tabs on the side trusses.

### 3. A test coupon

Make one small test part: a short length of chord with a single splice joint. The students should print this first to check bolt-hole fit and joint strength before they spend filament on full bridges.

## Recommended tools

Use Python so the whole model is parametric and can be regenerated with one command.

- **Preferred:** `build123d` (or `CadQuery`). Install with `pip install build123d`.
- **Fallback, if those will not install:** `shapely` + `trimesh`. Each side truss is a flat plate, so you can union rectangles for every member into a 2D outline with shapely, subtract the bolt holes, and extrude it to the thickness with `trimesh.creation.extrude_polygon`.
- Use `matplotlib` for drawings and diagrams.

Provide a single command that rebuilds everything, e.g. `python build_all.py`.

## Deliverables

```
bridge-project/
├── CLAUDE.md                 (this file)
├── config.py                 (all dimensions in one place)
├── build_all.py              (one command regenerates everything)
├── geometry.py               (node/member definitions for Warren, Pratt, Howe)
├── analysis.py               (force calculation, see below)
├── stl/
│   ├── warren/   side_truss_section_1.stl … floor_beam.stl, splice_plate.stl …
│   ├── pratt/
│   ├── howe/
│   └── test_coupon.stl
├── previews/                 (PNG renders of each part and the assembled bridge)
├── analysis/                 (force diagrams + tables for each design)
├── PARTS_LIST.md             (every part, quantity per bridge, volume, mass)
├── ASSEMBLY.md               (step-by-step assembly instructions with images)
├── PRINTING_GUIDE.md         (beginner guide to slicing and printing in Creality Print)
└── NOTES.md                  (design decisions and anything the students should know)
```

### Parts list and estimates (`PARTS_LIST.md`)

For each design, list:
- every part and how many are needed per bridge,
- each part's volume (from the mesh) and its mass at 1.24 g/cm³ (PLA, printed close to solid),
- the total mass per bridge,
- the total filament for **9 bridges** (3 designs × 3 copies) + the test coupon + 25% spare,
- a rough print time estimate, stated as an estimate, using about 30 g/hour (the K2 Pro is a fast printer; nearly solid parts print slower than its top speed).

Recommended slicer settings to put in the notes: 0.2 mm layers, 4 walls, 100% infill (the members are thin, so they are effectively solid), and the **same settings for every bridge**.

### Force analysis (`analysis.py`)

This is the physics core of the project, so keep it clear enough for a grade 9 student to follow.

1. For each design, build the 2D truss from the same node and member data used for the CAD model. Use a single source of truth so the analysis and the printed model can never disagree.
2. Use a pin support at one end and a roller at the other, with a load **P = 1 N** at the mid-span bottom node. Since this is one of two side trusses, each truss carries half the bucket load.
3. Solve for every member force using the **method of joints** (a linear system with numpy is fine). Mark tension as positive and compression as negative.
4. For each compression member, estimate the Euler buckling load
   `F_cr = π² · E · I / L²`, with `I = b·t³/12` about the weaker axis.
   Use E ≈ 3 GPa for printed PLA, and state clearly that this is an assumed value.
5. Use member strength (tension) or buckling (compression) to predict:
   - which member fails first,
   - the predicted failure load in N and kg of hanging load.
6. Output for each design:
   - a table (CSV + markdown) of member, length, force per 1 N load, tension/compression, and the predicted limit,
   - a diagram of the truss with members coloured by force (blue = tension, red = compression) and the predicted first-failure member highlighted.
7. Print a short comparison of the three designs: predicted failure load, mass, and predicted strength-to-weight ratio.

## Checks before you say you are done

- Every STL is watertight and opens without errors (check with `trimesh`: `mesh.is_watertight`).
- Every part fits the bed size above with a 5 mm margin. Print a table showing each part's bounding box.
- No web member is cut by a split.
- Splice plate holes line up with the chord holes. Check this numerically, not by eye.
- In the analysis, every joint is in equilibrium (the sum of forces is ~0), and the support reactions add up to the applied load.
- Pratt diagonals come out in tension and Howe diagonals in compression under the centre load. If not, the geometry is mirrored the wrong way.
- Render a PNG preview of every part and the assembled bridges, and look at them.

## The students are beginners

The students do not know any CAD or programming software, so Claude Code does all the modelling. Because of that:

- Install and run everything yourself. Never ask the students to install tools or run commands they would not understand. If they ever need to run something, give the exact command to copy and paste.
- To change a dimension, the students should only have to tell you in plain words (e.g. "make the bridge 10 mm taller"). You then edit `config.py` and regenerate everything.
- Write `PRINTING_GUIDE.md`: a step-by-step beginner guide for **Creality Print** on the K2 Pro. It should cover how to import the STL files, place them flat on the bed, apply the slicer settings from this brief, check the estimated time and filament, and send the job to the printer. Include which parts can share one print job (plate) to save time.
- After each change, show preview images so the students can see what changed without opening any 3D software.

## Style

- Explain each step in plain English before doing it, because the students will present this work.
- Prefer simple, readable code over clever code.
- Do not add designs, features or materials beyond this brief without asking.
