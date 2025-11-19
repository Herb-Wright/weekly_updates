---
title: Weekly Update
date: 2025 Nov 121
bibliography: 2025-11-21/refs.bib
link-citations: true
---

## 1 Last Time

Last time, we talked about a few things, but mostly about how to solve the sample-based CI-MPC problem more efficiently.

## 2 Registration Update

## 3 ADMM for Robust Contact-Implicit Control

Previously, ADMM [@boyd2011distributed] has been applied for contact-implicit control in C3 [@aydinoglu2024consensus]. In this section, I go over a formulation for applying ADMM in the sampling-based robust control setting.

### 3.1 No Feedback

Consider the robust control problem over uncertain model parameters $\xi$:
$$ \text{arg}\min_u \mathbb E_{\xi} \left[ C_\xi(u_{[T]}) \right], $$
where $C_\xi(u_{[T]}) = \sum_t C_t (x_t, u_t; \xi)$, with each $C_t$ being convex and quadratic in $x_t, u_t$, and $x_t$ is determined by rolling forward dynamics from $x_0$ with control inputs $u_t$ and simulator parameters $\xi$. We are specifically concerned with the case where this is a probabilistic version of the contact-implicit control problem formulated as follows:
$$ \text{arg}\min_{u, x, \lambda} \;\; \sum_t x_t^\top Q x_t + u_t^\top R u_t $$
$$ \text{s.t. }\;\; x_{t+1} = A x_t + B u_t + D \lambda_t + d $$
$$  0 \leq \lambda_t \perp E x_t + F \lambda_t + G u_t + c \geq 0 $$
We want to consider the problem where we approximate this control problem with a set of samples over unknown parameters $\xi$, giving us the following:
$$ \text{arg}\min_{u, x^{(i)}, \lambda^{(i)}} \;\; \sum_{i, t} {x_t^{(i)}}^\top Q x_t^{(i)} + {u_t}^\top R u_t $$
$$ \text{s.t. }\;\; x_{t+1}^{(i)} = A^{(i)} x_t^{(i)} + B u_t + D^{(i)} \lambda_t^{(i)} + d^{(i)} $$
$$ 0 \leq \lambda_t^{(i)} \perp E^{(i)} x_t^{(i)} + F^{(i)} \lambda_t^{(i)} + G^{(i)} u_t + c \geq 0. $$
This optimization problem is potentially prohibitively expensive. We can speed it up by using ADMM on an alternate formulation:
$$ \text{arg}\min_{v, u^{(i)}, x^{(i)}, \lambda^{(i)}} \;\; \sum_{i, t} {x_t^{(i)}}^\top Q x_t^{(i)} + {u_t^{(i)}}^\top R u_t^{(i)} $$
$$ \text{s.t. }\;\; x_{t+1}^{(i)} = A^{(i)} x_t^{(i)} + B u_t^{(i)} + D^{(i)} \lambda_t^{(i)} + d^{(i)} $$
$$ 0 \leq \lambda_t^{(i)} \perp E^{(i)} x_t^{(i)} + F^{(i)} \lambda_t^{(i)} + G^{(i)} u_t^{(i)} + c \geq 0. $$
$$ v = u^{(i)}, $$
where we have introduced new decision variables and corresponding constraints. However, this means each sample is pretty much decoupled from each other except for that last constraint. We can also consider the augmented Lagrangian of the above problem, where we do the same trick as C3 of lifting all other constraints into the objective (with $z = [u, x, \lambda]$ (each sample)):
$$ \mathcal L_\rho (z, v, y) = c(z) + \mathcal I_\mathcal{D}(z) + \mathcal I_\mathcal{H}(z) + \sum_i \left[{y^{(i)}}^\top (u^{(i)} - v)  + \rho \|u^{(i)} - v\|^2 \right] $$
or if we want the scaled version of the augmented Lagrangian:
$$ \mathcal L_\rho (z, v, y) = c(z) + \mathcal I_\mathcal{D}(z) + \mathcal I_\mathcal{H}(z) + \sum_i \rho \left[\|w^{(i)} + u^{(i)} - v \|^2  - \|w^{(i)}\|^2 \right] $$
It should be noted that only the last term (the squared term) depends on $v$. Thus, we can consider doing a ADMM algorithm like so:

**Repeat:** (Initialize $v, w$)

1. $z = \text{arg}\min_z \mathcal L_\rho (z, v, w)$  *// for the first run, ignore the $v$ terms in objective*
2. $v = \text{arg}\min_v \mathcal L_\rho (z, v, w)$
3. $w^{(i)}_{k+1} = w^{(i)}_k + u^{(i)} - v$

**End**

It should be noted that 1. above is simply an augmented version of the original contact-implicit problem, solved per sample, and that 2. is simply a very easy least-squares problem. We can use batched C3 for 1. and quickly solve 2. analytically, which has the following form:
$$ \text{arg}\min_v \mathcal L_\rho (z, v, w) = \text{arg}\min_v \sum_i \|w^{(i)} + u^{(i)} - v \|^2  $$
$$ = \frac{1}{n} \sum_i u_i + \frac{1}{n}\sum_i w_i $$



### 3.2 With Feedback Gains and C3+




<!-- The next step is to say that our $u_t$ *can* be different for each $\xi^{(i)}$ sample, but only through a feedback term on $x_t$: -->

## References


