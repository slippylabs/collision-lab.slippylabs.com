# Collision Lab

A 2D collision sandbox — SAT with minimum translation vector, GJK and EPA, ray casts, and swept AABB with the tunnelling case shown — draggable, with code you can paste into your engine. Runs entirely in your browser.

**Live:** <https://collision-lab.slippylabs.com/>

## What it does

- Drag two convex shapes (box, circle, triangle, pentagon, hexagon, any of them rotated) into each other.
- **SAT** with the minimum translation vector, drawn as a ghost of the pushed-out shape, and the projections shown one axis per row.
- **GJK + EPA**, with the simplex drawn as it is built and the Minkowski difference optionally shown beside it.
- **Ray casts** with the surface normal and the reflected ray.
- **Swept AABB**, with the discrete once-per-frame test visibly failing on a fast bullet.
- Export the SAT and swept routines in JavaScript or C#.

## How it works

SAT and GJK answer the same question from opposite ends — projections versus the Minkowski difference — so the page runs **both on every frame** and prints whether they agree. Two independent routes to one answer is the only self-check available in a browser, and it is shown to the reader rather than hidden.

Three things in here are the bugs this kind of code usually ships with:

**The MTV is not the width of the overlapping interval.** `min(aMax,bMax) − max(aMin,bMin)` is the right push only when two projections *partly* overlap. When one is contained in the other — a small shape deep inside a big one, exactly the case a physics step has to recover from — that quantity is the small shape's own width, and moving by it leaves the shapes still overlapping. The distance that clears the axis is `min(aMax − bMin, bMax − aMin)`.

**A sweep cannot answer for boxes that start overlapping.** Both entry times come out negative and the textbook version returns "no collision", which is a silent lie to a caller whose objects are interpenetrating. This one reports the contact at t = 0 with a flag that says "separate these", because an entry normal does not exist for a contact that already happened.

**A circle's ray normal must face the ray, like a polygon's.** A ray fired from inside a circle exits through a surface whose outward normal points *along* the ray; handing that back would have the two shape types disagree about what "the normal" means in the one case where it matters.

## Verification

The oracle for "do these two convex shapes overlap" is the **definition**, not another routine. `verify_collision.py` builds the Minkowski difference explicitly (every pairwise difference, convex-hulled by `scipy.spatial.ConvexHull`) and measures the distance from the origin; circles are solved analytically.

- **1,300 convex pairs**: SAT and GJK both agree with the exact answer and with each other. SAT's MTV length matches the exact penetration to **1.6e-14**, EPA's to **9.1e-08**.
- **Minimality**: on 85 penetrating pairs, the MTV separates the shapes and 97% of it does not — so it is the smallest push, not merely a sufficient one.
- **1,200 ray casts** against exact segment and quadratic intersection: worst |t| error **8.5e-14**, every normal unit length and facing the ray.
- **Swept AABB** against 40,000-substep marching: worst |t| error **2.4e-05**. 43 of the sweeps are caught *only* by the sweep — a once-per-frame test misses them at both ends of the frame.

**7,576 checks.**
