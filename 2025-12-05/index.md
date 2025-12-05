---
title: Weekly Update
date: 2025 Dec 05
bibliography: 2025-12-05/refs.bib
link-citations: true
---

## 1 Last Time

The last meeting was a couple weeks ago due to Thanksgiving break. We discussed the idea of incorporating the bilinear constraint into ADMM optimization. The past couple weeks for me have basically been getting things figured out for the class project. This week, I simply have a few figures that we can discuss.

## 2 Implementation of C3+ on the GPU (Using JAX)

I implemented C3+ [@bui2025push] in JAX. I used the Shur complement and Cholesky solve that we talked about. Because it is in JAX, it is easy to JIT compile and vmap. Here are the results from a timing test I did with random matrices to try to mimic the values in Push Anything's 4 object experiments:

![Timing results for my implementation of C3+ with random matrices](image-1.png)

As you can see it is very fast. I have created a repo with the code [here](https://github.com/Herb-Wright/c3p-jax).


## 3 RC3+ & C3+ on a Toy 1D Pushing System

In order to better understand the algorithms and such, I wrote down a very toy 1D quasi-dynamic pushing system (with friction). The system can be written as:
$$ \dot x_{1} = u - \lambda_1, \;\;\;\;\dot x_{2} = \lambda_1 - \lambda_2 $$
$$ 0 \leq \lambda_1 \perp x_2 - x_1 - d \geq 0 $$
$$ 0 \leq \mu - \lambda_2 \perp \lambda_1 - \lambda_2 \geq 0 $$
Then, I ran both RC3+ and C3+ on the system with a few different settings. Here is a gif of the result:

![Quasi-dynamic 1D pushing with friction. I test both C3+ and RC3+.](toy_quasidynamic_rc3.gif)

As you can see it all works. It should be noted that I run RC3+ by executing the first 4 timesteps solved from MPC. This can kind of be seen in the GIF, but may be worth discussing because I found that not doing this meant that RC3+ could also get stuck.

## 4 Timing RC3+

I also did a timing test on RC3+ with similar variables to Push Anything, here is that:

![Timing test for RC3+](image-2.png)

Once again, it is very fast. I changed the number of iterations to 4 to make it more fair to C3+, which does an additional QP step at the end—something that wouldn't work with RC3+.

## 5 Lab Presentation

I am signed up for the lab presentation on Monday, so I also thought it might be a good idea to ask you about slides/content. I have preliminary slides [here](https://docs.google.com/presentation/d/1ViBG_jvQvzFDr0zi8uiuuqMS9ZAq8PxpIOW3YRYq9Iw/edit?usp=sharing).

## 6 Next Steps

I do want to turn this into a project, and I have the implementation of the algorithm down now. The next steps in my mind are:

1. Getting it to work in Mujoco (3D finally)
2. Thinking of some good tasks to showcase

I still think ultimately that doing some sort of grasping with shape uncertainty would be the coolest. 

## References

