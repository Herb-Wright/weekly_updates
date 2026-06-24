---
title: Herbie's Weekly Update 🗓️
date: 2026 June 24
bibliography: 2026-06-24/refs.bib
link-citations: true
---


## Hyperparameter Tuning Struggles

I am trying to hyperparameter tune my method. Here are the cases I am using:

![tuning cases used](image-2.png)

However, I can't seem to find a great controller.

![video of hyperparameter tuning cases](./20260622Tregen_seed829174563_push_anything_best_t1642_14cases_styled.mp4)

![another video of hyperparameter tuning cases](./20260623Tn8_n3i2_t526_16cases_tune_costs_ee_goals.mp4)

Here is me trying to visualize open-loop rollouts on random EE samples:

![Perhaps a bug](image.png)

![Also perhaps a bug](./20260624Tn8_t1661_c3po_closed_loop_best_t1661_c3po_dairlib_ee_closed_loop_grid_2x4.mp4)

I can't seem to clearly find a bug in the LCS, but maybe I just need to take a bit of time and regroup:

![LCS visualization](image-3.png)

I was able to get the speed to be pretty much real-time by moving to 3 initialization iterations and 2 full iterations.

I also have this push "T" image I am trying to use for debugging purposes:

![Push T](image-1.png)

**Next:** I concluded that my current cases are not that good, and have collected a set of real examples of (mostly good) EE samples from running push anything with C3:

![New Cases](image-4.png)

I want to use these for hyperparameter tuning.
