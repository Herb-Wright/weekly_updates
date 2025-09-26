---
title: Weekly Update
date: 2025 Sep 26
bibliography: 2025-09-26/refs.bib
link-citations: true
toc: false
---

## 1 Last Time


## 2 VLMs for Planning

### 2.1 Some Relevant Papers

VLMs have been used for planning (and related things) in a few papers. One fairly well-cited Deepmind paper used a VLM in tandem with a learned policy to generalize to objects not in the policy's training distribution [@stoneopen]. A 2025 ICRA paper [@zhi2025closed] proposes using a feedback loop where if failure of the plan is detected, they reprompt the VLM. This is done *after* the failed execution has already taken place. Also in ICRA was the BUMBLE [@shah2025bumble] paper, where a VLM takes the place of a planner. They prompt the VLM for multiple things and have a way of maintaining a "memory" that they divide into short and long term and put in the context window when prompting. They also use a *skill library*. Another paper that leverages VLMs that is well cited that I may have mentioned before is Rekep [@huang2025rekep], where a VLM proposes keypoint-based subgoals and constraints in python code, which are then used for a constrained optimization solver. They have a prompt for the VLM that is over a page long and kind of feels like cheating. I have also mentioned [@athalye2024pixels] before, where a VLM outputs predicates. TAMP meets VLMs in [@kumar2024open] as well (but I didn't really read it). Perhaps the most similar idea to what you mentioned last meeting is [@yang2025guiding], which I talk about in the next subsection. Another one that explores using VLMs for replanning is [@pchelintsev2025lera]. Some other uses of VLMs in robotics can be seen in [@patel2025real] and [@zhao2024vlmpc] among others.

### 2.2 VLM Prompting with Failure

Here are some screenshots from the [project page](https://zt-yang.github.io/vlm-tamp-robot/) of [@yang2025guiding], where they reprompt a VLM when the PDDL/Motion planning solver fails:

![](image.png){width=49%}
![](image-1.png){width=49%}

They tell the VLM that it failed, the (discrete) sub-goals that didn't work, as well as a set of objects that had collisions with the robot. The experiments kind of showed this re-prompting to be successful.

### 2.3 My Take on Your Idea



## 3 Cluttered Shelves

### 3.1 Some Relevant Papers

I have [previously done an overview of similar papers to this task](https://herb-wright.github.io/weekly_updates/2025-09-12/#existing-work). Here I simply mention a couple more papers to be aware of. 

### 3.2 


## References



