# Geometry Culling with GJK-Inspired Bounds

This document describes a conservative culling technique for spatial queries, visualized using a unit square as the reference shape.

**Interactive version**: [Desmos geometry link](https://www.desmos.com/geometry/089j0ubebw) — drag point C to explore how the bounds change.

## Setup

![Culling visualization with correct √2 bound](images/culling-correct.png)

**Components:**

- **The square**: A unit square with vertices at (±0.5, ±0.5), centered at the origin
- **Point C**: The closest point to the origin (center of the square) found among *other objects* being tested — not a point on the square itself. In this example, C = (−0.674, 0.173), which lies outside the square.
- **Inner limit**: A circle of radius |C| centered at the origin
- **Outer limit**: A circle of radius |C| + √2 centered at the origin, where √2 is the full diagonal of the unit square (corner to opposite corner)

## Culling Logic

The inner and outer circles enable fast accept/reject decisions when testing whether other objects might contain points closer to the center than C:

| Test | Condition | Conclusion |
|------|-----------|------------|
| **Inner** | *Any* point of an object falls inside the inner circle | That point is closer to the center than C — a better candidate exists |
| **Outer** | The *entire* object lies outside the outer circle | No point on that object can be closer to the center than C — safe to reject |

Note the asymmetry:
- The inner test requires only partial intersection (one qualifying point suffices)
- The outer test requires complete exclusion (the entire object must be outside)

## The Uncertain Region (Blue)

The blue shaded region represents locations where an external object might be closer to some non-center point inside the square than to the center itself. Objects in this region cannot be trivially accepted or rejected — they require more detailed testing.

This is where the GJK-inspired support function calculations come in, using dot products against the square's vertices to determine tighter bounds.

## Why the Full Diagonal?

The outer limit uses √2 (the full diagonal), not √2/2 (the half-diagonal). This is critical for correctness.

![Incorrect culling with √2/2 bound](images/culling-incorrect.png)

When using the half-diagonal (√2/2 ≈ 0.707), the outer circle becomes too small. The blue "uncertain" region extends beyond the orange boundary, meaning objects in that overflow area would be incorrectly rejected despite potentially containing closer points.

The full diagonal √2 accounts for the maximum extent of the square — the distance from one corner to the opposite corner. This guarantees that any object entirely outside the outer circle truly cannot contain a point closer to the center than C.

## Caveats

1. **Shape-dependent bounds**: The √2 factor is specific to a unit square. Other convex shapes would require their own maximum extent calculation (e.g., diameter of a bounding circle, or maximum distance between any two points on the shape).

2. **Conservative, not tight**: These circular bounds are intentionally loose. The actual region where closer points might exist (the blue area) has a more complex shape determined by the support function calculations. The circles provide cheap early-out tests before falling back to more expensive exact tests.

3. **Reference point matters**: C must be the closest point to the center among all objects considered so far. If C is misidentified or measured from the wrong reference, the entire culling logic breaks down.

4. **Center of the square as origin**: This visualization assumes the square is centered at the origin, which is also the reference point for all distance calculations. Translating the square would require adjusting the coordinate system accordingly.
