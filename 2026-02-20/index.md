---
title: Herbie's Weekly Update 🗓️
date: 2026 Feb 20
bibliography: 2026-02-20/refs.bib
link-citations: true
---


## 1. Last Time

Last time, we mostly talked about how the ~70% success rate was not good enough. We had some ideas about enforcing that the contact happens in a sort of *trust region* (different from the contact trust region [@suh2025dexterous]). This week, I tried to implement a simple constraint for the cube, but found that it was not that beneficial, but moving to Float64's was.

## 2. Implementing State-space Constraints/Projection for Cube/Ball

The main mathematical justification is to add an extra constraint to the MPC problem:
$$ A_\text{cube} x \leq b_\text{cube} $$
Because this is not an equality constraint, it makes sense to implement the constraint in the $\delta$ step. Then, the $\delta$ step would have 3 parts:

1. Ridge regression for $K, v$
2. 1D MIQP for complementarity $\lambda, \eta$
3. *Project* $x$ into the constraint above

I decide to implement this projection by only projecting the end effector position into the constraints (a full projection would involve projecting the cube). Here is a gif to verify that my implementation of the projection is correct (I do the projection after each step step while applying a force/torque somewhere):

![Testing projection operation into state-space constraint](./testing_projection.gif)

The next thing to do was throw it into the ADMM algorithm and see how it performed. I noticed two different things: (a) it didn't really seem to work off the bat; and (b) changing hyperparameters caused numerical issues. Specifically, I was trying to change the $\rho_x$ hyperparameter to increase constraint satisfaction. This non only results in NaNs, but even if NaNs don't happen, the EE goes a bit wild:

![The added constraint, when $\rho$ is changed to incentivize the constraint, results in numerical instability](./numerical_instability.gif)

I decided to move to float64, but that didn't really solve it; If $\rho$ was too big, then the end effector would either not want to move, or it would result in numerical issues even with the higher precision. If $\rho$ was too small, then the ADMM algorithm would effectively *ignore* the constraint and still go off the end of the cube. My hypothesis is that the QP step has to try to balance the actual objective $Q$ and the ADMM matching for $x$, based on $\rho$, that tries to satisfy the constraint $A_\text{cube} x \leq b_\text{cube}$. Unfortunately, I think that using float64s is not a viable solution to this problem. 

**Experiment:** I used `float64`s and made minor hyperparameter changes (using $\rho_x=1.0$, which is low and what I used last week). Then I compare the difference of running the algorithm with and without the end effector projection on 300 randomized runs. I apply each change one after each other and record the results:

![Results from experiment. Error bars are 95% Wilson intervals. Note that y-axis starts at 0.5.](image.png)

As you can see, the projection really doesn't help. I tried changing $\rho$ to be bigger for both just the end effector and for the the entire state $x$. These all performed much worse or ran into numerical issues. It does look like the new hyperparameters help, and moving to Float64 improves performance (which suggests the conditioning of our problem might be a major issue). It should be noted that using Float64's means the algorithm takes about 5x as long.

## 3. Reweighting Revisited

I revisited the reweighting stuff, and saw that even using Float64's, it still did not result in performance improvement:

![Reevaluating reweighting. The prior distribution used was `Unif[0.23, 0.38]` for box side lengths](image-1.png)

My two best guesses is that this is due to either:

1. The way reweighting is put into the algorithm just makes it hard to actually use the reweighting effectively and just results in more difficult optimization
2. The hyperparameters being tuned without reweighting makes it difficult to see performance improvement with reweighting

## References









