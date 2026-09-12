# Print Risk Scanner

## Objective

Predict where a 3D printed part is likely to fail, before it is ever
printed, by analyzing the geometry alone. Print failures in layer based
manufacturing are not random, they tend to happen at specific geometric
features. This project encodes two of those known failure triggers into
a repeatable, automated check that runs on any mesh.

## Background

In layer based 3D printing, a part is built one thin horizontal slice at
a time. Two geometric conditions are well known causes of failed prints:

1. **Cross sectional area jump.** When one layer's cross section is
   much larger than the layer directly beneath it, the new layer is
   poorly supported and bonded to less material than it needs. This is
   a documented cause of layers separating from each other during or
   after printing.
2. **Unsupported overhang.** When a layer extends outward with no
   material underneath it, that section has nothing holding it in place
   during the build, which is a common cause of sagging and warping.

Both of these can be detected directly from mesh geometry without
running an actual print.

## Method

1. Load a 3D mesh (STL format) and confirm it is watertight, meaning it
   has no holes or gaps that would break a slicing operation.
2. Slice the mesh into evenly spaced horizontal layers, 0.5 millimeters
   apart, using ray intersection against the mesh surface, the same
   basic operation a printer's own slicer performs before generating a
   print path.
3. For each layer, compute:
   - cross sectional area, and the ratio of that area against the
     layer directly below it
   - the maximum radial distance from the layer's centroid to its
     outer edge, and how much that distance grows compared to the
     layer below
4. Normalize both signals to a zero to one scale and combine them into
   one composite risk score per layer, weighted equally.
5. Map the score back onto the full resolution mesh (after subdividing
   it for finer detail) and render it as a color heatmap, red for high
   risk, green for low risk.

## Results

Two synthetic parts were built with a known, deliberate failure point
each, so the scanner's output could be checked against ground truth
rather than trusted blindly.

| Test part | Total layers | Peak risk height (mm) | Peak risk score | Layers above 0.5 risk |
|---|---|---|---|---|
| Neck to cap overhang part | 65 | 25.0 | 1.0 | 1 |
| Cantilever arm part | 59 | 24.0 | 0.5 | 0 |

**Part one** had a wide base, a thin neck, and a wide cap sitting on top
of the neck starting at height 25mm. The scanner's highest risk layer
landed at exactly 25.0mm, matching the known failure point, and both
the area jump signal and the overhang signal independently agreed on
that location.

**Part two** had a tall post with an arm beginning to extend sideways
at height 24mm. The scanner's highest risk layer landed at 24.0mm,
again matching the known failure point.

Two different shapes, two different failure patterns, same code, no
manual tuning between runs, both caught at the correct location.

## Deliverables

- `print_risk_scanner.ipynb`, the complete notebook, runs top to bottom
  in Google Colab with no external setup
- `test_part.stl` and `cantilever_part.stl`, the two validation
  geometries referenced in the results above
- A reusable `analyze_print_risk()` function that takes any mesh and
  returns per layer risk data, ready to run against new geometry
- An upload cell in the notebook for testing arbitrary STL files beyond
  the two included here

## Why this is useful

Catching a likely failure point before printing means less wasted
material, less wasted machine time, and faster iteration on part
design or orientation. A tool like this could sit ahead of a print
queue as an automatic check, or be used by a designer during CAD work
to catch problem geometry before it ever reaches a printer.

## How to run it

Open `print_risk_scanner.ipynb` in Google Colab and run the cells in
order from the top. The final cells let you upload your own STL and
run the same analysis on it.

## Built with

Python, trimesh for mesh loading and slicing, matplotlib for the 3D
risk visualizations, pandas for the summary table, networkx included
as groundwork for future topology based checks, such as automatically
detecting thin necks or isolated islands using graph connectivity
rather than fixed thresholds.

## Possible next steps

Testing against real printed parts to see how well the composite risk
score actually predicts observed failure rate, adding a thermal
shrinkage estimate to catch warping caused by uneven curing rather than
just geometry, and running a proper design of experiments across print
orientation and support density to see which variables move the risk
score the most.

## - Built by Sourabh More
