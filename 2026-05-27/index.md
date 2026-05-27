---
title: Herbie's Weekly Update 🗓️
date: 2026 May 27
bibliography: 2026-05-27/refs.bib
link-citations: true
---

## 1. Some Figures

These were in the poster, so you already saw them, but here are the latest results:

![Latest Trifinger results](image-1.png)

I have been trying to get the PETS-Mj baseline working, but it seems to be far too slow:

![Comparison of MJPC and PETS-Mj on Trifinger combined uncertainty. For PETS-Mj, N=8 models were used in the ensemble and 3 CEM iterations were performed.](image.png)

## 2. Push Anything Tuning

I have also been trying to tune for push anything, but that has proven difficult. Here is a video of the MPC returned trajectories:

![The MPC plan sucks.](./20260527T173823Z_push_anything_global_best_t760_mpc_plan_t0p4.mp4)

However, I think I discovered a bug because I was trying to copy the dairlib push anything parameters, I think I copied `n_friction_directions=2` from dairlib, but in my code it really should be 4 because I count friction directions differently.


## 3. Printed Poster

I printed out the poster for ICRA:

![ICRA CR2 Poster](image-2.png)
