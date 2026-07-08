---
title: "State Estimation for Lunar Transfer Trajectories"
excerpt: "Implemented and compared Batch Least Squares and Unscented 
Kalman Filtering for spacecraft orbit determination in the CR3BP using 
simulated Deep Space Network measurements. <br/><img src='/images/lunar_traj.png'>"
collection: portfolio
---

## Overview

As humanity returns to the Moon through NASA's Artemis program, accurate 
real-time navigation during the translunar coast phase is critical. This 
project implements and compares two state estimation approaches — Batch 
Least Squares (BLS) and an Unscented Kalman Filter (UKF) — for estimating 
the position and velocity of a spacecraft traveling from Low Earth Orbit 
to the Moon.

The translunar coast is particularly challenging for navigation. The 
spacecraft traverses a highly nonlinear gravitational environment governed 
by Earth-Moon three-body dynamics, while ground station coverage becomes 
sparse as distance from Earth increases. This project directly addresses 
these challenges through simulation and rigorous filter comparison.

---

## System Overview

### Dynamics Model

Spacecraft motion was modeled using the nondimensional **Circular 
Restricted Three-Body Problem (CR3BP)** in the Earth-Moon synodic frame. 
The equations of motion are:

$$\ddot{x} - 2\dot{y} = -\frac{\partial \bar{U}}{\partial x}$$

$$\ddot{y} + 2\dot{x} = -\frac{\partial \bar{U}}{\partial y}$$

$$\ddot{z} = -\frac{\partial \bar{U}}{\partial z}$$

where the effective potential is:

$$\bar{U}(x,y,z) = -\frac{1}{2}(x^2+y^2) - \frac{\mu_1}{r_1} - \frac{\mu_2}{r_2} - \frac{1}{2}\mu_1\mu_2$$

The trajectory was initialized from a 200 km LEO with a Trans-Lunar 
Injection (TLI) burn of 3.345 km/s, producing a 2-day lunar flyby 
trajectory. Trajectory correctness was verified by confirming that the 
Jacobi constant (energy integral) varied by less than 1e-9 across the 
simulation.

### Measurement Model

Range and Doppler (range-rate) measurements were simulated from three 
**Deep Space Network (DSN)** ground stations:

| Station | Location |
|---|---|
| Goldstone | California, USA |
| Madrid | Spain |
| Canberra | Australia |

Station positions and velocities were computed by converting geodetic 
coordinates through ECEF and ECI frames into the CR3BP rotating frame, 
accounting for Earth's rotation. Measurement noise was modeled based on 
DSN two-way ranging specifications:

- **Range:** σ = 1 m
- **Doppler:** σ = 0.1 mm/s

The three stations provided near-complete coverage across the 2-day 
trajectory, with only a brief gap at the start when Earth blocks all 
antenna lines of sight.

---

## Estimation Methods

### Batch Least Squares (BLS)

BLS processes all measurements simultaneously to minimize a weighted 
sum of squared residuals. The nonlinear problem is solved iteratively 
using the Gauss-Newton method, accumulating the information matrix 
and right-hand side vector across all measurements at each iteration:

$$\Lambda_l = \Lambda_{l-1} + \left[\tilde{H}(\mathbf{x}^*_l)\Phi(t_l, t_0)\right]^T P_{vv,l}^{-1} \left[\tilde{H}(\mathbf{x}^*_l)\Phi(t_l, t_0)\right]$$

A **line search modification (BLSLS)** was added to improve convergence 
under poor initial conditions, scaling the Gauss-Newton step by a 
factor α ∈ (0, 1] to prevent divergence into infeasible states.

### Unscented Kalman Filter (UKF)

The UKF propagates a set of **2n+1 sigma points** through the nonlinear 
dynamics and measurement models without requiring Jacobians, making it 
well-suited for the highly nonlinear CR3BP environment. UKF parameters 
were set to α = 1 and κ = 0, with β = 2 for Gaussian distributions.

At each measurement step, sigma points are propagated through the CR3BP 
dynamics, the predicted mean and covariance are computed, and the 
posterior is updated using the Kalman gain. Unlike BLS, the UKF 
processes measurements sequentially and requires only a single pass 
through the data.

---

## Results

### Robustness to Poor Initial State Estimates

All three filters were tested with initial state deviations of 
**[10, 50, 1000] m** in position and **[1, 5, 100] m/s** in velocity.

| Filter | Max Deviation Handled | Final Position Error | Final Velocity Error |
|---|---|---|---|
| BLS | 50 m / 5 m/s | — | — |
| BLSLS | 1000 m / 100 m/s | 3.04 m | 0.0001 m/s |
| UKF | 1000 m / 100 m/s | 11.3 m | 0.0004 m/s |

The standard BLS failed to converge beyond the medium deviation level 
and fully diverged at 60 m / 6 m/s. The line search modification 
allowed BLSLS to handle even the highest deviation level. The UKF 
matched BLSLS robustness while running in approximately **half the time**.

### Robustness to Measurement Outages

Filters were tested by removing measurements within a time window 
[t_start, t_end]. The critical measurement region is the lunar flyby 
at approximately 38 hours, corresponding to 0.43–0.44 ND time units.

| Filter | Outage Window Handled | Compute Time |
|---|---|---|
| BLS | [0.003, 0.44] ND | ~12 s |
| BLSLS | [0.003, 0.43] ND (200 iterations) | ~24 s |
| UKF | [0.003, 0.44] ND | ~6 s |

Exceeding 0.44 ND in the outage window caused all filters to diverge, 
with position errors on the order of millions of meters. The UKF handled 
the larger outage window at one quarter the compute time of BLSLS.

### Overall Conclusion

The **UKF is recommended for real-time cislunar navigation** applications 
due to its combination of robustness, accuracy, and speed. The BLSLS 
achieves marginally lower estimation error and is preferred for 
post-processing applications where compute time is not a constraint.

---

## Tools & Technologies

- **Language:** MATLAB (BLS, UKF) and Python (CR3BP simulation, 
  measurement model)
- **Dynamics Integration:** ODE113 with adaptive step size 
  (RelTol = 1e-10, AbsTol = 1e-12)
- **Key Methods:** Gauss-Newton iteration, Cholesky decomposition, 
  unscented transform, Armijo-style line search, state transition 
  matrix propagation

---

## Links

- [Full Report (PDF)](/files/lunar_state_estimation.pdf)
- [Code Repository](https://github.com/yourusername/lunar-state-estimation)
