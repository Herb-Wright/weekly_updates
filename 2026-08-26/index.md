---
title: Herbie's Weekly Update 🗓️
date: 2026 August 26
link-citations: true
---


## 1. Last Time

Last time was a while ago, but I was still struggling to get my method to work well in the push anything stack. In the past few weeks I have made significant progress. I detail the last week and a half or so here.

## 2. C3 vs ARCtIC in Simulation I

![ARCtIC vs C3 after some tuning, debugging, etc. in simulation](image.png)

Here are the results from this run:

![The actual final positions of each config](image-2.png)

I was able to identify a few problems from looking at the diagnostics. Overally, ARCtIC tended to leave pushes too early. I changed the hysteresis parameters and also updated the logic for continuing a push past 16 MPC iterations:

![Update to continuation logic](image-10.png)

## 3. C3 vs ARCtIC in Simulation II

After debugging and making some fixes, I decided to do a larger, 50-config experiment between the two of them. These were the 50 configs I used, in one big collage:

![50 configs for the experiment](image-5.png)

Then, here are the results:

![Bar plot of second experiment](image-3.png)

As you can see, my method actually does better than C3 here. That's good news! You can even plot the average letter pose error over time between the methods and get this:

![Mean pose error over time during trial (C3 vs ARCtIC)](image-4.png)

For completeness sake, I figured I should also show the actual experiments. Here is a video of the first third of the trials; sped up to like 50x speed:

![First third of trials at 50x speed](./20260820T223111Z_c3po_always_pose_admm_optuna_best_ridge1e-7_eevellim_hyst2x_r069_tuned50_rt1_c3_vs_arctic_sidebyside_all50_first10min_5x.mp4)

However, it is not all good news. Despite having a lower success rate compared to ARCtIC at each tolerance, C3 did seem to be faster when it came to time-to-goal among configs where both methods were successful:

![Box-and-whiskers plot for time-to-goal on both-success configs](image-6.png)

Here are some other differences between the runs:

![Differences between C3 and ARCtIC](image-7.png)

It did seem like ARCtIC resulted in shorter pushes, but during the seconds of the push, performed better than C3:

![Push improvement over time of the push. lines stop at median push length.](image-8.png)

The main reason for the shorter pushes is that the sampling C3 controller first potentially aborts due to no progress after 16 MPC iterations, not a specific amount of elapsed time. Because C3 runs at a slower rate, it takes longer to get through those 16 MPC iterations. The pushes are actually quite widely distributed; here is a plot of just 100 pushes per method, so you can see the spread:

![100 random pushes per method](image-9.png)

## 4. Towards Understanding Uncertainty

I wanted to start doing a bit of looking into the effect of uncertainty. I grabbed 200 random pushes from a previous experiment and tracked the improvement on them as a metric. The first thing I did was to try to optimize for the right $\alpha$ and uncertainty distribution. Here is a plot of some different values:

![Alpha/uncertainty tuning](image-11.png)

Once I settled on a value, I tried varying the drake model object-ground friction parameter, and got this:

![Changing the friction model](image-12.png)

Not that strong of results here, but it is all quite preliminary.

I decided to run a quick 20-config experiment using C3, my method, no feedback, and no uncertainty. Here are the results:

![Results from 20 config experiment.](image-13.png)