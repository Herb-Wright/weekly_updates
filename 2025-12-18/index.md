---
title: Weekly Update
date: 2025 Dec 12
bibliography: 2025-12-18/refs.bib
link-citations: true
---

## 1 Last Time

Last time, we talked about my hyperparameter tuning woes with C3+, the main takeaways were that (1) I should tune the cost to deincentivise a striking sort of shove at the beginning, and (2) I should investigate why not all the $u_0$'s were the same. We also discussed what the project could look like in the long term, including brainstorming what the real world experiment for the down-the-line paper would be.


## 2 Brainstorming Experiments

![Cartoon of some potential tasks](image.png)

**Pivoting with Shape Uncertainty:** In this task, the goal is to pivot the block (or whatever other object) with only 1 contact point, without completely tipping it over. When there is uncertainty about the shape, the contact points with the ground that are used for linearization change and affect the LCS dynamics of different trajectories. The goal with this experiment would be to show that steering improves success during the pivoting motion.

**Pushing with Friction Uncertainty:** In this task, the goal is to push the block (or arbitrary object) to a target location. I think this could even be 2d planar pushing instead of just in a straight line. If there is uncertainty about friction, we could arrive in failure modes where the planned push doesn't have enough force to overcome friction. The goal would be to show that steering allows for adapting to friction level. 

**Pushing with Center of Mass Uncertainty:** In this task, the goal is to push a taller block, where the end effector starts high. If there is center of mass and friction uncertainty, we would want the control strategy to be able to "go low" and push lower down on the block instead of on the top, where there is risk of toppling/tipping. The goal would be to show that a robust, steering approach recognizes when it needs to do this.

**"Lifting" with Shape Uncertainty:** In this task, there would be 3 actuated spheres surrounding an object with shape uncertainty. The idea is that by having an idea about the different possible shapes, a robust movement of the EEs could make the lifting operation better. 

**"Unstacking" with Friction Uncertainty:** In this task, there are two stacked blocks and the goal is to remove the second one while ensuring the top block doesn't move too far away in the xy-plane. In order to acheive this, the two spheres need to clamp down on the second block and allow friction to slide the top block off. The goal is to show that understanding uncertainty about friction helps plan better control strategies.

**Placing with Shape Uncertainty:** In this task, the block starts clamped by the two spheres, then must be placed in a narrow area, where it might require moving another block out of the way. If there is shape uncertainty, this could be difficult to do precisely. The goal would be to show that a robust approach, which reasons about the uncertainty, is better able to complete the action. I am less confident that an LCS would effectively be able to do this one, but perhaps there is something similar that would be better.

There are also other tasks that have been done in the literature, here is an image of some of them:

![Some other contact-rich tasks that have been done in the literature (from [@shirai2025hierarchical], [@jiang2024contact], and [@kang2025dexterous])](image-7.png)

## 3 Adventures with Robust C3+

### 3.1 Understanding the Algorithm

Recall that our algorithm basically alternates between:

1. Solving QP for $u, x, \lambda, \eta$ by ignoring $u^{(i)} = K x^{(i)} + v$ and $0 \leq \lambda \perp \eta \geq 0$ constraints (except for regularization in augmented Lagrangian)
2. Solving for $K, v$ and projecting $\lambda, \eta$ into the complementarity constraints

The very first thing I did was notice that some hyperparameters created NaNs during the algorithm. Upon inspection, I found that the QP step would occasionally return NaNs if the Shur complement matrix was poorly conditioned ($\kappa \approx 10^9$). This happened for some hyperparameter values, because the Shur complement matrix looked something like $S = -M_2 M_1^{-1} M_2^\top$. So if $M_1, M_2$ were poorly conditioned a little bit, $S$ would be even worse. Fortunately, there was an easy fix—just use float64's instead of float32's. So, I did that. I also made some changes where I moved the ball to the other side of the block so the expected $u$'s would be positive, I made the ball not have torque control (only xyz forces) and made the control cancel out gravity. 

The next thing I noticed after I was getting unintuitive results with RC3+ was that the first QP step gave kind of the opposite result of what we want. Ideally, we should see that the block doesn't go as far for higher friction values, and that the control strategy wants to push higher (bigger $u$) on $t=1$ (where $t$ starts at $0$). Instead, what happened was the opposite after the very first QP step, but after a lot of iterations, it would reverse:

![The first QP step outputs the opposite of what we want and it takes time to get to the good solution ](image-3.png)

After spending a bit being puzzled by this, I the best explanation that I came up with for what is going on is that the first QP step is using the wrong directions of the friction cone in the Anitescu formulation—it is using the forward pointing ones to push it towards the goal, and if $\mu$ is higher, it can use more force, and thus needs less force from the ball-block contact, explaining why the $x$ for $t=1$ was reversed. Here is a simplified graph to explain what is going on:

![After first QP step, the ball when low friction is present is pushed further than high friction ](image-2.png)

The next puzzle was the $u$'s at $t=1$. If you squint at the first image in this section, you will see that the $x$ for the ball is positive at $t=1$, which differs from the above image. This is potentially puzzling, but I think it comes down to how $\rho$ acting on $\eta$ works in the first QP step. At the first QP, the $\delta$ variables are set to 0, and in Anitescu's formulation, a penalty $\|\eta_k\|^2$ is similar to doing $\|\phi(q_k)\|^2$, where $\phi$ is the distance. This means that at the first QP step, there is a sort of force pulling the ball to be in contact with the block. Thus, we can see that how influential the $\rho$ hyperparameters can be.

### 3.2 The Cube Gets Pushed

With these insights in mind, I had the idea to first warm start the optimization by enforcing $K=0$ and $\rho$ is constant, so that I could cache the Cholesky decompositions for the QP step and run this about 10 times, before running the full algorithm, with varying $\rho$ and $K$ for about 5 iterations. Then I was able to, even if it is still kind of stochastic, get the sort of graph I was looking for.

![This is what we want to be seeing](image-4.png)

Here is a GIF of pushing the block:

![RC3+ pushing block.](./finally_kind_of.gif)

We can also compare the block error after 10 (s) from both a feedback and no feedback strategy:

![Positional error in meters after 10 (s) of each control strategy](image-5.png)

We see that there is a difference between having feedback and not having feedback, even if this is kind of a toy example and perhaps isn't that strong of evidence of anything. I should note that for the above example, it took about 0.1 (s) to solve and I was using 25 samples.

### 3.3 Potential Improvement?

I think we could enforce a known $u_0$ constraint and simply execute the first $K, v$ and try to predict how the world will change as the action is executed like shown in this diagram:

![Illustration of what I think might help](image-6.png)

The benefit of this is that you wouldn't have to roll-out 2 timesteps, you would always be just rolling out one. I also have a hunch that it (enforcing a $u_0$ constraint) could help with what I have been dealing with above.

Another thought I had is that we could resample our uncertainty at each ADMM iteration, which might prevent us from overfitting/becoming biased to the samples we have.

Also, would it ever make sense to use an even more simplified model, like a quasidynamic one (similar to [@pang2023global]) in order to make things faster/better behaved?

## 4 Other Stuff

### 4.1 Classes Next Semester

I have been approved for the CIS 7000 Robot Learning class, so I will be taking that next semester, which is on Monday from 1:45-4:45pm, **which conflicts with when lab meeting is currently**. I have also been told that STAT 9270 might be a good class to take (Bayesian Statistical Theory and Methods), but I am on the fence currently.


## References
