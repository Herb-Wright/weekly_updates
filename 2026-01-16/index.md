---
title: Herbie's Weekly Update 🗓️
date: 2026 Jan 16
bibliography: 2026-01-16/refs.bib
link-citations: true
---


## 1. Last Time

Last meeting, I was having some issues getting C3+ to work for the pivoting task. I also had some ideas for improving the robust CI-MPC algorithm we have been discussing. There were concerns about whether there was some sort of bug in the LCS and how to run the $K_t, v_t$ controllers at a different frequency than the MPC solve. In this write-up I make steps towards getting C3+ working on the cube pivoting task. 

The cliff-notes version of this write-up is:

- I don't think the "gap" was a bug, I think it really is an artifact of Anitescu friction
- I was able to *kind of* get C3+ to tip the cube, but it was hacky—perhaps there is evidence that linearization is just kinda bad for this task
- I have a collection of thoughts about running $K$ at a different frequency and the reweighting idea.
 

## 2. C3+ Cube Pivoting

### 2.1. Debugging The LCS

After about a day of debugging, I have come to the conclusion that there was not really a bug in the GIF from last time. Here is a cartoon of what I think was happening:

![My hypothesis for why no contact was made in the GIF](image-1.png)

Basically, because of the gap enforced by Anitescu friction approximation seems to cause the the ball to not have to contact the block in order to lift it. When I printed out the complementarity parts like distance, etc. it all seemed to check out. It seemed like the friction was really never sticking and was *always* sliding. The distance was about $\phi \approx 0.05$ at its largest when force was applied, which feels fairly reasonable. It should also be noted that as the cube tips, the LCS linearization will predict a smaller gap/distance than the actual gap/distance, thus making it seem even more off in the GIF from last time.

### 2.2. Debugging C3+ Algo and Costs

After looking into the LCS, I figured the best thing to do was simply *shrink* the cube in the LCS down and see if I can get a sort of proof-of-life that C3+ *can* do the tip up. My first few attempts, after tuning hyperparameters were kind of frustrating, as it seemed to be okay sometimes, and fail other times:

![Illustration of the bug I was dealing with](image.png)

I was able to fix the above by making some changes and solving a minor bug with the shrinking. This meant that I was finally able to get C3+ to tip the cube:

![C3+ successfully tips the cube! (yellow is beginning location for solve, green is trajectory of ball $q$'s from MPC solve)](./c3p_tips_cube_success.gif)

However, in order to get this to work I need a few pieces (changing any of these causes it to not work):

1. I had to shrink the cube for the LCS, specifically I had to shrink it all the way from side lengths of 0.3 to side lengths of 0.2—I tried 0.25 but it didn't work.
2. I had to add a PD controller to track the $q, v$ trajectory from the MPC solve alongside executing the $u$'s
3. I had to tune the hyperparameters obviously (like $\rho$); changing almost any of these by an order of magnitude causes failure (including changing $\Delta t$)

I don't really enjoy how sensitive it seemed to all these settings, even with such a toy problem as this. On one hand, this could be good motivation for a robust approach, on the other hand, it may signal that trying to get a more complicated procedure like robust control to work will be difficult. I am imagining that getting it working will at least require (a) having some sort of way to do the PD controller at a higher freq tracking states and (b) having something that can change the uncertainty distribution such that it finds that it needs to think the cube is smaller to work.

### 2.3 Lowering $\Delta t$

I decided to try to lower $\Delta t$, but I couldn't even get it to work when I changed the initial condition to very close to where you would need to be:

![Video of trying C3+ on the cube pivoting with a smaller $\Delta t$. It fails to complete the task in both cases.](./smaller_dt.mp4)

I fear that C3+ just isn't solving it good enough. Here are some graphs I generated:

![Graphs of C3+ first MPC solve for $\Delta t = 0.02$](image-2.png)

Perhaps this task is just too hard for C3+ to work? Like between the linearization and ADMM, maybe it just can't find reasonable plans? Like I am imagining the linearization might drive an intelligent solution to want to do something like the following:

![Illustration of what linearization might mean for the solution](image-3.png)

### 2.4. Back to $\Delta t = 0.1$

Just for good measure, I also tried the starting position for $\Delta t = 0.1$ and the cube being the true size (not shrunk). I don't think C3+ did particularly well. Here is a GIF of that:

![C3+ with the real block size cannot find a good solution.](./C3p_trash.gif)

It should be noted that complementarity did not get satisfied by this solve despite running 50 ADMM iterations. I also tried changing rho a bit and could not get any better. If you try to actually roll out the MPC closed loop, you get this:

![Rolling out the MPC closed loop cannot tip the block.](./not_good.mp4)

So, maybe there is something going on here. It could potentially be a hyperparameter issue. You can clearly see it kind of oscillating between modes around the corner, but it never wants to actually make contact in a way that tips the cube. I'm not sure exactly what the solution is here; it could be a fundamental linearization problem or a me-being-bad-at-hyperparameter tuning problem. 

**Shrinking the Cube 0.3→0.25:** Check out this behavior:

![When the cube is shrunk to 0.25, it tips the cube, then goes crazy.](./chaos.mp4)

I mean that kinda feels like I have a bug or ADMM is failing to solve at all or something. But also, why does it only happen like many timesteps in? My best guess is that it has something to do with linearizing around the corner of the shrunk cube when it is on the face of the big cube.

Perhaps the solution is to simply change the task? Here is the cube pivoting task which was done in the original C3 paper, that seems like it would be easier:

![Cube pivoting in C3 paper](image-4.png)

## 3. Other Thoughts

### 3.1. Thinking about $K$ for RC3+

Last time, there was concern about just grabbing the $K$ that was solved for by the robust method and using that at a higher frequency. If this is a problem, I think something like the following might be more stable:

1. Solve for $K$, regularized towards a PD controller $K_\text{pd}$
2. Then if $\bar x$ is the mean state and $\tilde K = K - K_\text{pd}$, we calculate the quantity $\hat v = v + \tilde K x_0 + K_\text{pd} \bar x$
3. Finally, we run the controller $u = K_\text{pd} (x - \bar x) + \hat v$ at the higher frequency

I think this could potentially resolve some of the problems with running $K$ at a higher frequency, but there still may be others stemming from the fact that $K_\text{pd}$ has the same gains at each frequency.

### 3.2. Thinking about Forward Sim of LCS

We will likely have to do forward simulation of the LCS for at least the async version of the algorithm. In such a case, we really care the most about speed. A common way to solve complementarity problems efficiently in physics simulation is to use projected Gauss-Seidal. This would not be very efficient on a GPU as it requires looping through each element of the complementarity constraint. We care about utilizing the GPU, so maybe we could use the projected Jacobi method, where we iterate:
$$ \lambda = \max\left(0, \lambda - \frac{F \lambda + q}{\text{diag}(F)} \right) $$
This operation, when jitted, could potentially have speed improvements over a QP solver, albeit sacrificing some accuracy.

I wonder if this could also be used by the reweighting idea to make it faster.
