SVG → Creality Plotter (svg2gcode Logic)

One standalone HTML file (svg2gcode-creality.html) that converts an SVG into G-code for a Creality/Ender printer equipped with a pen. You only need a web browser: no installation, no internet connection, and nothing is uploaded.

Quick Start

Open svg2gcode-creality.html in Chrome, Edge, or Firefox.

Select your printer model. The bed size is filled in automatically, but you can change the dimensions manually. Set the margin.

Drag and drop an .svg file into the upload area. The drawing is automatically scaled to the largest possible size within the margin.

Choose Lines, Fill, or Both.

Configure the pen settings: Pen Up Z and Pen Down Z.

Check the preview. If it shows ✓ Within margin, everything fits correctly.

Download the .gcode file or the ZIP package. The ZIP contains both the generated G-code and the original SVG so they always stay together.

Copy the file to your SD card and start it on the printer.

**Modes**

**Mode**	What happensLines	Draws only the SVG outlines. Useful for coloring pages that you want to fill in by hand later.

**Fill**	Generates only hatch shading (parallel lines) inside shapes.

**Both**	Draws the fill first, then inserts a pause (M0 Change Pen), then draws the outlines. Change pens and resume printing on the printer.

**Fill Settings**

Line spacing and angle of the hatch lines.

Fill inside: Automatic uses filled shapes if the SVG contains them, otherwise closed outlines. You can also force a specific behavior.

Fill nested shapes too: Off = shapes inside other shapes become holes (for example, an eye inside a face); On = all enclosed shapes are filled.
White fills are treated as empty space, so a white background is not shaded.
Advanced Settings
Curve tolerance (0.05 mm): Maximum deviation when flattening curves. Smaller values produce smoother curves but generate larger G-code files.
SVG DPI (96): Used for SVG files based on pixel units.
Travel speed: Speed of pen-up movements.
Smart ordering: Reduces unnecessary travel moves between paths.
Show travel moves: Displays pen-up movements in the preview.
What Happens to the SVG

The conversion follows svg2gcode-style logic:

The entire SVG is loaded and preserved as a single drawing. All relative positions, sizes, and proportions remain unchanged. Only the outer SVG receives scaling (viewBox, width, height).

Supported elements:

path
rect (including rounded corners)
circle
ellipse
line
polyline
polygon

Also supported:

transform (matrix, translate, scale, rotate, skewX, skewY)
transform-origin
<g>
<use> with <defs>
Nested <svg> elements
CSS classes defined in <style>
Inline style attributes
display:none

All path commands are supported: M L H V C S Q T A Z, both absolute and relative. Arcs are converted into exact cubic curves, not rough straight-line approximations.

Units (mm, cm, in, pt, pc, px) are converted to millimeters. The Y-axis is flipped so that (0,0) is in the lower-left corner, matching printer coordinates.

The drawing is scaled and positioned on the print bed.

Path order can optionally be optimized (nearest-neighbor + 2-opt).

Lines and/or hatching are converted into G-code using:

G21
G90
G28
Pen up/down via G0/G1 Z
Drawing via G1
Styling Rules
stroke creates drawable lines.
A shape with only a fill generates an outline (for Lines) and/or hatching (for Fill).
fill="none" without a stroke is invisible and skipped.

This follows standard SVG rendering rules.

Unsupported Features

You will receive a warning in the debug window if these are encountered:

<text> (convert text to paths first, e.g. Inkscape → Object to Path)
<image>
clip-path
mask
Pattern fills
Why No G2/G3 Arcs?

Most standard Ender firmware versions do not reliably support G2/G3 arc commands, so all curves are converted into short line segments instead.

Safety
G28 homes the Z-axis as well. With a pen holder installed, the pen may hit the build plate if your Z-offset is incorrect. Always test with a small drawing first.
Set Pen Down Z so the pen just touches the paper. Z=0 is determined by G28.
If the drawing extends outside the printable area, the tool asks for confirmation before allowing download.
Always verify the preview and dimensions before running the file.
Debug

Below the preview is the Processing (Debug) panel.

It displays:

Imported SVG information
Number of paths detected
Number of hatch lines generated
Travel distance before and after optimization
All warnings and unsupported-feature notices
Testing

Tested in a headless Chromium environment:

svg2gcode Reference Comparison
Compared against 10 of svg2gcode's own test files.
Same number of paths, matching order, and identical start/end points.
Maximum deviation: 0.007 mm.
Browser Reference Testing
Shapes with transformations, arcs, <use>, and nested <svg> elements were compared against actual browser rendering.
Generated geometry matched browser-rendered positions.
Fill Algorithm

Filled area correctness verified on:

Circles
Squares with holes
Even-odd fill rules
Non-zero fill rules

Filled area is typically 3–6% smaller than the theoretical area due to the 0.3 mm boundary margin.

G-code Validation

All three modes generate valid G-code:

Within print-bed boundaries
No drawing while the pen is lifted
ZIP export validated

All 13 sample SVG files from svg2gcode processed in under one second.

Not tested on a physical printer.

Known Limitations
The 2-opt optimizer is a simplified version of svg2gcode's tabu-search optimizer.
Above 3,000 paths: nearest-neighbor only.
Above 12,000 paths: no optimization.
Filling uses a scanline hatch algorithm.
Each hatch line is drawn individually with pen-up moves between them.
Nested <svg> elements are not clipped to their own viewport.
<symbol> elements with a viewBox are not scaled independently.
