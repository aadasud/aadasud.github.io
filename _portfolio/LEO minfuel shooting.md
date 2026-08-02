---
title: "Minimum-Fuel LEO Altitude Raise via Indirect Optimal Control"
excerpt: "Derived and numerically solved the full Pontryagin Minimum Principle boundary value problem for a bang-bang, minimum-fuel electric-propulsion orbit raise, uncovering and resolving a genuine costate scale degeneracy along the way. <br/><img src='/images/leo_trajectory_final.png'>"
collection: portfolio
---

## Overview

Given a real, flight-proven ion thruster and a small satellite in low Earth 
orbit, what thrust-direction history minimizes the propellant needed to 
raise the orbit to a higher circular altitude? This project derives the 
full indirect optimal control solution via Pontryagin's Minimum Principle 
— Hamiltonian, costate equations, bang-bang switching structure, and 
transversality conditions — then solves the resulting two-point boundary 
value problem numerically via shooting, using an analytic state-transition 
matrix for a robust, `fsolve`-compatible Jacobian.

Along the way, the project surfaces a genuine, provable mathematical 
degeneracy in the naive costate parameterization — a scale-invariance that 
made the shooting Jacobian numerically singular — and resolves it by 
reducing to a minimal costate representation, dropping the system's 
condition number by roughly ten orders of magnitude.

---

## Mission Setup

**Spacecraft:** 8 kg small satellite, circular 400 km LEO parking orbit

**Thruster:** Busek BHT-200 — a real, flight-proven Hall-effect thruster 
(TacSat-2, FalconSat-5/6)
- Thrust: 13 mN
- Specific impulse: 1390 s → exhaust velocity \\(c = I\_{sp}g\_0 = 13.63\\) km/s

**Maneuver:** Circular-to-circular altitude raise, 400 km → 700 km (300 km 
raise), completed in roughly 18 orbits

---

## Theory

### Dynamics and the Hamiltonian

The spacecraft state is described in polar coordinates — radius \\(r\\), 
angle \\(\theta\\), radial velocity \\(u\\), tangential velocity \\(v\\), and 
mass \\(m\\) — with a controllable thrust vector of magnitude \\(T\\) and 
steering angle \\(\alpha\\):

$$\dot r = u, \quad \dot\theta = \frac{v}{r}, \quad \dot u = \frac{v^2}{r}-\frac{\mu}{r^2}+\frac{T}{m}\sin\alpha, \quad \dot v = -\frac{uv}{r}+\frac{T}{m}\cos\alpha, \quad \dot m = -\frac{T}{c}$$

A costate is attached to each state — \\(\lambda\_r,\lambda\_u,\lambda\_v,\lambda\_m\\) 
— with dynamics derived from the Hamiltonian \\(H=\lambda^Tf\\) via 
\\(\dot\lambda = -\partial H/\partial x\\). Minimizing \\(H\\) over the 
steering angle gives the classical primer-vector result: thrust always 
points exactly opposite the vector \\((\lambda\_u,\lambda\_v)\\):

$$\sin\alpha^* = \frac{-\lambda\_u}{\sqrt{\lambda\_u^2+\lambda\_v^2}}, \qquad \cos\alpha^* = \frac{-\lambda\_v}{\sqrt{\lambda\_u^2+\lambda\_v^2}}$$

### Bang-Bang Thrust Structure

Because thrust magnitude \\(T\\) enters the Hamiltonian *linearly* (not 
quadratically), minimizing \\(H\\) over the bounded control \\(T\in[0,T\_{max}]\\) 
always drives the solution to one endpoint or the other — there is no 
interior optimum. This produces a **switching function**

$$S(t) = \frac{\sqrt{\lambda\_u^2+\lambda\_v^2}}{m} + \frac{\lambda\_m}{c}$$

with thrust at full power when \\(S(t)>0\\) and off when \\(S(t)<0\\). \\(S(t)\\) 
was confirmed to stay strictly positive across the entire converged 
trajectory — for this weak-thrust, modest-transfer regime, the optimal 
strategy is continuous full thrust with no coast arcs.

### Transversality and the Sign of \\(\lambda\_m(t\_f)\\)

Since \\(m(t\_f)\\) is free (we want to *maximize* it, not hit a specific 
value) while \\(r(t\_f),u(t\_f),v(t\_f)\\) are fixed to the target orbit, the 
free-endpoint transversality condition applies specifically to the mass 
costate. Careful application of the augmented-cost first-variation result 
— \\(\left(\phi\_x+\psi\_x^T\nu-\lambda\right)\big|\_{t\_f}=0\\) for the free 
state components — gives

$$\lambda\_m(t\_f) = \frac{\partial\phi}{\partial m(t\_f)} = \frac{\partial(-m(t\_f))}{\partial m(t\_f)} = -1$$

for the standard Hamiltonian convention \\(H=\lambda^Tf\\), minimizing 
\\(J=-m(t\_f)\\). This was independently verified two ways: it is the only 
sign consistent with the correctly-derived switching function producing 
genuine (not degenerate) thrust-on behavior, and a full shooting solve 
under this convention converges to the same physical trajectory as the 
equivalent \\(+1\\) convention, differing only in the internal sign of the 
otherwise-decoupled \\(\lambda\_m\\) track.

---

## The Costate Scale Degeneracy

Initial shooting attempts, solving directly for all four initial costates 
\\((\lambda\_r,\lambda\_u,\lambda\_v,\lambda\_m)(0)\\), produced a Jacobian with 
condition number \\(\sim10^{15}\\) — numerically singular. 

The cause traces directly to the steering law: since \\(\alpha^*\\) is built 
from *normalized* ratios of \\((\lambda\_u,\lambda\_v)\\), and every term in 
the costate ODEs is linear and homogeneous in 
\\((\lambda\_r,\lambda\_u,\lambda\_v)\\), scaling the entire initial vector by 
any positive constant \\(k\\) leaves \\(\alpha^*(t)\\) — and therefore the 
entire physical trajectory — **exactly** unchanged. This was confirmed 
directly: scaling the converged costates by factors of 0.5× to 5× left 
\\(r\_f, u\_f, v\_f\\) bit-for-bit identical.

**Resolution:** the scale freedom was removed by fixing \\(\lambda\_v(0)=-1\\) 
(chosen for its reliably nonzero, known sign) and solving only for the two 
genuinely meaningful ratios plus \\(\lambda\_m(0)\\) — reducing the shooting 
problem from 4 unknowns to a minimal, non-degenerate set of 3. This 
dropped the Jacobian condition number to \\(\sim10^5\\), and the resulting 
system solves cleanly with `fsolve` using an analytic Jacobian built from 
a 9-state state-transition matrix integrated alongside the trajectory.

---

## Results

Solved via MATLAB's `fsolve` with an analytic STM-based Jacobian 
(`SpecifyObjectiveGradient`), starting from a generic, physically-motivated 
initial guess (near-zero radial/steering ratios, \\(\lambda\_m(0)\approx-1\\)) 
— no problem-specific tuning required:

```
Solution [ratio_r, ratio_u, lam_m0]: [-1.158487, 0.108055, -0.971391]
Final residual norm: 2.119320e-07
```

`fsolve` reported a genuine, unambiguous convergence — "Equation solved," 
sum-of-squares residual driven to \\(4.49\times10^{-14}\\) in 6 iterations, 
gradient optimality below tolerance.

**Final boundary condition errors:**

| Quantity | Error |
|---|---|
| Final radius | 0.91 m |
| Final radial velocity | 17.4 mm/s |
| Final tangential velocity | 0.2 mm/s |
| \\(\lambda\_m(t\_f)\\) | exact to displayed precision |

**Propellant used:** 96.4 g out of 8 kg initial mass (≈1.2% of total 
spacecraft mass) to complete the 300 km raise.

![LEO altitude raise trajectory, switching function, and steering history](/images/leo_trajectory_final.png)

*Top left: the trajectory spiral, with the altitude change exaggerated 20× 
relative to Earth's radius — at true scale, a 300 km raise on a 6778 km 
orbit is only a ~4.4% radius change and the spiral is visually 
indistinguishable from a thick circle, so the exaggeration (clearly 
labeled) is used purely to make the ~18-orbit spiral shape visible. Top 
right: altitude gain vs. time, the true-scale, unambiguous view of 
transfer progress. Bottom row: the switching function and optimal 
steering angle, both oscillating with the orbital period as the 
spacecraft's position relative to periapsis/apoapsis-like geometry cycles.*

The switching function stayed positive throughout the ~18-orbit transfer, 
confirming continuous-thrust operation with no coast arcs — consistent 
with this mission's very low thrust-to-mass ratio relative to the transfer 
required.

---

## Tools & Methods

MATLAB, Pontryagin's Minimum Principle, two-point boundary value problem 
solving via indirect shooting, analytic state-transition matrix (STM) for 
Jacobian computation, `fsolve` with user-supplied gradients, costate 
degeneracy diagnosis and minimal-representation reformulation, bang-bang 
optimal control

---

## Links

- [View Code on GitHub](https://github.com/yourusername/leo-minfuel-shooting)
