---
title: "LQR Station-Keeping at Earth-Moon L1"
excerpt: "Designed an LQR controller to stabilize a spacecraft at the unstable L1 libration point, then swept the control penalty across five orders of magnitude to map a parabolic fuel-cost tradeoff and a distinct accuracy-vs-fuel tradeoff under persistent disturbances. <br/><img src='/images/lqr_traj_comparison.png'>"
collection: portfolio
---

## Overview

Collinear libration points in the Circular Restricted Three-Body Problem 
(CR3BP) are unstable saddle equilibria — a fact with real consequences for 
missions like JWST (L2) and the planned Lunar Gateway. Any spacecraft placed 
near one of these points will drift away exponentially without active 
control. This project designs a Linear Quadratic Regulator (LQR) to hold a 
spacecraft near Earth-Moon L1, validates the controller against the full 
nonlinear dynamics, and then sweeps the control penalty weighting across 
five orders of magnitude to characterize how control aggressiveness trades 
off against fuel cost and station-keeping accuracy — revealing a parabolic 
fuel-cost curve with a genuine optimum, and a separate, distinct 
accuracy-vs-fuel tradeoff when a persistent disturbance is present.

---

## Theory

### Why L1 Needs Active Control

The libration points are equilibria of the CR3BP where gravitational and 
centrifugal forces balance in the rotating frame. Linearizing the CR3BP 
equations of motion about L1 produces a $6\times6$ system matrix whose 
eigenvalues reveal the local stability structure:

$$\delta\dot{\mathbf{x}} = A\,\delta\mathbf{x}, \qquad A = \begin{bmatrix} 0_{3\times3} & I_{3\times3} \\ U_{\text{pos}} & 2\Omega \end{bmatrix}$$

where $U_{\text{pos}}$ is the Hessian of the effective potential and $2\Omega$ 
is the Coriolis coupling matrix. At L1, this yields:

$$\lambda \in \{\pm 2.932,\ \pm 2.334i,\ \pm 2.269i\}$$

one real positive eigenvalue (unstable), one real negative eigenvalue 
(stable), and two oscillatory center-manifold pairs — the textbook 
saddle $\times$ center $\times$ center structure. The positive real 
eigenvalue is what forces the station-keeping problem to exist: any 
perturbation along that direction grows as $e^{2.93t}$ in nondimensional 
time.

### Control Design

With full 3-axis thrust authority ($B = [0_{3\times3};\ I_{3\times3}]^T$), 
the linearized pair $(A,B)$ was confirmed fully controllable (rank 6), and 
an LQR gain was computed by solving the algebraic Riccati equation for a 
range of control-penalty weightings $R$, holding $Q = I_6$ fixed. The 
resulting control law $\mathbf{u} = -K\,\delta\mathbf{x}$ was applied to 
the **full nonlinear** CR3BP dynamics (not just the linear model used for 
design) — a standard and important validation step, since a controller 
that only works on its own linearization isn't very useful.

---

## Results

### LQR Stabilizes the Saddle Point

A small initial offset of approximately 38 km from L1 grows to roughly 
32,000 km within about 4.6 days under no control. With LQR active, the 
same offset is driven down by roughly three orders of magnitude and held 
near L1 indefinitely, even under a small continuous disturbance 
representing unmodeled effects like solar radiation pressure.

![Trajectory comparison: uncontrolled divergence vs. LQR-controlled, with start/end markers](/images/lqr_traj_comparison.png)

The closed-loop eigenvalues confirm exactly what changed: the previously 
unstable mode ($+2.93$) becomes stable ($-2.90$), and the oscillatory 
center-manifold modes each picked up modest additional damping — LQR 
didn't just barely tame the bad mode, it improved the whole system's 
behavior.

### The Q/R Fuel-Cost Tradeoff Is Parabolic, Not Monotonic

An initial sweep across $R \in [1, 10000]$ suggested total $\Delta V$ 
simply increased with $R$ (gentler control costing more fuel, since the 
correction had to be sustained for longer). Extending the sweep down to 
$R = 0.01$ revealed the fuller picture: **the true relationship is 
parabolic**, with a sharp minimum near $R \approx 1$ and cost rising on 
*both* sides.

![Total delta-V vs. R showing the parabolic tradeoff for the offset-only case](/images/lqr_dv_comparison.png)

This makes physical sense once both regimes are considered:

- **Very small $R$ (aggressive control):** the controller commands large, 
  fast corrective accelerations. Without any actuator limits in the model, 
  overly aggressive gains can overshoot and fight their own correction, 
  wasting fuel on oscillatory over-correction rather than converging 
  efficiently.
- **Very large $R$ (gentle control):** the controller applies a small, 
  low-level correction sustained over a much longer time. Even though the 
  instantaneous thrust is smaller, the total integrated cost grows because 
  the correction has to be maintained for so long.

The minimum near $R \approx 1$ balances these two competing effects — a 
genuine optimal control penalty for this particular offset and dynamics, 
rather than an arbitrarily "good enough" tuning choice.

### Under a Persistent Disturbance: A Fuel-vs-Accuracy Tradeoff, Not Just Fuel-vs-Speed

With a small constant disturbance active (representing an unmodeled effect 
like solar radiation pressure), the annual station-keeping $\Delta V$ 
showed a similar early dip-and-rise pattern, but continued to *decrease* 
at larger $R$ values rather than turning back upward the way the 
offset-only case did.

Very large values of $R$ were excluded from the final sweep because they 
failed to settle within the simulation window (900 nondimensional time 
units) and disrupted the settling-time bookkeeping used elsewhere in the 
analysis — but the trend while they were still included pointed to a 
distinct explanation: **at large $R$, the controller trades 
station-keeping accuracy for fuel savings.** Rather than continuing to 
fully reject the disturbance, an overly gentle controller allows the 
steady-state position error to grow, and in doing so requires less 
average counter-thrust. Beyond a certain $R$, that steady-state error 
exceeds the desired station-keeping margin (2% of the initial offset, in 
this study) well before the simulation window ends — meaning the apparent 
fuel savings come at the cost of no longer meeting the mission's 
positional accuracy requirement.

This is a meaningfully different tradeoff than the offset-only case: there, 
the "cost" of an overly gentle controller was purely extra fuel. Here, the 
cost of an overly gentle controller is a station-keeping box that may be 
too large to be useful — a much more mission-relevant failure mode, and a 
good illustration of why real GNC teams tune station-keeping controllers 
against an explicit accuracy requirement rather than fuel cost alone.

---

## Tools & Methods

MATLAB, algebraic Riccati equation solving (`lqr`), nonlinear ODE 
integration (`ode45`), CR3BP linearization, controllability analysis, 
eigenstructure analysis, unit nondimensionalization/redimensionalization

---

## Links

- [View Code on GitHub](https://github.com/yourusername/lqr-l1-stationkeeping)
