# Knitting Pattern Visualization

A proof-of-concept that takes a real knitting pattern and produces both a full specification reference page and a pattern-accurate 3D model rendered as technical drawings. The goal: if you hand this to a knitter, they should know exactly what they're making.

**Live site:** [garygisclair.github.io/knitting](https://garygisclair.github.io/knitting/)

---

## What This Project Does

This project takes the [Craft Fix Circular Knit Hat](https://www.craftfix.com/circular-knit-hat-pattern/) pattern (a free, beginner-friendly stockinette beanie in 8 sizes) and produces:

1. **A complete pattern specification page** with size charts, gauge info, instructions, crown decrease schedule, and dimensional diagrams
2. **A pattern-accurate 3D model** built in Blender using the exact mathematical specifications from the pattern
3. **Technical wireframe drawings** rendered from the 3D model in an architectural presentation style

The 3D model is not decorative — every dimension is derived from the pattern's gauge and stitch counts using knitting math.

---

## How It Was Built — Step by Step

### Step 1: Research the Pattern

We needed a real knitting pattern with complete specifications: gauge, stitch counts, finished measurements, and a full crown decrease schedule. We searched for free patterns from well-known sources (Craft Fix, Tin Can Knits, Purl Soho, Nimble Needles) and selected the **Craft Fix Circular Knit Hat** because it had the best data:

- 8 sizes (Newborn through Adult XL)
- Exact gauge: 18 stitches and 24 rounds per 4 inches in stockinette on 5mm needles
- Cast-on counts for every size
- Complete crown decrease schedule with stitch-by-stitch instructions
- Finished measurements for fitted and slouchy versions

**Source:** [craftfix.com/circular-knit-hat-pattern](https://www.craftfix.com/circular-knit-hat-pattern/)

### Step 2: Calculate the 3D Model Dimensions

Every dimension of the 3D model was derived from the pattern's gauge and stitch counts. No measurements were estimated or eyeballed. Here's the math for Adult Size 7:

**Gauge constants:**
- 4.5 stitches per inch (18 stitches / 4 inches)
- 6 rounds per inch (24 rounds / 4 inches)

**Derived dimensions:**

| Parameter | Calculation | Result |
|-----------|-------------|--------|
| Finished circumference | 88 stitches / 4.5 sts/in | 19.56" / 49.7 cm |
| Body radius | 19.56" / (2 x pi) | 3.112" / 7.9 cm |
| Total height (fitted) | Pattern specification | 9.0" / 22.9 cm |
| Brim height (1x1 rib) | 9 rounds / 6 rnds/in | 1.5" / 3.8 cm |
| Crown depth | 16 rounds / 6 rnds/in | 2.67" / 6.8 cm |
| Body height | Total - crown - brim transition | 5.17" / 13.1 cm |
| Rib contraction | ~12% inward pull (standard for 1x1 rib) | 88% of body radius |

**Crown decrease profile** — the radius at each decrease level was calculated from the stitch count:

```
Radius = Stitches / (Gauge x 2 x pi)

88 sts -> R = 3.112"    80 sts -> R = 2.829"    70 sts -> R = 2.476"
60 sts -> R = 2.122"    50 sts -> R = 1.768"    40 sts -> R = 1.415"
30 sts -> R = 1.061"    20 sts -> R = 0.707"    10 sts -> R = 0.354"
```

The crown uses a **dome curve** rather than a linear cone. The decrease radii above are exact (from the pattern), but the height at each level follows a hemisphere arc instead of a straight line. This matches how real knitted fabric drapes — it doesn't form a perfect cone.

The dome mapping uses:
```
cos(theta) = stitch_radius / body_radius
height = crown_base + crown_height x sin(theta)
```

### Step 3: Build the 3D Model in Blender

**Tool:** Blender 4.x via the [Blender MCP server](https://github.com/ahujasid/blender-mcp) (Model Context Protocol), which allows Claude Code to execute Python code directly in a running Blender instance.

The model was built programmatically using bmesh (Blender's mesh editing API):

1. **Created profile points** — a series of (radius, height) pairs defining the beanie's cross-section:
   - Rib zone: contracted to 88% of body radius (simulates rib elasticity)
   - Transition: rib expands back to full stockinette radius
   - Body: cylindrical at full radius with very slight barrel (0.8% outward at center — knit fabric relaxes slightly)
   - Crown: radii from the decrease schedule mapped onto a dome curve

2. **Revolved the profile** — each profile point was swept around 360 degrees using **88 segments**, matching the actual cast-on stitch count. This means every vertical edge in the mesh represents one stitch column.

3. **Connected the rings** — quad faces were created between adjacent profile rings, with triangle fans at the top gather point.

4. **Applied subdivision surface** (level 2 viewport, level 3 render) for smooth curvature.

5. **UV unwrapped** using cylindrical projection for proper texture mapping.

**Key detail:** Using 88 mesh segments (not a round number like 64 or 128) was deliberate — it matches the pattern's cast-on count so the wireframe grid literally represents the stitch structure.

### Step 4: Apply the Wireframe Material

For the technical drawing look, we used two materials:

1. **Fill material** — a flat diffuse shader in light blue-gray (#dce0f0) with full roughness, giving the flat CAD-like fill
2. **Wireframe overlay** — a Blender Wireframe modifier using an emissive material in dark navy (#1e2d59), thickness 0.03 units, applied as a second material slot

This combination produces the characteristic technical drawing appearance: light fill with dark construction lines showing the mesh topology.

### Step 5: Render the Technical Views

We set up an orthographic camera (no perspective distortion, like a real engineering drawing) and rendered 5 views:

| View | Camera Position | Purpose |
|------|----------------|---------|
| Front elevation | (0, -30, 4.5) | Shows dome profile, rib contraction |
| Side elevation | (30, 0, 4.5) | Same as front (radially symmetric) |
| 3/4 perspective | (14, -12, 10) | Shows 3D form and wireframe curvature |
| Top / plan view | (0, 0, 30) | Shows circular cross-section, stitch columns converging |
| Back elevation | (0, 30, 4.5) | Confirms symmetry |

**Render settings:**
- Engine: Cycles (GPU)
- Samples: 64
- Resolution: 800 x 800 px
- Background: transparent (PNG alpha)
- Ortho scale: 12-14 (sized to frame full beanie with margin)
- Single sun light at 45 degrees for even illumination

### Step 6: Build the SVG Dimensions Diagram

The 2D schematic diagram on the reference page is a hand-coded SVG (not generated from Blender). It shows:

- The beanie silhouette as a path with a quadratic bezier curve for the dome crown
- A brim band rectangle with label
- Dimension lines with tick marks for width (10" flat / 19.6" circumference) and height (9")
- A dashed line marking where crown decreases begin
- Zone labels (STOCKINETTE body, CROWN) with stitch counts
- A gather point marker at the top

The SVG uses the same measurements calculated in Step 2.

### Step 7: Compose the Architecture Sheet

The technical drawing views are composed into an architectural presentation sheet inspired by [architectural elevation drawings](https://framerusercontent.com/images/sEznQOhtN0yIg2J7o6n0GNjSYKY.jpg). The sheet uses:

- **Layout:** CSS Grid, 3 columns x 2 rows for the views, plus a footer row
- **Background:** Layered CSS radial gradients simulating a warm watercolor parchment wash
- **Dimension annotations:** Tick marks and measurement lines under each view
- **Specification callout boxes:** Gauge, construction zones, and fit details
- **Title block:** Pattern name, size, cast-on count, construction method (bottom-right, like real architectural drawings)
- **Scale bar:** Reference ruler (bottom-left)
- **Grid cell borders:** Subtle separator lines between views

### Step 8: Build the Full Reference Page

The complete HTML page (`index.html`) combines everything into a single document:

1. Header with pattern name, difficulty badge, yarn/needle specs
2. Gauge box with stitches and rows per inch
3. Materials grid (yarn, needles, notions, construction method)
4. Size chart table — all 8 sizes with head circumference, finished circumference, height, and cast-on counts in both imperial and metric
5. SVG dimensions diagram
6. 3D technical drawing architecture sheet
7. Step-by-step instructions for Adult Size 7
8. Crown decrease schedule with visual stitch-count progress bars
9. 3D model specifications table (showing how every dimension was calculated)
10. Crown profile formula reference
11. Ribbing rounds by size
12. Abbreviations table

The page is self-contained HTML+CSS with no JavaScript dependencies. The only external assets are the 5 PNG renders from Blender.

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **Claude Code** (Claude Opus 4.6) | Research, pattern math, HTML/CSS authoring, Blender scripting, project coordination |
| **Blender 4.x** | 3D modeling and rendering |
| **Blender MCP Server** | Bridge between Claude Code and Blender (executes Python via Model Context Protocol) |
| **bmesh (Blender Python API)** | Programmatic mesh construction |
| **Cycles render engine** | GPU-accelerated rendering with wireframe overlay |
| **GitHub Pages** | Static site hosting |

No design tools (Figma, Photoshop, Illustrator) were used. The SVG diagram is hand-coded. The architecture sheet layout is pure CSS. The 3D model was built entirely through Python code executed in Blender.

---

## File Structure

```
knitting/
  index.html        # Complete pattern reference page (HTML + CSS + inline SVG)
  view-front.png     # Front elevation render (800x800, transparent bg)
  view-side.png      # Side elevation render
  view-34.png        # 3/4 perspective render
  view-top.png       # Top/plan view render
  view-back.png      # Back elevation render
  README.md          # This file
```

The Blender source file (`beanie-pattern-accurate.blend`) is stored locally at `C:\Users\gary\Downloads\Projects\Knitters\` but not committed to the repo due to size.

---

## Pattern Source

**Classic Stockinette Beanie** by [Craft Fix](https://www.craftfix.com/circular-knit-hat-pattern/)

- Free pattern, beginner level
- 8 sizes: Newborn, 3-6 months, 6-12 months, Toddler, Child, Teen, Adult, Adult XL
- Yarn: Worsted/Aran weight (#4)
- Needles: 5mm (US 8), 16" circular + DPNs for crown
- Gauge: 18 stitches x 24 rounds = 4" in stockinette
- Construction: Knit in the round, bottom-up

---

## What This Proves

This project demonstrates that knitting patterns contain enough mathematical information to generate accurate 3D models. The key insight is that **gauge is a coordinate system** — once you know stitches per inch and rows per inch, every stitch count in the pattern maps to a real-world measurement, and every measurement maps to a point in 3D space.

This could be extended to:
- Any knitting pattern with complete specifications
- Interactive 3D viewers (three.js, Sketchfab)
- Size comparison visualizations across the full size range
- Yarn quantity estimation from the 3D surface area
- Pattern validation (does the math actually produce the claimed finished measurements?)
