---
title: "Optimal Two-Impulse Orbital Transfer via Stark Structure Method"
excerpt: "Implemented a bi-impulsive Lambert-type transfer solver using the Stark structure formulation, mapping the full cost/time tradeoff space for a LEO-to-Molniya transfer and visualizing the globally optimal trajectory in 3D. <br/><img src='/images/stark_3d_transfer.png'>"
collection: portfolio
---

## Overview

Given two orbits and a desired transfer angle, how much does it cost in 
\\(\Delta V\\) and in time to move a spacecraft from one to the other? This 
project answers that question using the **Stark structure method**, a 
lesser-known alternative to the classical universal-variables Lambert 
solver, applied to a real and recognizable orbit pair: an ISS-like low 
Earth orbit and a Molniya-like highly elliptical orbit. The solver sweeps 
the full space of departure points and transfer angles, identifies the 
globally optimal two-impulse transfer, and visualizes it in 3D alongside 
both parking orbits.

---

## Theory

### The Boundary Value Problem

A two-impulse orbital transfer is a boundary value problem: given a 
departure position \\(\mathbf{r}\_1\\) and arrival position \\(\mathbf{r}\_2\\), 
find the transfer orbit connecting them, along with the departure and 
arrival velocities \\(\mathbf{v}\_1^+\\) and \\(\mathbf{v}\_2^-\\) that minimize 
the total impulse cost

$$\Delta V = \|\mathbf{v}\_1^+ - \mathbf{v}\_1^-\| + \|\mathbf{v}\_2^+ - \mathbf{v}\_2^-\|$$

where \\(\mathbf{v}\_1^-\\) and \\(\mathbf{v}\_2^+\\) are the spacecraft's 
velocities on its original and final orbits, respectively.

### The Stark Structure Formulation

Rather than the standard universal-variables approach, this solver uses 
the **Stark structure** method, which transforms the boundary value 
problem into a polynomial root-finding problem in the transfer orbit's 
angular momentum. The chord vector between the two position vectors, 
\\(\mathbf{c} = \mathbf{r}\_2 - \mathbf{r}\_1\\), defines a natural basis: 
velocity components are decomposed into a **chord-aligned** component 
\\(v\_c\\) and a **radial-aligned** component \\(v\_\rho\\) at each end of the 
transfer.

Applying the boundary conditions in this basis yields a **quartic 
polynomial** in the transfer orbit's angular momentum \\(h\\):

$$h^4 + a\_3 h^3 + a\_1 h + a\_0 = 0$$

whose coefficients depend on the known geometry (\\(\mathbf{r}\_1\\), 
\\(\mathbf{r}\_2\\), \\(\mathbf{v}\_1^-\\), \\(\mathbf{v}\_2^+\\)) and gravitational 
parameter \\(\mu\\). Each **real, positive root** of this polynomial 
corresponds to a physically valid transfer geometry. For each root, the 
transfer semi-major axis, eccentricity, and Lagrange time-of-flight are 
computed, and the root yielding the lowest total impulse energy 
\\(J = \tfrac{1}{2}(\Delta\mathbf{v}\_1^T\Delta\mathbf{v}\_1 + \Delta\mathbf{v}\_2^T\Delta\mathbf{v}\_2)\\) 
is selected as the optimal transfer at that departure/arrival geometry.

The solver handles both **elliptical** transfer orbits (via the standard 
Lagrange time-of-flight equation using \\(\arcsin\\)) and **hyperbolic** 
transfer orbits (via the hyperbolic form using \\(\text{arcsinh}\\)), 
resolves the long-way vs. short-way transfer ambiguity by checking the 
eccentric anomaly change against the transfer angle, and includes a small 
perturbation scheme to avoid the coordinate singularity that occurs at 
exactly 180° transfer angles.

---

## Problem Setup

The solver was applied to a transfer between two real, recognizable orbit 
types:

**Orbit 1 — ISS-like:** \\(a = 6798\\) km, \\(e \approx 0.0003\\), 
\\(i = 51.6°\\)

**Orbit 2 — Molniya-like:** \\(a = 26{,}600\\) km, \\(e = 0.74\\), 
\\(i = 63.4°\\) (the critical inclination, which prevents apsidal 
precession), \\(\omega = 270°\\) (placing apogee over the northern 
hemisphere)

For every combination of departure true anomaly \\(f\_1\\) (0° to 360°) and 
transfer angle \\(\Delta f = f\_2 - f\_1\\) (10° to 350°), the solver computed 
the optimal two-impulse transfer cost and time of flight, producing a full 
porkchop-style cost/time map.

---

## Results

### Cost and Time Contour Maps

![Impulse cost and contour plot with global minimum annotated](/images/stark_contour_plot_dV.png)
![Transfer time contour plot with global minimum annotated](/images/stark_contour_plot_ToF.png)

The contour maps reveal the characteristic low-cost transfer regions 
across the departure/transfer-angle space. The **global minimum** transfer 
was found at:

$$\Delta V\_{\min} = 3.39 \text{ km/s} \quad \text{at } f\_1 = 277°,\ \Delta f = 345.3°,\ \text{TOF} = 5.42 \text{ hr}$$

This value is physically sensible: an unconstrained Hohmann-style raise 
from LEO to a Molniya-class apogee alone would cost roughly 2.4–2.5 km/s, 
so the additional ~0.9–1.0 km/s reflects the cost of the 11.8° plane 
change between the ISS-like (51.6°) and Molniya-like (63.4°) inclinations 
— a good physical sanity check on the solver's output.

### Optimal Transfer Trajectory in 3D

![3D visualization of the optimal transfer arc between ISS-like and Molniya-like orbits](/images/stark_3d_transfer.png)

The optimal transfer identified by the polynomial solver was independently 
verified by propagating the resulting departure velocity \\(\mathbf{v}\_1^+\\) 
forward through simple two-body dynamics for the computed time of flight. 
The resulting arc closely connects the two parking orbits at the predicted 
departure and arrival points, confirming consistency between the 
Stark-structure boundary value solution, the Lagrange time-of-flight 
calculation, and independent numerical propagation.

---

## Tools & Methods

MATLAB, polynomial root-finding, Lagrange time-of-flight equations 
(elliptical and hyperbolic forms), Stark structure orbital transfer 
theory, two-body numerical propagation (`ode45`) for trajectory 
verification

---

## Code

- Code available upon request.
