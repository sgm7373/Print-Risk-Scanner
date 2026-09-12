# Print Risk Scanner

A tool that looks at a 3D model before it gets printed and predicts where
on that model a print is likely to fail.

It works by slicing the mesh into thin horizontal layers, the same way a
3D printer would build the part up layer by layer, then checking each
layer against two things that are known to cause real print failures:

1. A sudden jump in cross section area from the layer right below it.
   A small layer suddenly having to support a much bigger one above it
   is a common cause of layers separating from each other.
2. Outward growth with nothing supporting it underneath. An unsupported
   overhang is one of the most common reasons a print sags or warps
   during the build.

Both checks are combined into a single risk score per layer, and that
score gets painted directly onto the 3D shape so you can see exactly
where the danger zones are before committing to a print.

## Why I built this

Most print failures do not show up randomly, they show up at specific
geometric features, a thin neck feeding into something wide, an arm
sticking out with no support below it, a sudden change in the shape of
the part. I wanted to see if those features could be caught ahead of
time with simple geometry checks instead of finding out after a failed
print.

## How it was tested

I built two test shapes with a known failure point built into each one
on purpose, so I would have a way to check whether the scanner actually
finds the right spot instead of just producing a number that looks
reasonable.

**Part one:** a wide base, a thin neck, and a wide cap sitting on top of
the neck. The scanner correctly flagged the exact layer where the neck
meets the cap, both from the area jump check and the overhang check
independently.

**Part two:** a tall post with an arm sticking out sideways partway up.
The scanner correctly flagged the layer where the arm starts, right
where it has nothing supporting it from below.

Same code, two different shapes, two different failure patterns, both
caught in the right place with no manual tuning between runs.

## What is in this repo

- `print_risk_scanner.ipynb`, the full notebook, runs start to finish in
  Google Colab with no setup beyond the first cell
- `test_part.stl`, the neck and overhang cap test geometry
- `cantilever_part.stl`, the cantilever arm test geometry

## How to run it

Open the notebook in Google Colab, run the cells in order from the top.
The notebook also includes a cell that lets you upload your own STL file
and run the same analysis on it.

## What it uses

Python, trimesh for mesh slicing and geometry, matplotlib for the 3D
visualizations, pandas for the summary table, networkx as a placeholder
for future topology based checks like detecting isolated thin features.

## Possible next steps

Testing against real printed parts to see how well the risk score
actually predicts failure rate, adding a thermal shrinkage estimate for
resin based printing, and turning the geometry checks into a proper
design of experiments study across orientation and support settings.
