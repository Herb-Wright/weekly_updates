---
title: Weekly Update
date: 2025 Dec 12
bibliography: 2025-12-19/refs.bib
link-citations: true
---

## 1 Last Time

Last time, we talked about my hyperparameter tuning woes with C3+. We also discussed what the project could look like in the long term.


<!-- ## 2 Searching for The Example

> In the future of robotic manipulation off the assembly line, robots will encounter new objects in their environment and be expected to perform useful tasks with them, such as cooking with kitchen utensils, using tools, opening doors, and packing items. Model-based control methods work increasingly well in contact-rich scenarios [1, 2], but rely on models of the manipulated objects

In the above quote, citations 1 and 2 are , the C3 line of work from the lab. -->

## 2 Towards a Paper

![Cartoon of some potential tasks](image.png)

**Pushing with Friction Uncertainty:**

**Pivoting with Shape Uncertainty:**

**Pushing with Center of Mass Uncertainty**

**"Lifting" with Shape Uncertainty**

**"Unstacking" with Friction Uncertainty**

**Placing with Shape Uncertainty**

## 3 Updates on Cube Thing

The first thing I did was move to Float64's because my shur complement matrix was poorly conditioned, leading to NaN's with certain hyperparameter choices. Then I thought I was getting somewhere, but I noticed something undesireable happening.

I think this is the best explanation that I came up with for what is going on:

![After first QP step, the ball when low friction is present is pushed further than high friction.](image-1.png)



