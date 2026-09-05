# Simplified Stiffened Panel Analysis Tool

## Self-Contained MVP Scope and Engineering Basis

This document consolidates the complete project scope and embeds the engineering equations needed to understand and implement the tool without access to the original course PDF.

> **Engineering limitation:** This is a simplified linear-elastic pre-buckling model. The application must not be treated as a certification-level solver without independent verification, validated curve data, test cases, and engineering review.

---

## 1. Project objective

Build a graphical engineering tool for simplified analysis of flat stiffened panels. The engineer draws the panel layout, defines dimensions, material, thickness, supports, external nodal loads, and the applicable buckling curve for every bay. The software then:

1. detects the panel bays and connectivity;
2. solves the global shear-flow distribution;
3. plots the normal-load diagram for a selected boundary/stiffener;
4. evaluates elastic shear buckling for each rectangular bay;
5. calculates a bay-specific critical external load when a unique scalar solution exists;
6. exports a full engineering PDF report.

A later phase may support trade-off studies, but optimization is excluded from the MVP.

---

## 2. Product identity

The product is a **simplified aircraft-style panel analysis workbench**.

It is not:

- a general finite element solver;
- a mesh generator;
- a nonlinear solver;
- a detailed stress-recovery tool;
- a stiffener sizing or crippling tool;
- a post-buckling solver.

The intended experience is: “draw a conceptual panel and obtain transparent analytical results.”

---

## 3. Structural idealization

### 3.1 Skin

The skin is the thin sheet forming each rectangular bay. In the MVP, the skin carries the shear flow.

### 3.2 Boundaries or stiffeners

Every line drawn around or inside the panel is treated as a structural boundary/stiffener. These lines subdivide the skin into bays.

MVP stiffeners are geometric entities only. They do not have:

- area `A`;
- moment of inertia `I`;
- section shape;
- separate material;
- Euler buckling calculation;
- crippling calculation.

### 3.3 Analytical philosophy

The agreed concept is a simplified boom-skin style idealization:

- the skin carries shear flow;
- global equilibrium determines load transfer;
- normal forces along boundaries are reconstructed from equilibrium with the adjacent shear flows;
- the boundaries are assumed sufficiently rigid for the pre-buckling idealization.

---

## 4. Geometry input

### 4.1 What the user draws

The user draws only:

- the outer boundary of the complete panel;
- internal horizontal or vertical lines that subdivide the panel.

The user does not manually create nodes, connectivity tables, stiffener entities, or bay entities.

### 4.2 Automatic detection

The software automatically:

- creates vertices at line endpoints and intersections;
- detects connectivity;
- identifies outer-boundary and internal vertices;
- treats every line as a boundary/stiffener;
- detects every closed rectangular bay;
- numbers bays automatically as `Bay 1`, `Bay 2`, and so on.

### 4.3 Geometry restrictions

The MVP permits:

- a single connected structure;
- rectangular or square bays;
- horizontal and vertical boundaries only;
- closed bays only.

The MVP rejects:

- multiple disconnected structures;
- open bays;
- inclined boundaries;
- trapezoidal or skewed bays;
- curved boundaries;
- arbitrary quadrilaterals.

### 4.4 Dimensions

For every detected bay, the user enters:

- width;
- height.

The dimensions are not inferred from how large the sketch appears on screen.

For stability calculations, define:

\[
a_i=\max(\text{width}_i,\text{height}_i)
\]

\[
b_i=\min(\text{width}_i,\text{height}_i)
\]

and:

\[
\boxed{r_i=\frac{a_i}{b_i}\geq 1}
\]

Thus, `a` is the longer bay dimension and `b` is the shorter bay dimension.

### 4.5 Naming

Bay names are automatic only. Custom names are outside the MVP.

---

## 5. Material and thickness

### 5.1 Material

The entire panel uses one global, isotropic, linear-elastic material.

The user can:

- select from a material library; or
- create a custom material.

Required properties:

- Young's modulus `E`;
- Poisson's ratio `nu`.

The selected buckling-curve convention determines whether `nu` appears explicitly in the final equation.

### 5.2 Thickness

One global skin thickness `t` applies to all bays.

Different thicknesses by bay and tapered skins are outside the MVP.

---

## 6. Supports

Available support types:

- fixed;
- pinned;
- roller.

Supports may be applied only at vertices on the outer boundary of the complete panel.

Supports are prohibited at internal intersections.

### Hard rule: load-support exclusivity

A vertex cannot contain both an external load and any support type. This includes fixed, pinned, and roller supports.

Suggested error:

> **Validation error:** A vertex cannot simultaneously contain a support and an external load. Remove the load or the support.

---

## 7. External loads

### 7.1 Number of loads

The user may define any number of external loads.

### 7.2 Location

Loads may be applied only at vertices on the outer boundary of the complete panel.

Loads are prohibited:

- at internal intersections;
- in the middle of a boundary/stiffener;
- inside a bay;
- at any support vertex.

### 7.3 Definition

The user defines:

- load magnitude;
- tension or compression;
- horizontal or vertical orientation when both are geometrically possible at a corner.

The load is normal to the selected outer boundary.

- Compression points inward toward the panel.
- Tension points outward away from the panel.

### 7.4 Excluded load types

The MVP excludes:

- distributed loads;
- moments;
- arbitrary angled loads;
- loads along a segment;
- internal nodal loads.

---

## 8. Units and symbols

Recommended internal unit system:

- force: N;
- length: mm;
- stress and modulus: MPa = N/mm^2;
- shear flow: N/mm.

| Symbol | Meaning | Unit |
|---|---|---|
| `a_i` | Longer dimension of Bay i | mm |
| `b_i` | Shorter dimension of Bay i | mm |
| `r_i` | Aspect ratio `a_i/b_i` | dimensionless |
| `t` | Global skin thickness | mm |
| `E` | Young's modulus | MPa |
| `nu` | Poisson's ratio | dimensionless |
| `K_i` | Buckling coefficient | dimensionless |
| `q_i` | Shear flow in Bay i | N/mm |
| `tau_i` | Actual shear stress | MPa |
| `tau_cr,i` | Critical elastic shear-buckling stress | MPa |
| `U_i` | Buckling utilization | dimensionless |
| `RF_i` | Buckling reserve factor | dimensionless |
| `P_j` | External nodal load | N |
| `P_cr,i` | Bay-specific critical load | N |
| `N_s(x)` | Normal load along selected boundary | N |

Unit check:

\[
\frac{q}{t}=\frac{\mathrm{N/mm}}{\mathrm{mm}}=\mathrm{N/mm^2}=\mathrm{MPa}
\]

---

## 9. Global shear-flow analysis

### 9.1 Global rather than local solution

The tool solves shear flow using the complete connected structure, all supports, and all loads. A load may influence bays that do not directly touch the loaded vertex because equilibrium transfers actions through the entire structure.

### 9.2 Linear form

For Bay `i`, the shear-flow response can be represented as:

\[
q_i=f_i(P_1,P_2,\ldots,P_n)
\]

Under the linear-elastic assumptions:

\[
\boxed{q_i=\alpha_{i1}P_1+\alpha_{i2}P_2+\cdots+\alpha_{in}P_n}
\]

The response coefficients depend on geometry, supports, connectivity, load positions, and load directions.

### 9.3 System of equations

The automatic solver builds an equilibrium system, generically:

\[
\mathbf{Aq}=\mathbf{b}
\]

where:

- `q` contains the unknown shear-flow values;
- `A` represents the equilibrium relationships;
- `b` represents the applied loads and reactions after the chosen formulation is assembled.

The exact row construction must be validated against known hand calculations. The solver must detect singular, inconsistent, and underdetermined systems.

### 9.4 Automatic and detailed modes

Automatic mode builds and solves the equations.

Optional detailed-calculation mode shows:

- sign conventions;
- equilibrium equations;
- intermediate substitutions;
- support reactions;
- solved shear flows;
- residual checks;
- equations used to build normal-load diagrams.

Suggested control:

```text
[ ] Show detailed calculations
```

### 9.5 Visualization

Display:

- signed numerical `q` values;
- arrows showing flow direction;
- a legend defining positive and negative flow.

---

## 10. Actual shear stress

For every bay:

\[
\boxed{\tau_i=\frac{q_i}{t}}
\]

For comparison with a positive critical magnitude:

\[
\boxed{|\tau_i|=\frac{|q_i|}{t}}
\]

The sign of `q_i` remains visible because the sign indicates direction.

### Physical basis

A pure shear state corresponds to principal normal stresses at 45 degrees. One principal stress is tensile and the other is compressive, with magnitudes equal to the shear-stress magnitude. The compressive principal stress drives the plate-buckling behavior. Therefore, the simplified stability check compares `|q|/t` with a critical shear-buckling stress.

---

## 11. Relevant elastic stability theory

### 11.1 Background from Euler stability

Elastic buckling of a slender member depends on material stiffness, geometry or slenderness, and support conditions. For thin plates, the same qualitative dependencies remain, but two-dimensional plate behavior and Poisson coupling modify the critical-stress expression.

The model is valid for elastic buckling. It does not represent yielding, crushing, plastic buckling, or material failure.

### 11.2 General thin-plate convention

A commonly used critical plate stress format is:

\[
\tau_{cr}=k_s\frac{\pi^2E}{12(1-\nu^2)}\left(\frac{t}{b}\right)^2
\]

However, some charts redefine the plotted coefficient so that the constants are incorporated into `K`.

### 11.3 Equation convention used for this project

The relevant course material presents the shear-buckling equation in the compact form:

\[
\boxed{\tau_{cr,i}=K_iE\left(\frac{t}{b_i}\right)^2}
\]

For this project, `K_i` must be the coefficient associated with this exact equation convention.

**Critical implementation rule:** Do not multiply by both the chart coefficient `K` and an extra factor such as `pi^2/[12(1-nu^2)]` unless the curve dataset explicitly uses the alternative `k_s` convention. Otherwise the normalization would be counted twice.

Every embedded curve dataset must state its equation convention.

Dimensional check:

\[
[K E(t/b)^2]=[E]=\text{stress}
\]

---

## 12. K versus a/b curves

### 12.1 Purpose

The chart supplies the buckling coefficient as a function of bay aspect ratio:

\[
\boxed{K=K(a/b)}
\]

The reference contains four curves representing four typical edge-restraint cases. Greater edge restraint generally produces higher elastic buckling resistance.

### 12.2 User interaction

For every bay, the user selects one applicable curve from a dropdown.

The stability screen contains:

**Left side**

- Bay dropdown;
- Curve 1, 2, 3, or 4 dropdown;
- explanation of the selected curve;
- width and height;
- normalized `a` and `b`;
- calculated `a/b`;
- interpolated `K`.

**Right side**

- the complete `K` versus `a/b` graph;
- all four curves;
- selected curve highlighted;
- vertical marker at the current `a/b`;
- interpolation point;
- calculated `K`.

### 12.3 Required embedded data

Because the user will not have the PDF, the software package must contain:

- the meaning of each curve;
- digitized `(a/b, K)` coordinates;
- valid range;
- equation convention;
- interpolation rule;
- source/verification note.

The 100-point approximate digitized datasets are embedded in Section 31. Their numerical values are suitable for approximate curve interpolation. The exact physical edge-restraint descriptions of K1 through K4 must still be independently verified before engineering release.

### 12.4 Linear interpolation

Between points `(r1,K1)` and `(r2,K2)`:

\[
\boxed{K(r)=K_1+\frac{r-r_1}{r_2-r_1}(K_2-K_1)}
\]

Rules:

1. Return exact tabulated values at exact points.
2. Show the two interpolation points in detailed mode.
3. Do not silently extrapolate.
4. Block or clearly warn when `a/b` is outside the verified curve range.
5. Plot the interpolation point.

Suggested curve data structure:

```yaml
curve_id: 1
display_name: verified edge-restraint description
equation_convention: tau_cr = K * E * (t/b)^2
x_variable: a_over_b
y_variable: K
valid_range:
  minimum: verified_value
  maximum: verified_value
points:
  - [ratio_1, K_1]
  - [ratio_2, K_2]
verification_note: independently checked digitization
```

---

## 13. Bay stability calculation

For each Bay `i`:

### Step 1: Normalize dimensions

\[
a_i=\max(h_i,w_i),\qquad b_i=\min(h_i,w_i)
\]

### Step 2: Calculate aspect ratio

\[
r_i=\frac{a_i}{b_i}
\]

### Step 3: Interpolate coefficient

\[
K_i=K_{\text{selected curve}}(r_i)
\]

### Step 4: Calculate critical shear-buckling stress

\[
\boxed{\tau_{cr,i}=K_iE\left(\frac{t}{b_i}\right)^2}
\]

### Step 5: Obtain global shear flow

\[
q_i=f_i(P_1,\ldots,P_n)
\]

### Step 6: Calculate actual shear stress

\[
\boxed{\tau_i=\frac{q_i}{t}}
\]

### Step 7: Calculate utilization

\[
\boxed{U_i=\frac{|\tau_i|}{\tau_{cr,i}}}
\]

Interpretation:

- `U_i < 1`: below the idealized elastic buckling limit;
- `U_i = 1`: at the idealized critical condition;
- `U_i > 1`: above the idealized elastic buckling limit.

### Step 8: Calculate reserve factor

\[
\boxed{RF_i=\frac{\tau_{cr,i}}{|\tau_i|}=\frac{1}{U_i}}
\]

If `q_i=0`, show `Not governing under current load` rather than infinity.

The UI should say “below critical,” “critical,” or “above critical,” not simply “safe” or “failed,” because this simplified calculation does not include all real failure mechanisms.

---

## 14. Load-to-bay association

A load is associated with a bay when the load is applied at an outer-boundary vertex belonging to that bay.

If a loaded vertex belongs to two adjacent bays, the load is associated with both bays. The two bays may produce different critical-load values because the bays may have different:

- shear flow;
- dimensions;
- aspect ratio;
- selected curve;
- `K` coefficient;
- critical stress.

This geometric association rule determines whether a unique scalar critical-load calculation is allowed. It does not limit the global influence of a load in the shear-flow solution.

---

## 15. Maximum external load for one bay

### 15.1 Eligibility

The program calculates a scalar maximum external load for the selected bay only if exactly one external load is associated with that bay.

### 15.2 General linear relation

For one associated variable load `P`:

\[
\boxed{q_i(P)=\alpha_iP+q_{i,0}}
\]

where:

- `alpha_i` is the response coefficient for the selected load;
- `q_i,0` is the shear-flow contribution from any other globally acting loads not geometrically associated with the bay.

The critical condition is:

\[
\boxed{\left|\frac{\alpha_iP_{cr,i}+q_{i,0}}{t}\right|=\tau_{cr,i}}
\]

or:

\[
|\alpha_iP_{cr,i}+q_{i,0}|=t\tau_{cr,i}
\]

Candidate roots are:

\[
P_{cr,i}^{(+)}=\frac{t\tau_{cr,i}-q_{i,0}}{\alpha_i}
\]

\[
P_{cr,i}^{(-)}=\frac{-t\tau_{cr,i}-q_{i,0}}{\alpha_i}
\]

Retain only the smallest positive root consistent with the user-defined tension/compression sense and loading direction.

### 15.3 Proportional single-load case

If `q_i,0=0`:

\[
q_i=\alpha_iP
\]

and:

\[
\boxed{P_{cr,i}=\frac{t\tau_{cr,i}}{|\alpha_i|}}
\]

Substituting the critical stress:

\[
\boxed{P_{cr,i}=\frac{K_iEt^3}{|\alpha_i|b_i^2}}
\]

This expression shows the simplified sensitivity:

- proportional to `K`;
- proportional to `E`;
- proportional to `t^3`;
- inversely proportional to `b^2`;
- inversely proportional to the load-to-shear-flow coefficient magnitude.

### 15.4 Zero influence

If `alpha_i=0`, do not divide by zero or report infinite capacity. Display:

> The selected external load has zero calculated influence on the shear flow of this bay under the current model.

### 15.5 Multiple associated loads

If more than one load is associated with the selected bay:

\[
q_i=\sum_j\alpha_{ij}P_j
\]

The critical boundary is:

\[
\boxed{\left|\sum_j\alpha_{ij}P_j\right|=t\tau_{cr,i}}
\]

This is a load-combination relationship, not a unique value for each load. Therefore, the MVP does not calculate individual maximum loads.

It still reports:

- current `q_i`;
- current `tau_i`;
- `tau_cr,i`;
- utilization;
- reserve factor;
- an explanation that the individual maximum loads are not unique.

Suggested message:

> **Critical load calculation unavailable:** More than one external load is associated with this bay. The current stress, critical stress, utilization, and reserve factor are still reported.

Multiple loads applied to different bays are allowed. The uniqueness rule is evaluated independently for the selected bay.

---

## 16. Normal-load diagrams

### 16.1 User selection

After solving, the user selects one boundary/stiffener. The tool plots the normal-load diagram only for the selected boundary.

### 16.2 Calculation basis

The diagram depends only on:

- geometry;
- supports;
- external loads;
- calculated shear flows;
- equilibrium.

It does not depend on stiffener area or inertia in the MVP.

### 16.3 Differential equilibrium

For local coordinate `x` along boundary `s`:

\[
\boxed{\frac{dN_s}{dx}=p_s(x)}
\]

where `p_s(x)` is the signed resultant line loading induced by adjacent shear flows.

For constant adjacent shear flows over a segment, a generic expression is:

\[
p_s=q_{\text{left}}-q_{\text{right}}
\]

with signs generated from the adopted arrow convention. Then:

\[
\boxed{N_s(x)=N_s(x_0)+p_s(x-x_0)}
\]

Thus, `N(x)` is linear over a region with constant shear flows.

At a permitted external load vertex, the diagram may have a jump equal to the signed load component along the selected boundary direction.

For consecutive pieces:

\[
N_s^{(k+1)}(x_k^+)=N_s^{(k)}(x_k^-)+\Delta N_k
\]

where `Delta N_k` is the signed concentrated action at the vertex.

### 16.4 Output

Display:

- local coordinate origin and direction;
- positive normal-load convention;
- every piecewise `N(x)` equation;
- key values;
- values immediately before and after jumps;
- maximum and minimum normal load;
- plotted engineering diagram;
- contributing shear-flow terms.

The sign of `q_left-q_right` must come from the actual orientation and equilibrium, not merely from visual left/right screen position.

---

## 17. Equilibrium and solver checks

### 17.1 Global forces

\[
\sum F_x=0
\]

\[
\sum F_y=0
\]

### 17.2 Global moments

\[
\sum M_O=0
\]

about a defined reference point `O`.

### 17.3 Local equilibrium

At relevant vertices and boundary segments, internal force resultants, support reactions, and applied loads must balance within tolerance.

### 17.4 Linear-system residual

For:

\[
\mathbf{Aq}=\mathbf{b}
\]

calculate:

\[
\mathbf{r}=\mathbf{Aq}-\mathbf{b}
\]

and a normalized residual such as:

\[
\epsilon_r=\frac{\|\mathbf{r}\|_2}{\max(\|\mathbf{b}\|_2,1)}
\]

Do not release results if the residual exceeds the validated tolerance.

### 17.5 Stability checks

Detect:

- insufficient or unstable supports;
- singular equations;
- inconsistent equations;
- underdetermined shear flows;
- missing dimensions;
- missing material properties;
- missing curve selection;
- aspect ratios outside curve range;
- zero thickness;
- nonpositive modulus;
- load-support conflict.

---

## 18. Stability user interface

The stability module is bay-centric.

The user chooses a bay from a dropdown. The geometry highlights that bay.

Always show for the selected bay:

- Bay ID;
- width and height;
- normalized `a` and `b`;
- `a/b`;
- selected curve;
- interpolated `K`;
- signed `q`;
- actual `tau=q/t`;
- critical `tau_cr`;
- utilization;
- reserve factor;
- critical external load when uniquely available.

Also show a compact table comparing all bays.

---

## 19. Results visualization

After pressing Solve, show:

- model geometry;
- Bay numbers;
- supports;
- load arrows;
- tension/compression sense;
- signed shear-flow values;
- shear-flow arrows;
- sign-convention legend;
- selected Bay highlight;
- utilization/criticality colors plus exact values;
- normal-load plot for a selected boundary;
- `K` curve graph and interpolation point.

Colors supplement numerical values. Colors do not replace numerical results.

---

## 20. Solve workflow

1. Draw one connected orthogonal panel layout.
2. Automatically detect vertices, boundaries, and Bays.
3. Enter width and height for every Bay.
4. Select or define the global material.
5. Enter global skin thickness.
6. Apply supports only to outer-boundary vertices.
7. Apply loads only to different outer-boundary vertices.
8. Select the buckling curve for every Bay.
9. Press **Solve**.
10. Validate geometry, supports, loads, properties, and equation solvability.
11. Solve reactions and global shear flow.
12. Check equilibrium residuals.
13. Plot shear flow.
14. Plot a selected normal-load diagram.
15. Calculate Bay stability results.
16. Calculate Bay-specific critical load only when unique.
17. Export the engineering report.

The software does not automatically re-solve while editing. An explicit Solve button is required.

---

## 21. Validation rules

### Geometry

Reject:

- disconnected geometry;
- open Bays;
- nonrectangular Bays;
- inclined or curved lines;
- invalid intersections;
- missing Bay dimensions.

### Loads

Reject:

- internal loads;
- loads at segment midpoints;
- loads inside Bays;
- loads on supports;
- arbitrary angles;
- distributed loads;
- moments;
- missing magnitude or tension/compression setting.

### Supports

Reject:

- internal supports;
- supports on loaded vertices;
- support arrangements that leave the model unstable.

### Properties

Reject or warn:

- `t <= 0`;
- `E <= 0`;
- invalid Poisson's ratio;
- Bay width or height `<= 0`;
- missing curve selection;
- `a/b` outside the verified curve range.

Do not silently extrapolate the curves.

---

## 22. Full PDF engineering report

The report includes:

### Model description

- model title;
- geometry;
- Bay numbering;
- width and height of every Bay;
- normalized `a` and `b`;
- material;
- `E` and `nu`;
- global thickness;
- supports;
- external loads;
- sign conventions.

### Shear-flow analysis

- equilibrium equations;
- reactions;
- shear-flow equations and results;
- diagram with arrows and values;
- residual check;
- intermediate steps when detailed calculations are enabled.

### Normal-load analysis

For the selected boundary:

- coordinate convention;
- contributing shear flows;
- piecewise `N(x)` equations;
- diagram;
- maximum and minimum values.

### Stability analysis

For every Bay:

- dimensions;
- `a/b`;
- selected curve and its meaning;
- interpolation points;
- interpolated `K`;
- `q`;
- `tau=q/t`;
- `tau_cr=K E (t/b)^2`;
- utilization;
- reserve factor;
- critical load when unique;
- explanation when unavailable.

### Curve chart

- all embedded curves;
- curve descriptions;
- selected curve;
- current aspect-ratio marker;
- interpolated point;
- equation convention.

### Conclusions

- selected Bay result;
- comparison of all Bays;
- highest current utilization;
- every uniquely calculated Bay-specific critical load;
- warnings and limitations.

---

## 23. Course topics intentionally excluded from MVP

### Compressed-plate buckling

The theory also treats plates under direct compression and explains the effect of edge restraint, Poisson coupling, and the number of buckling waves. The MVP focuses on shear flow and critical shear buckling.

### Curved panels

Curved panels can have different or higher buckling resistance than flat panels and require curvature-dependent terms. The MVP supports flat panels only.

### Post-buckling behavior

After shear buckling, stresses may redistribute and diagonal tension can develop. This changes loads in the surrounding reinforcements. The MVP stops at first elastic buckling and claims no post-buckling reserve.

### Stiffener stability

Real stiffeners can require Euler-column buckling checks and effective-length assumptions. The MVP has no stiffener area or inertia, so this is excluded.

### Stiffener deformability

Flexible reinforcements can alter the pure-shear model and the post-buckling diagonal-tension angle. The MVP assumes rigid idealized boundaries in the pre-buckling regime.

### Inelastic effects

Plastic buckling, crushing, yielding, and material failure are excluded.

---

## 24. Explicit MVP exclusions

- FEA and meshing;
- geometric or material nonlinearity;
- plasticity;
- contact;
- stress concentrations;
- imperfections;
- post-buckling reserve;
- diagonal-tension post-buckling calculation;
- stiffener area, inertia, and section shapes;
- stiffener Euler buckling;
- crippling;
- curved panels;
- variable thickness;
- multiple materials;
- arbitrary Bay shapes;
- interior loads;
- distributed loads;
- moments;
- arbitrary load angles;
- internal supports;
- load and support on the same vertex;
- disconnected structures;
- general multidimensional load envelopes;
- optimization;
- automatic design suggestions;
- custom Bay names;
- Excel export.

---

## 25. Possible Phase 2 features

- configuration trade-off studies;
- native project saving and reopening;
- variable thickness by Bay;
- material zones;
- stiffener section properties;
- Euler buckling of stiffeners;
- compression and shear interaction;
- post-buckling diagonal tension;
- curved panels;
- verified alternative curve families;
- automated curve selection from boundary conditions;
- multiple-load scaling and load envelopes;
- Excel export;
- optimization and design suggestions.

---

## 26. Required embedded data before release

To eliminate dependence on the external PDF, the deliverable must include:

1. this self-contained equation set;
2. symbol and unit definitions;
3. the verified meaning of every curve;
4. digitized `(a/b,K)` points for all four curves;
5. valid range for each curve;
6. equation convention for each curve;
7. interpolation and out-of-range rules;
8. shear-flow sign convention;
9. normal-force sign convention;
10. at least one verified complete example;
11. solver tolerances;
12. engineering limitations.

The approximate numerical curve points are embedded in Section 31. Because these values were digitized from an image, they must remain labeled as approximate and should be independently checked before engineering release.

---

## 27. Minimum acceptance tests

### Test A: aspect ratio

For width 400 mm and height 600 mm:

\[
a=600\ \mathrm{mm},\quad b=400\ \mathrm{mm},\quad a/b=1.5
\]

### Test B: interpolation

For tabulated points `(1.0,K1)` and `(2.0,K2)`, the result at `a/b=1.5` must equal the midpoint between `K1` and `K2`.

### Test C: shear stress

For:

\[
q=100\ \mathrm{N/mm},\quad t=2\ \mathrm{mm}
\]

verify:

\[
\tau=50\ \mathrm{MPa}
\]

### Test D: utilization

For:

\[
\tau=50\ \mathrm{MPa},\quad \tau_{cr}=100\ \mathrm{MPa}
\]

verify:

\[
U=0.5,\quad RF=2.0
\]

### Test E: one-load critical result

For `q=alpha P`, substitute the calculated critical load and verify:

\[
|q(P_{cr})|/t=\tau_{cr}
\]

within tolerance.

### Test F: multiple associated loads

Verify that the software provides current stress, critical stress, utilization, and reserve factor but does not report individual critical loads.

### Test G: load-support conflict

Verify that loads on fixed, pinned, or roller support vertices block the solve.

### Test H: global equilibrium

Verify force and moment equilibrium and the linear-system residual.

### Test I: normal-load diagram

For constant net line loading `p_s`, verify that `N(x)` is linear and that its slope equals `p_s`.

### Test J: curve convention

Verify that the program uses either:

\[
\tau_{cr}=K E(t/b)^2
\]

or the alternative explicit plate constant convention, but never multiplies both normalizations together.

---

## 28. Final self-contained calculation sequence

1. Validate connected orthogonal geometry.
2. Detect Bays, boundaries, and vertices.
3. Classify outer-boundary and internal vertices.
4. Receive every Bay's width and height.
5. Normalize `a` and `b`.
6. Receive global material and thickness.
7. Receive outer-boundary supports.
8. Receive outer-boundary loads.
9. reject all load-support overlaps.
10. Build the global equilibrium equations.
11. Solve support reactions and global shear flows.
12. Check force, moment, local, and residual equilibrium.
13. Display signed shear flows and directions.
14. Build the selected boundary's normal-load diagram.
15. Calculate every Bay's `a/b`.
16. Interpolate `K` from the Bay's selected embedded curve.
17. Calculate `tau_cr=K E (t/b)^2`.
18. Calculate `tau=q/t`.
19. Calculate utilization and reserve factor.
20. Determine the loads associated with the selected Bay.
21. If exactly one is associated, solve its critical magnitude.
22. If more than one is associated, suppress individual maximum-load results.
23. Show full selected-Bay details and all-Bay comparison.
24. Export the full PDF engineering report.

---

## 29. Engineering disclaimer for the application

> This tool performs a simplified linear-elastic analysis of flat, rectangular, thin-skinned stiffened panels. Results represent idealized pre-buckling shear flow, normal-load equilibrium, and first elastic shear-buckling estimates. The model does not include imperfections, yielding, plastic buckling, post-buckling strength, stiffener deformability, stiffener buckling, curved-panel effects, local stress concentrations, or certification factors. Engineering judgment and independent verification are required.

---

## 30. Frozen decisions log

- Existing-panel verification is the primary use.
- Trade-off studies are a future secondary use.
- Aircraft-style simplified panels are the target.
- User draws boundaries and internal divisions only.
- The software creates vertices, connectivity, stiffeners, and Bays automatically.
- One connected structure only.
- Rectangular/square orthogonal Bays only.
- User enters width and height for every Bay.
- Stiffeners are geometric lines only.
- Boom-skin style analytical idealization.
- Skin carries shear flow.
- One global material, library or custom.
- One global thickness.
- Multiple external loads allowed globally.
- Loads only at outer-boundary vertices.
- Loads defined by magnitude and tension/compression.
- Horizontal or vertical load direction only.
- Supports only at outer-boundary vertices.
- Fixed, pinned, and roller supports.
- A node can never contain both a support and a load.
- Global shear-flow solution.
- Automatic equations plus optional detailed-calculation mode.
- Shear-flow values, arrows, and sign legend.
- Normal-load diagram generated for a user-selected boundary.
- Stability analysis selected by Bay dropdown.
- Selected Bay highlighted.
- One buckling curve selected per Bay.
- Dropdown plus visible chart and curve explanation.
- `K` calculated from `a/b` by interpolation.
- Full selected-Bay results plus compact all-Bay comparison.
- Actual stress is `q/t`.
- Critical stress uses the curve-compatible equation `K E (t/b)^2`.
- Maximum external load calculated only for a Bay with exactly one associated load.
- Multiple associated loads suppress the scalar maximum-load result.
- Explicit Solve button.
- Full engineering PDF report.


---

## 31. Embedded Digitized K-Curve Dataset

### 31.1 Status and intended use

The following dataset contains **100 aspect-ratio points for each of the four curves**, for a total of **400 digitized K values**. The common aspect-ratio axis ranges from \(a/b=1.000\) to \(a/b=7.500\).

> **Important accuracy notice:** These values are approximate points digitized from a graph. They are not original tabulated values published by the source. They are suitable for approximate implementation and interpolation of the displayed curves, but they must not be represented as exact source data or used as the sole basis for certification-level structural design.

The four series have the following approximate behavior:

- **K1:** starts near 12.30 and approaches approximately 8.50.
- **K2:** starts near 10.80 and approaches approximately 8.45.
- **K3:** starts near 8.30 and approaches approximately 5.33.
- **K4:** starts near 7.95 and approaches approximately 5.05.

The curve names `K1` through `K4` remain provisional identifiers until their exact physical edge-restraint descriptions are independently verified.

### 31.2 Mandatory interpolation rule

For an input ratio \(r=a/b\) that exactly matches a tabulated ratio, the software returns the corresponding tabulated \(K\) value.

For any input ratio between two consecutive tabulated points \((r_j,K_j)\) and \((r_{j+1},K_{j+1})\), the software must use **piecewise linear interpolation** on the user-selected curve:

\[
oxed{
K(r)=K_j+rac{r-r_j}{r_{j+1}-r_j}\left(K_{j+1}-K_j
ight)
}
\]

This interpolation is performed independently for `K1`, `K2`, `K3`, or `K4`, according to the curve selected for the bay.

Rules:

1. The valid embedded-data interval is \(1.000\leq a/b\leq7.500\).
2. Exact tabulated inputs return exact stored values.
3. Inputs strictly between adjacent points use linear interpolation.
4. The software must not silently extrapolate below 1.000 or above 7.500.
5. An out-of-range value must produce a blocking validation message unless a future, separately validated extrapolation policy is introduced.
6. Detailed-calculation mode must show the selected curve, bounding points, interpolation fraction, and resulting \(K\).
7. Internal calculations should retain more precision than the displayed two-decimal K values when such precision exists in the stored dataset. With the present dataset, the stored K precision is two decimals.

Suggested out-of-range message:

> **Aspect ratio outside curve range:** The embedded digitized curves are valid only for \(1.000\leq a/b\leq7.500\). Revise the bay dimensions or provide a separately validated coefficient.

### 31.3 Dataset

| Point | a/b | K1 | K2 | K3 | K4 |
|---:|---:|---:|---:|---:|---:|
| 1 | 1.000 | 12.30 | 10.80 | 8.30 | 7.95 |
| 2 | 1.066 | 12.10 | 10.75 | 8.22 | 7.82 |
| 3 | 1.131 | 11.91 | 10.70 | 8.14 | 7.69 |
| 4 | 1.197 | 11.71 | 10.65 | 8.05 | 7.56 |
| 5 | 1.263 | 11.53 | 10.57 | 7.95 | 7.39 |
| 6 | 1.328 | 11.37 | 10.49 | 7.85 | 7.21 |
| 7 | 1.394 | 11.20 | 10.41 | 7.77 | 7.06 |
| 8 | 1.460 | 11.05 | 10.32 | 7.67 | 6.89 |
| 9 | 1.525 | 10.85 | 10.22 | 7.58 | 6.75 |
| 10 | 1.591 | 10.70 | 10.14 | 7.47 | 6.59 |
| 11 | 1.657 | 10.55 | 10.05 | 7.36 | 6.45 |
| 12 | 1.722 | 10.31 | 9.87 | 7.25 | 6.33 |
| 13 | 1.788 | 10.18 | 9.78 | 7.17 | 6.22 |
| 14 | 1.854 | 10.05 | 9.69 | 7.08 | 6.13 |
| 15 | 1.919 | 9.88 | 9.55 | 6.96 | 6.00 |
| 16 | 1.985 | 9.76 | 9.42 | 6.86 | 5.92 |
| 17 | 2.051 | 9.72 | 9.39 | 6.83 | 5.87 |
| 18 | 2.116 | 9.66 | 9.36 | 6.79 | 5.84 |
| 19 | 2.182 | 9.59 | 9.32 | 6.73 | 5.78 |
| 20 | 2.247 | 9.54 | 9.28 | 6.66 | 5.74 |
| 21 | 2.313 | 9.47 | 9.24 | 6.59 | 5.69 |
| 22 | 2.379 | 9.39 | 9.20 | 6.51 | 5.65 |
| 23 | 2.444 | 9.33 | 9.16 | 6.43 | 5.61 |
| 24 | 2.510 | 9.27 | 9.10 | 6.35 | 5.55 |
| 25 | 2.576 | 9.22 | 9.06 | 6.29 | 5.51 |
| 26 | 2.631 | 9.18 | 9.03 | 6.25 | 5.48 |
| 27 | 2.697 | 9.13 | 9.00 | 6.20 | 5.45 |
| 28 | 2.763 | 9.08 | 8.96 | 6.15 | 5.42 |
| 29 | 2.828 | 9.05 | 8.93 | 6.10 | 5.39 |
| 30 | 2.894 | 9.00 | 8.89 | 6.06 | 5.36 |
| 31 | 2.960 | 8.99 | 8.89 | 6.03 | 5.35 |
| 32 | 3.025 | 8.98 | 8.88 | 5.98 | 5.32 |
| 33 | 3.091 | 8.96 | 8.86 | 5.95 | 5.29 |
| 34 | 3.157 | 8.94 | 8.84 | 5.92 | 5.27 |
| 35 | 3.222 | 8.93 | 8.83 | 5.89 | 5.25 |
| 36 | 3.288 | 8.91 | 8.81 | 5.86 | 5.23 |
| 37 | 3.354 | 8.89 | 8.79 | 5.83 | 5.21 |
| 38 | 3.419 | 8.88 | 8.77 | 5.80 | 5.19 |
| 39 | 3.485 | 8.86 | 8.76 | 5.78 | 5.17 |
| 40 | 3.551 | 8.84 | 8.74 | 5.75 | 5.15 |
| 41 | 3.616 | 8.83 | 8.73 | 5.72 | 5.14 |
| 42 | 3.682 | 8.81 | 8.72 | 5.70 | 5.12 |
| 43 | 3.747 | 8.80 | 8.71 | 5.68 | 5.11 |
| 44 | 3.813 | 8.79 | 8.70 | 5.66 | 5.10 |
| 45 | 3.879 | 8.78 | 8.69 | 5.64 | 5.09 |
| 46 | 3.944 | 8.77 | 8.69 | 5.61 | 5.08 |
| 47 | 4.010 | 8.76 | 8.68 | 5.60 | 5.08 |
| 48 | 4.076 | 8.75 | 8.67 | 5.58 | 5.07 |
| 49 | 4.141 | 8.74 | 8.66 | 5.56 | 5.06 |
| 50 | 4.207 | 8.73 | 8.65 | 5.55 | 5.05 |
| 51 | 4.273 | 8.72 | 8.64 | 5.54 | 5.05 |
| 52 | 4.338 | 8.71 | 8.63 | 5.53 | 5.04 |
| 53 | 4.404 | 8.70 | 8.62 | 5.52 | 5.04 |
| 54 | 4.470 | 8.69 | 8.61 | 5.51 | 5.03 |
| 55 | 4.535 | 8.68 | 8.60 | 5.50 | 5.03 |
| 56 | 4.601 | 8.67 | 8.60 | 5.49 | 5.02 |
| 57 | 4.667 | 8.66 | 8.59 | 5.48 | 5.02 |
| 58 | 4.732 | 8.65 | 8.59 | 5.47 | 5.02 |
| 59 | 4.798 | 8.65 | 8.58 | 5.46 | 5.01 |
| 60 | 4.864 | 8.64 | 8.58 | 5.46 | 5.01 |
| 61 | 4.929 | 8.63 | 8.57 | 5.45 | 5.01 |
| 62 | 4.995 | 8.62 | 8.57 | 5.44 | 5.01 |
| 63 | 5.061 | 8.62 | 8.57 | 5.44 | 5.01 |
| 64 | 5.126 | 8.61 | 8.56 | 5.43 | 5.01 |
| 65 | 5.192 | 8.61 | 8.56 | 5.43 | 5.01 |
| 66 | 5.258 | 8.60 | 8.55 | 5.42 | 5.01 |
| 67 | 5.323 | 8.59 | 8.55 | 5.42 | 5.01 |
| 68 | 5.389 | 8.59 | 8.54 | 5.41 | 5.01 |
| 69 | 5.455 | 8.58 | 8.54 | 5.41 | 5.01 |
| 70 | 5.520 | 8.58 | 8.53 | 5.40 | 5.01 |
| 71 | 5.586 | 8.57 | 8.53 | 5.40 | 5.01 |
| 72 | 5.652 | 8.57 | 8.53 | 5.40 | 5.01 |
| 73 | 5.717 | 8.56 | 8.52 | 5.39 | 5.01 |
| 74 | 5.783 | 8.56 | 8.52 | 5.39 | 5.01 |
| 75 | 5.848 | 8.55 | 8.51 | 5.38 | 5.02 |
| 76 | 5.914 | 8.55 | 8.51 | 5.38 | 5.02 |
| 77 | 5.980 | 8.55 | 8.50 | 5.38 | 5.02 |
| 78 | 6.045 | 8.54 | 8.50 | 5.37 | 5.02 |
| 79 | 6.111 | 8.54 | 8.50 | 5.37 | 5.02 |
| 80 | 6.177 | 8.54 | 8.49 | 5.37 | 5.02 |
| 81 | 6.242 | 8.53 | 8.49 | 5.36 | 5.02 |
| 82 | 6.308 | 8.53 | 8.49 | 5.36 | 5.03 |
| 83 | 6.374 | 8.53 | 8.48 | 5.36 | 5.03 |
| 84 | 6.439 | 8.53 | 8.48 | 5.35 | 5.03 |
| 85 | 6.505 | 8.52 | 8.48 | 5.35 | 5.03 |
| 86 | 6.571 | 8.52 | 8.48 | 5.35 | 5.03 |
| 87 | 6.636 | 8.52 | 8.47 | 5.35 | 5.03 |
| 88 | 6.702 | 8.52 | 8.47 | 5.35 | 5.03 |
| 89 | 6.768 | 8.51 | 8.47 | 5.34 | 5.04 |
| 90 | 6.833 | 8.51 | 8.47 | 5.34 | 5.04 |
| 91 | 6.899 | 8.51 | 8.46 | 5.34 | 5.04 |
| 92 | 6.965 | 8.51 | 8.46 | 5.34 | 5.04 |
| 93 | 7.030 | 8.51 | 8.46 | 5.34 | 5.04 |
| 94 | 7.096 | 8.51 | 8.46 | 5.34 | 5.04 |
| 95 | 7.162 | 8.51 | 8.46 | 5.35 | 5.05 |
| 96 | 7.227 | 8.51 | 8.46 | 5.35 | 5.05 |
| 97 | 7.293 | 8.51 | 8.46 | 5.34 | 5.05 |
| 98 | 7.359 | 8.50 | 8.45 | 5.34 | 5.05 |
| 99 | 7.434 | 8.50 | 8.45 | 5.33 | 5.05 |
| 100 | 7.500 | 8.50 | 8.45 | 5.33 | 5.05 |


### 31.4 Implementation pseudocode

```text
function interpolate_K(selected_curve, ratio):
    if ratio < 1.000 or ratio > 7.500:
        raise AspectRatioOutOfRange

    if ratio exactly matches a stored a/b value:
        return stored K for selected_curve

    find consecutive rows j and j+1 such that:
        ratio_j < ratio < ratio_j_plus_1

    K_low  = K[selected_curve][j]
    K_high = K[selected_curve][j+1]

    fraction = (ratio - ratio_j) / (ratio_j_plus_1 - ratio_j)
    K = K_low + fraction * (K_high - K_low)

    return K
```

### 31.5 Interpolation output for the detailed report

For traceability, the calculation report should show an interpolation block like:

```text
Selected bay: Bay 2
Selected curve: K3
Calculated a/b: 2.600
Lower point:  a/b = 2.576, K3 = 6.29
Upper point:  a/b = 2.631, K3 = 6.25
Interpolation fraction: (2.600 - 2.576) / (2.631 - 2.576)
Interpolated K3: 6.2725
```

The values above illustrate the method. The application must calculate the final result from the stored data without prematurely rounding intermediate calculations.
