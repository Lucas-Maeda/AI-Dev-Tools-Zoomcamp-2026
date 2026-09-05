# Simplified Stiffened Panel Analysis Tool

## Consolidated Brainstorming and MVP Scope

This document consolidates the full project brainstorming conducted so far. It records the agreed scope, modeling assumptions, inputs, calculation workflow, user-interface behavior, results, validation rules, reporting requirements, and unresolved technical points.

---

## 1. Project Vision

The project will be a **simplified stiffened-panel analysis tool** aimed at engineers performing preliminary or educational aerospace structural calculations.

The tool is not intended to be a general finite element solver. The engineer should draw a conceptual panel layout, define a small set of properties, loads, and supports, and obtain transparent analytical results.

The primary purposes are:

1. Calculate the global shear-flow distribution in the idealized panel.
2. Plot the normal-load diagram for a structural boundary selected by the user.
3. Perform bay-by-bay buckling verification using classical analytical equations and predefined curves of buckling coefficient \(K\) versus aspect ratio \(a/b\).
4. Calculate the maximum admissible external load for a selected bay when the solution is uniquely defined.
5. Export a complete engineering calculation report in PDF format.

A future secondary purpose may be to support quick trade-off studies between alternative panel configurations, but optimization and automated design suggestions are outside the MVP.

---

## 2. Core Philosophy

The tool should behave like a **simplified aerospace structures calculator and analysis workbench**, not like an FEM preprocessor.

The engineer should think:

> “I am drawing my conceptual panel and obtaining understandable engineering results.”

The engineer should not need to think about:

- finite element meshes;
- element formulations;
- node and connectivity tables;
- nonlinear solution controls;
- contact;
- local stress concentrations;
- detailed stress recovery.

The calculations should be fast, explainable, traceable, and compatible with hand-calculation methods.

---

## 3. Intended Structural Idealization

### 3.1 Boom-skin concept

The selected analytical philosophy is a **boom-skin idealization** suitable for simplified aircraft-style panel calculations.

For the MVP:

- the skin carries shear flow;
- boundary lines act as idealized stiffeners or supporting members;
- normal-load diagrams are obtained from geometry, supports, external loads, global equilibrium, and shear-flow equilibrium;
- the boundary lines do not require physical section properties;
- no detailed stiffener-strength or stiffener-buckling calculation is performed.

### 3.2 Meaning of skin and stiffener

- **Skin:** the thin plate or sheet forming each rectangular bay.
- **Stiffener/boundary:** the line surrounding or dividing the skin bays.

The skin is not the stiffener.

### 3.3 Stiffener representation

In the MVP, the stiffeners are **geometric boundaries only**.

They have:

- a start vertex;
- an end vertex;
- a position in the panel layout.

They do not have:

- cross-sectional area \(A\);
- moment of inertia \(I\);
- a separate material;
- Euler buckling checks;
- crippling checks.

Every line drawn as part of the panel layout is automatically treated as a stiffener or structural boundary.

---

## 4. Geometry Definition

### 4.1 What the user draws

The user draws only:

- the external boundary of the complete panel;
- the internal lines that divide the panel into bays.

The user does not explicitly create nodes, stiffeners, connectivity, or bays.

### 4.2 What the software detects automatically

The software must automatically:

- create vertices at endpoints and intersections;
- determine line connectivity;
- treat all drawn lines as structural boundaries;
- detect every closed bay;
- number bays automatically as `Bay 1`, `Bay 2`, `Bay 3`, and so forth;
- number the structural boundaries internally, when needed;
- distinguish outer-boundary vertices from internal intersections.

### 4.3 Geometry restrictions

The MVP supports:

- one connected panel structure per model;
- rectangular or square bays only;
- perfectly horizontal and vertical bay boundaries;
- closed regions only.

The MVP does not support:

- disconnected panel structures in one project;
- skewed panels;
- trapezoidal bays;
- arbitrary quadrilaterals;
- curved boundaries;
- open or incomplete bays.

### 4.4 Bay dimensions

After the software detects the bays, the user must enter, for each bay:

- width \(b\);
- height \(a\).

The dimensions are not inferred from the apparent scale of the sketch.

The software calculates:

\[
\frac{a}{b}
\]

for each bay.

### 4.5 Bay naming

Bay names are generated automatically only:

- Bay 1;
- Bay 2;
- Bay 3;
- etc.

Custom bay names are outside the MVP.

---

## 5. Material and Thickness

### 5.1 Material assignment

The complete panel uses **one global material**.

The user may:

- choose a material from a library; or
- define a custom material.

The required material properties include:

- Young’s modulus \(E\);
- Poisson’s ratio \(\nu\), when required by the final verified buckling formula.

Different materials per bay and material zones are outside the MVP.

### 5.2 Thickness

The complete panel uses **one global skin thickness** \(t\).

The user enters the thickness once, and the same value applies to every bay.

Different thicknesses per bay and tapered skins are outside the MVP.

---

## 6. Supports

### 6.1 Available support types

The user assigns supports using familiar support categories:

- fixed;
- pinned;
- roller.

### 6.2 Support location

Supports may be assigned only to vertices on the **outer boundary of the complete panel**.

Supports are not allowed at internal intersections.

### 6.3 Load-support exclusivity

A vertex cannot simultaneously contain an external load and any type of support.

This rule applies equally to:

- fixed supports;
- pinned supports;
- roller supports.

If a load and support are assigned to the same vertex, the model must be rejected before solving.

Suggested validation message:

> **Validation Error:** A vertex cannot simultaneously contain a support condition and an external load. Remove the load or the support.

---

## 7. External Loads

### 7.1 Number of loads

The user may define as many external loads as required, subject to the location and direction rules below.

### 7.2 Permitted load location

External loads may be applied only at vertices on the **outer boundary of the complete panel**.

External loads are prohibited at:

- internal intersections;
- points in the middle of a boundary line;
- points in the middle of a stiffener;
- support vertices.

### 7.3 Load definition

For each load, the user defines:

- magnitude;
- tension or compression;
- an allowed horizontal or vertical direction when the vertex geometry provides more than one possible direction.

The load is always normal/perpendicular to the selected external boundary direction.

### 7.4 Direction convention

- **Compression:** the arrow points inward, toward the panel structure.
- **Tension:** the arrow points outward, away from the panel structure.

At an outer corner, the user may choose the applicable horizontal or vertical load direction.

At an internal intersection, no load is permitted, eliminating ambiguity about inward and outward directions.

### 7.5 Excluded load types

The MVP does not support:

- loads applied inside a bay;
- loads applied along a boundary segment;
- distributed edge loads;
- applied moments;
- loads at internal vertices;
- arbitrary angled loads.

---

## 8. Global Shear-Flow Analysis

### 8.1 Global solution

The tool must solve the **global shear-flow distribution** using:

- the complete connected geometry;
- all support conditions;
- all external nodal loads.

A load may influence bays that do not directly touch the loaded vertex because the structure transfers load globally through equilibrium.

The tool must not use a purely local load-transfer assumption.

### 8.2 One shear-flow model/value per bay

Each bay has one associated shear-flow result or shear-flow model for the simplified analysis.

The exact mathematical formulation must be validated against the intended Aula 7 methodology before implementation.

### 8.3 Automatic mode

In automatic mode, the software must:

1. detect the bays and boundaries;
2. interpret supports and loads;
3. build the equilibrium equations;
4. solve for all shear-flow values \(q_i\);
5. check equilibrium and solution consistency.

### 8.4 Detailed-calculation mode

The software must also provide an optional control such as:

```text
[ ] Show detailed calculations
```

When enabled, the software should display:

- equilibrium equations;
- sign conventions;
- intermediate substitutions;
- solved shear-flow values;
- equations used to construct normal-load diagrams.

This mode is intended for engineering verification, teaching, and comparison with hand calculations.

### 8.5 Shear-flow visualization

The results must show:

- numerical shear-flow values on the relevant boundaries or bays;
- arrows indicating shear-flow direction;
- a sign-convention legend explaining positive and negative flow.

The display should not rely on a color contour alone.

---

## 9. Normal-Load Diagram

### 9.1 Calculation basis

The normal-load diagram is not entered by the user.

It is calculated solely from:

- geometry;
- restrictions/supports;
- external loads;
- calculated shear-flow distribution;
- equilibrium along the structural boundary.

No stiffener area, inertia, or material is used in the MVP normal-load calculation.

### 9.2 User interaction

After solving, the user selects which structural boundary/stiffener should be inspected.

The software then plots the normal-load diagram for that selected boundary only.

### 9.3 Normal-load output

For the selected boundary, the tool should show:

- the normal-load function \(N(x)\), potentially piecewise;
- the plotted normal-load diagram;
- key values at relevant locations;
- maximum normal load;
- minimum normal load;
- contributing shear-flow terms;
- detailed equilibrium equations when detailed-calculation mode is enabled.

The visual form should resemble a conventional engineering normal-force diagram, like the reference notebook calculation discussed during brainstorming.

---

## 10. Buckling-Curve Interface

### 10.1 Reference chart

The stability module uses a chart containing approximately four curves of:

\[
K \;\text{versus}\; a/b
\]

The final definitions and exact data for those curves must be taken from and validated against the reference material, particularly Aula 7 from the `PEEA_I` notebook/PDF.

### 10.2 Curve selection per bay

For each bay, the user selects the applicable curve:

- Curve 1;
- Curve 2;
- Curve 3;
- Curve 4.

Each bay may use a different curve.

### 10.3 Proposed UI

The stability screen should contain:

#### Left side

- selected bay dropdown;
- curve-selection dropdown;
- short explanation of what the selected curve represents;
- calculated \(a\), \(b\), and \(a/b\);
- interpolated \(K\).

#### Right side

- the reference \(K\) versus \(a/b\) chart;
- all available curves for review;
- the selected curve highlighted;
- the current \(a/b\) position marked;
- the interpolated \(K\) point shown on the graph.

This interface is intended to make the interpolation transparent and allow the engineer to understand why a particular \(K\) value was used.

### 10.4 Curve descriptions

A brief explanation above or beside the chart should state:

- what the graph represents;
- what the horizontal and vertical axes mean;
- what each curve represents;
- when each curve should be selected.

The actual descriptions must be copied or derived accurately from the verified Aula 7 source, not guessed.

---

## 11. Stability Analysis

### 11.1 Bay-centered workflow

The stability analysis is performed for a bay selected by the user from a dropdown list.

The selected bay should also be highlighted in the geometry.

The interface should show:

- full details for the selected bay; and
- a compact comparison table for all bays.

### 11.2 Actual shear stress

For bay \(i\), the actual shear stress is calculated from the global shear-flow result:

\[
\tau_i = \frac{|q_i|}{t}
\]

The absolute value may be used for comparison against a positive critical magnitude, while the signed \(q_i\) remains visible in the shear-flow results.

### 11.3 Critical buckling stress

The discussed simplified form is:

\[
\tau_{cr,i} = K_i E\left(\frac{t}{b_i}\right)^2
\]

where:

- \(K_i\) is interpolated from the selected curve;
- \(E\) is the global Young’s modulus;
- \(t\) is the global skin thickness;
- \(b_i\) is the bay width.

**Important:** the exact equation, constants, dimensional convention, possible dependence on \(\nu\), and whether the chart represents shear buckling or compression buckling must be verified against Aula 7 before coding.

### 11.4 Required results for every selected bay

The stability module always displays:

- bay identifier;
- width \(b\);
- height \(a\);
- aspect ratio \(a/b\);
- selected curve;
- interpolated \(K\);
- shear flow \(q\);
- actual stress \(\tau=q/t\);
- critical buckling stress \(\tau_{cr}\);
- utilization.

The utilization is:

\[
U_i = \frac{|\tau_i|}{\tau_{cr,i}}
\]

The display should make clear whether the current bay is below, at, or above the critical value.

### 11.5 All-bay comparison

While a selected bay remains the focus, the interface should include a small table comparing all bays, such as:

| Bay | \(a/b\) | Curve | \(K\) | \(\tau\) | \(\tau_{cr}\) | Utilization |
|---|---:|---|---:|---:|---:|---:|
| Bay 1 | ... | ... | ... | ... | ... | ... |
| Bay 2 | ... | ... | ... | ... | ... | ... |

The selected bay remains highlighted.

---

## 12. Association Between Loads and Bays

### 12.1 Association rule

An external load is considered associated with a bay when the load is applied at any outer-boundary vertex that bounds that bay.

### 12.2 Shared vertex

If a loaded outer-boundary vertex belongs to two bays, the same load is associated with both bays.

Because the two bays may have different:

- shear flows;
- dimensions;
- aspect ratios;
- selected curves;
- interpolated \(K\) values;

that one external load may produce two different bay-specific critical-load results.

### 12.3 Global versus bay-specific behavior

The global shear-flow solution always considers the entire structure and every external load.

The association rule is used specifically to determine whether a unique bay-level maximum-load calculation is allowed.

---

## 13. Maximum External Load Calculation

### 13.1 Exactly one load associated with the selected bay

If exactly one external load is associated with the selected bay, the software may calculate that load’s critical value.

If the shear flow is linear in the external load \(P\), for example:

\[
q_i(P) = \alpha_i P
\]

then the limiting condition is:

\[
\frac{|q_i(P_{cr})|}{t} = \tau_{cr,i}
\]

or:

\[
\frac{|\alpha_i P_{cr}|}{t}
=
K_iE\left(\frac{t}{b_i}\right)^2
\]

The software solves this equation for the bay-specific maximum external load \(P_{cr}\).

### 13.2 Multiple loads associated with the same bay

If two or more external loads are associated with the selected bay, the software must **not** calculate individual maximum load values.

For example, if:

\[
q_i = \alpha_i P_1 + \beta_i P_2
\]

then imposing:

\[
\frac{|q_i|}{t} = \tau_{cr,i}
\]

provides a relationship between \(P_1\) and \(P_2\), but not a unique value for each load without an additional load relationship.

The MVP will not create a load envelope or solve arbitrary load combinations.

In this case, the tool still reports:

- current \(q_i\);
- current \(\tau_i\);
- \(\tau_{cr,i}\);
- utilization;
- a clear explanation that a unique maximum external load is unavailable because multiple loads are associated with the bay.

Suggested message:

> **Critical load calculation not available:** More than one external load is associated with this bay. The critical buckling stress and current utilization are still provided.

### 13.3 Loads associated with different bays

Multiple external loads do not inherently prevent the critical-load calculation when each selected bay has only one associated external load.

The uniqueness rule is evaluated separately for each bay selected by the user.

### 13.4 Scope limitation

The MVP is not a general load-envelope solver and does not calculate independent allowable combinations of several variable external loads.

---

## 14. Results Visualization

### 14.1 Geometry view

After solving, the geometry view should display:

- bay numbers;
- supports;
- external loads;
- load arrows and tension/compression direction;
- shear-flow arrows;
- shear-flow numerical values;
- a sign-convention legend;
- the currently selected bay highlighted.

### 14.2 Safety visualization

The results should combine:

- exact engineering values; and
- a simple visual indication of utilization or criticality.

A color approach may indicate safe, warning, and governing/critical conditions, but numerical values must always remain available.

### 14.3 Normal-load view

The user selects a boundary and sees the corresponding normal-load diagram and equations.

### 14.4 Buckling-curve view

The user selects a bay and sees the chart, selected curve, current \(a/b\), interpolation point, \(K\), stresses, utilization, and maximum load when applicable.

---

## 15. Solve Workflow

The tool must use an explicit **Solve** button.

Edits do not automatically trigger a complete recalculation.

The intended workflow is:

1. Draw the panel layout.
2. Let the software detect vertices, boundaries, and bays.
3. Enter width and height for each bay.
4. Define the global material.
5. Define the global skin thickness.
6. Apply supports to allowed outer-boundary vertices.
7. Apply external loads to allowed outer-boundary vertices.
8. Select a buckling curve for each bay.
9. Press **Solve**.
10. Review validation messages, shear flows, normal-load diagrams, and stability results.
11. Export the full PDF engineering report.

---

## 16. Model Validation Rules

The software must validate the model before solving.

### 16.1 Geometry validation

Reject the model if:

- more than one disconnected structure exists;
- a bay is open;
- a bay is not rectangular or square;
- a bay has a slanted or curved boundary;
- a boundary intersection creates unsupported topology;
- required bay dimensions are missing or invalid.

### 16.2 Load validation

Reject or prevent:

- a load at an internal intersection;
- a load in the middle of a boundary;
- a load at a support vertex;
- an arbitrary angled load;
- a distributed load;
- an applied moment;
- a load without magnitude or tension/compression definition.

### 16.3 Support validation

Reject or prevent:

- a support at an internal vertex;
- a support at a vertex containing a load;
- an invalid or insufficient support configuration that prevents a stable equilibrium solution.

### 16.4 Property validation

Reject or warn about:

- nonpositive thickness;
- nonpositive Young’s modulus;
- missing material data;
- nonpositive bay width or height;
- missing curve selection;
- an \(a/b\) value outside the digitized range of the selected curve.

For an aspect ratio outside the curve range, the MVP should not silently extrapolate unless extrapolation is explicitly validated by the source methodology.

### 16.5 Solver validation

The solver should detect and report:

- inconsistent equations;
- underdetermined shear-flow systems;
- singular or unstable support configurations;
- equilibrium residuals above the accepted numerical tolerance;
- missing normal-load paths;
- cases in which a unique maximum load cannot be calculated.

---

## 17. PDF Engineering Report

The MVP must export a **full engineering report in PDF format**.

The PDF is a core deliverable because the calculation must be reviewable and traceable.

### 17.1 Model description

Include:

- project/model title;
- panel geometry;
- bay numbering;
- width and height of every bay;
- global thickness;
- material and properties;
- supports;
- external loads;
- sign conventions.

### 17.2 Shear-flow analysis

Include:

- shear-flow diagram;
- arrows indicating flow direction;
- numerical values;
- sign-convention legend;
- equilibrium equations and intermediate calculations when detailed-calculation mode is enabled.

### 17.3 Normal-load analysis

For the structural boundary selected for reporting, include:

- boundary identifier;
- normal-load equation \(N(x)\);
- piecewise equations if applicable;
- normal-load diagram;
- maximum and minimum values;
- detailed derivation when requested.

### 17.4 Stability analysis

For every bay, include:

- bay identifier;
- width \(b\);
- height \(a\);
- \(a/b\);
- selected curve;
- interpolated \(K\);
- shear flow \(q\);
- actual stress \(\tau=q/t\);
- critical buckling stress \(\tau_{cr}\);
- utilization;
- maximum external load, when uniquely calculable;
- explanation when the maximum load is unavailable.

### 17.5 Buckling chart

Include:

- the reference \(K\) versus \(a/b\) chart;
- curve descriptions;
- selected curve for the relevant bay;
- interpolation point.

### 17.6 Conclusions

Include:

- the selected bay result;
- a comparison of all bays;
- the most highly utilized bay under the current load case;
- any bay-specific critical external loads that were uniquely calculated;
- limitations or validation warnings applicable to the analysis.

---

## 18. Explicit MVP Exclusions

The first version will not include:

- finite element meshing;
- full FEA;
- geometric or material nonlinearity;
- contact;
- local stress concentrations;
- plasticity;
- post-buckling reserve;
- crippling;
- Euler buckling of stiffeners;
- stiffener area or inertia inputs;
- stiffener section libraries;
- separate materials by bay;
- separate thicknesses by bay;
- tapered skins;
- arbitrary panel shapes;
- angled boundaries;
- interior loads;
- distributed loads;
- applied moments;
- arbitrary load directions;
- supports at internal vertices;
- loads at support vertices;
- multiple disconnected structures;
- general load-envelope calculation;
- optimization;
- automated design modification suggestions;
- trade-off automation in the initial MVP;
- custom bay names;
- spreadsheet export.

---

## 19. Potential Phase 2 Features

Possible future additions include:

- rapid trade-off comparison between panel layouts;
- automated suggestions to add boundaries or change dimensions;
- separate thickness per bay;
- material zones;
- stiffener section properties;
- Euler buckling of stiffeners;
- combined skin and stiffener failure modes;
- compression buckling if distinct from the current shear-buckling methodology;
- combined compression and shear interaction;
- post-buckling behavior;
- crippling;
- variable-load scaling or load-envelope analysis;
- Excel export;
- reusable native project files, if not included in the first implementation;
- automatic curve selection from physical edge conditions;
- additional verified curve families.

---

## 20. Important Technical Validations Before Implementation

The conceptual scope is well defined, but several equations and conventions must be verified before coding.

### 20.1 Confirm the type of buckling curves

Verify whether the Aula 7 curves are:

- shear-buckling curves;
- compression-buckling curves; or
- another class of stability curves.

This determines whether comparing \(q/t\) directly with the critical value from the chart is correct.

### 20.2 Confirm the exact critical-stress equation

Verify the full equation, including:

- constants;
- units;
- the role of Poisson’s ratio;
- definitions of \(a\) and \(b\);
- whether \(b\) must be the shorter side;
- edge-condition assumptions;
- applicability limits.

The simplified expression discussed was:

\[
\tau_{cr}=K E\left(\frac{t}{b}\right)^2
\]

but this must not be implemented until confirmed against Aula 7.

### 20.3 Digitize the curves

The four \(K\) versus \(a/b\) curves need verified numerical data.

The implementation should define:

- tabulated curve points;
- interpolation method;
- curve-domain limits;
- behavior at exact data points;
- behavior outside the valid domain;
- numerical tolerance.

### 20.4 Validate the global shear-flow formulation

The equilibrium formulation must be validated using known hand-calculation examples from Aula 7.

Validation should cover:

- one bay;
- multiple connected bays;
- shared boundaries;
- multiple outer-boundary loads;
- different support arrangements;
- shear-flow signs and directions;
- normal-load reconstruction;
- equilibrium residuals.

### 20.5 Confirm “one shear flow per bay”

The precise meaning and limits of one shear-flow result/model per bay should be confirmed from the reference methodology, especially where a bay has multiple boundary segments and shares internal boundaries with adjacent bays.

### 20.6 Define load association precisely

The agreed user-facing rule is geometric: a load is associated with a bay if the loaded outer-boundary vertex bounds that bay.

The implementation must preserve the separation between:

- global influence of loads in the complete equilibrium solution; and
- bay-level association used to decide whether a unique \(P_{cr}\) may be reported.

---

## 21. Consolidated MVP Definition

The MVP is:

> A simplified aircraft-style stiffened-panel analysis workbench in which the engineer draws one connected orthogonal panel layout, enters each rectangular bay’s width and height, defines one global material and skin thickness, applies tension or compression loads and supports only at separate outer-boundary vertices, solves the global shear-flow distribution, plots a selected boundary’s normal-load diagram, and performs bay-by-bay buckling verification using a user-selected \(K\) versus \(a/b\) curve. A bay-specific maximum external load is calculated only when exactly one external load is associated with that bay. The tool provides transparent calculations and exports a full PDF engineering report.

---

## 22. Decisions Log

The following choices were explicitly made during brainstorming:

- Primary purpose: verify an existing stiffened panel and determine stability limits.
- Secondary future purpose: quick configuration trade-off studies.
- Structural context: simplified aircraft-style panels.
- User experience: graphical panel drawing, not a generic form or FEM model builder.
- Geometry creation: draw panel boundaries/internal divisions; automatic vertices, boundaries, connectivity, and bays.
- Bay shape: rectangular/square only.
- Bay orientation: horizontal/vertical sides only.
- Dimensions: width and height entered per bay.
- Structure count: one connected structure only.
- Stiffeners: geometric lines only, no section properties.
- Analytical approach: boom-skin idealization.
- Skin role: carries shear flow.
- Thickness: one global value.
- Material: one global material, selectable from a library or custom.
- Loads: multiple loads allowed globally.
- Load locations: outer-boundary vertices only.
- Load definition: magnitude plus tension/compression; horizontal or vertical according to the selected external boundary.
- Internal loads: prohibited.
- Supports: fixed, pinned, or roller.
- Support locations: outer-boundary vertices only.
- Load and support on the same vertex: prohibited for all support types, including fixed nodes.
- Shear-flow solution: global, not purely local.
- Shear-flow solver: automatic with optional detailed equations.
- Shear-flow display: values, arrows, and sign-convention legend.
- Normal-load diagram: generated on demand for a user-selected boundary.
- Buckling: bay-centered.
- Curve selection: one of approximately four curves selected per bay.
- Curve UI: dropdown plus chart and explanatory text.
- \(K\): computed by interpolation using the calculated \(a/b\).
- Bay selection: dropdown.
- Bay display: selected-bay details plus compact all-bay comparison.
- Selected bay: highlighted in the geometry.
- Critical stress: always calculated for the selected bay.
- Maximum external load: calculated only when exactly one load is associated with the selected bay.
- Multiple loads associated with one bay: no unique maximum-load result in the MVP.
- Solve behavior: explicit Solve button.
- Export: full PDF engineering report.

---

## 23. Next Recommended Project Step

Before selecting the software platform or beginning UI development, the next technical checkpoint should be to extract and verify the Aula 7 methodology:

1. transcribe the exact equations;
2. identify the meaning of all four curves;
3. digitize the curves;
4. define the interpolation rules;
5. reproduce at least one complete hand-worked example;
6. confirm the shear-flow and normal-load equations;
7. use that example as the first solver acceptance test.

This validation should precede implementation because the largest remaining project risk is the mathematical interpretation of the reference method, not the graphical interface.
