# Tseng et al. 2026 GRL — Toy Model: Liouville / KDE Framework

This folder contains a self-contained toy model that replicates the
probabilistic framework described in **Tseng et al. (2026, GRL)**
using a simple 1-D scalar ODE instead of the full atmospheric model.

---

## System

$$\frac{dx}{dt} = v(x) = x - x^3$$

| Property | Value |
|---|---|
| Stable fixed points | $x = \pm 1$ |
| Unstable fixed point | $x = 0$ |
| Pitchfork bifurcation | — |

The velocity field $v(x) = x - x^3$ has analytical divergence

$$\text{div}(x) \equiv \frac{\partial v}{\partial x} = 1 - 3x^2$$

- Near $x = 0$: $\text{div} > 0$ → trajectories **diverge** → density decreases
- Near $x = \pm 1$: $\text{div} < 0$ → trajectories **converge** → density increases

---

## Methodology (from Tseng et al. 2026)

### 1. Ensemble Propagation
$N$ members are initialized at $t = 0$ from a prescribed distribution
and integrated forward using **RK4** with a small time step $\Delta t$.

### 2. Liouville Density Update (Lagrangian)
The continuity equation for probability density in phase space is the
**Liouville equation**:

$$\frac{\partial \rho}{\partial t} + \frac{\partial (v \,\rho)}{\partial x} = 0$$

Along each trajectory $x_j(t)$, the density obeys:

$$\frac{d \rho_j}{dt} = -\rho_j \,\frac{\partial v}{\partial x}\bigg|_{x_j}$$

whose formal solution (one step, constant divergence) is:

$$\rho_j(t + \Delta t) = \rho_j(t)\,\exp\!\left[-\left.\frac{\partial v}{\partial x}\right|_{x_j(t)} \Delta t\right]$$

In practice the exponent is accumulated over all steps.  After each step
the weights are renormalised so they sum to 1.

### 3. Kernel Density Estimation (KDE)

An Eulerian grid density is reconstructed from the ensemble using a
Gaussian kernel:

$$\hat{\rho}(x, t) = \frac{1}{N}\sum_{j=1}^{N} \frac{1}{\sqrt{2\pi}\,h}
    \exp\!\left[-\frac{(x - x_j(t))^2}{2h^2}\right]$$

The **Liouville-weighted KDE** replaces the uniform weight $1/N$ with
the Lagrangian weight $\rho_j(t)$:

$$\hat{\rho}_{\rm Liou}(x, t)
    = \sum_{j=1}^{N} \rho_j(t)\,\frac{1}{\sqrt{2\pi}\,h}
    \exp\!\left[-\frac{(x - x_j(t))^2}{2h^2}\right]$$

The analytical time derivative of the standard KDE (used in the paper's
SI for the divergence cross-check) is:

$$\frac{\partial \hat{\rho}}{\partial t}\bigg|_{\rm KDE}
    = \frac{1}{N}\sum_{j=1}^{N} \frac{x - x_j}{h^2}\,
      \frac{1}{\sqrt{2\pi}\,h}
      \exp\!\left[-\frac{(x-x_j)^2}{2h^2}\right] \dot{x}_j$$

### 4. Shannon Entropy

$$H(t) = -\int \hat{\rho}(x,t)\,\ln \hat{\rho}(x,t)\,dx$$

Entropy is anchored to a reference time $t_{\rm ref}$ (typically $t = 0$)
and the change $\Delta H(\tau) = H(\tau) - H(t_{\rm ref})$ is tracked.

As the ensemble collapses towards the two stable fixed points $x = \pm 1$,
$\Delta H$ becomes **negative** (reduced uncertainty).

### 5. Hovmöller PDF Diagram
A Hovmöller diagram shows $\hat{\rho}(x, t)$ as a 2-D color field
with $x$ on the horizontal axis and lead time $t$ on the vertical axis,
analogous to Fig. 6 in the main manuscript.

---

## Files

| File | Description |
|---|---|
| `toy_liouville_cubic.ipynb` | Main notebook: ensemble integration, Liouville weights, KDE, entropy, Hovmöller |
| `README.md` | This file |

---

## Quick start

```bash
cd Tseng_et_al_2026GRL
jupyter lab toy_liouville_cubic.ipynb
```

Run all cells in order.  The notebook is self-contained and requires only
`numpy`, `scipy`, and `matplotlib`.

---

## Connection to Paper

| Paper concept | Toy-model analogue |
|---|---|
| Barotropic model trajectory $\mathbf{x}(t)$ | Scalar trajectory $x(t)$ |
| Divergence $\nabla \cdot \mathbf{v}$ | $\partial v / \partial x = 1 - 3x^2$ |
| Ensemble-mean KDE PDF | $\hat{\rho}(x,t)$ (uniform weights) |
| Liouville-corrected PDF | $\hat{\rho}_{\rm Liou}(x,t)$ (Lagrangian weights) |
| Hovmöller PDF diagram | Same, for 1-D system |
| $\Delta H(\tau)$ entropy | Same definition |
