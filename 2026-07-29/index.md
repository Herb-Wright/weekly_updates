---
title: Herbie's Weekly Update 🗓️
date: 2026 July 29
bibliography: 2026-07-29/refs.bib
link-citations: true
---


## 1. Last Time


## 2. New Hyperparameters

I landed on some new hyperparameters to use:

```json
{
  "q_obj_pos": 5000,
  "q_obj_ori": 500,
  "q_obj_vel": 0,
  "q_ee_pos": 0,
  "q_ee_vel": 10,
  "rho_mult": 3,
  "rho_lambda": 200,
  "rho_eta": 5,
  "rho_EE_last_iter_mult": 10,
  "rho_x": 20,
  "rho_u": 10,
  "R_val": 10,
  "alpha": 1000,
  "kp_ee": 11.4,
  "kd_ee": 1.14,
  "schur_ridge": 1e-09
}
```

And make some minor changes to how things execute/operate, including "zeroing" the LCS around the current position. Regardless, here are some visualizations:

![ADMM visualization of tuning cases.](./20260728T_alpha1k_rhox20_q5k_schur_f32_gravfix_admm_iteration_16cases.mp4)

![Open-loop LCS rollouts for the tuning cases.](./20260728T_alpha1k_rhox20_q5k_schur_f32_gravfix_lcs_cost2_rollout_16cases_grid.mp4)

![Mujoco closed-loop rollouts on the tuning cases](./20260728T_alpha1k_rhox20_q5k_schur_f32_gravfix_arctic_pd_mujoco_closed_loop_16cases_grid.mp4)

As for the cost calculation, here is a visualization of it assigning costs to different EE samples to make sure that that works as well:

![Visualization of EE sample costs](./20260728T_alpha1k_rhox20_q5k_schur_f32_gravfix_live_lcm_ee_improvement_overlay_pd_kp20_kd0p5.png)


## 3. Transferring to the Push Anything Example

I also tried to transfer these hyperparameters to the actual dairlib push anything stack. This required fixing some bugs, then I eventually got the following:

![Sped up video of push anything with my backend](./sped_up_push_anything_arctic.mp4)

As you can see, it ended in a clear failure. I generated a GIF that zooms in and slows down on that push so you can see what the behavior was leading up and during the push: 

![Push anything failure at the end](./slowed_down_failure.gif)

As you can, see there was some weird behavior, but it is not immediately obvious to me what happened. It turns out that this occassional "explosive" push happens sometimes with my setup, so I need to debug it. Here is another example that I found programmatically:

![Example of explosive push](./20260729T160330Z_explosive_push.gif)

### 3.2. Update

I have fixed what I think was the bug (it still seems like there might be something wrong, but I have mitigated at least). And now I have this performance on push anything:

![Latest Push anything with my backend](./new_arctic_push_anything.mp4)

