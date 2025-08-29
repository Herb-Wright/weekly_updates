---
title: "Meeting Prep: Thoughts on the Future of Robotics"
date: 2025 August 29
bibliography: 2025-08-29_posa_meeting/refs.bib
link-citations: true
toc: false
---

# 

*I have not (and will not) proofread this. This is mostly just me rambling in order to get my thoughts somewhat in order on the future of robotics.*

# 1 Introduction

Excitement abounds in robotics at the current moment. Piggy-backing on lessons from the related field of ML, data-driven approaches to many manipulation problems have produced impressive and promising demos. Approaches like VLAs (pi0 [@black2410pi0] from Physical Intelligence, LBMs from TRI [@lbmtri2025] come to mind), Reinforcement Learning ([@lum2024dextrahg] is RL applied to dexterous grasping that Martin (postdoc at Utah) worked on) are some examples. We have also seen large models trained on internet-scale data from fields like NLP and CV become commonplace as components in systems proposed to solve various manipulation sub-problems. I have personally found Grounded SAM [@ren2024grounded] and CLIP [@radford2021learning] to be helpful in my own research. It is more and more common to hear sentiments such as ["more data solves robotics"](https://www.youtube.com/watch?si=UWme-89_Za054aeA&t=196&v=PfvctjoMPk8&feature=youtu.be), pointing to things like the "bitter lesson" [@sutton2019bitter] or scaling laws [@hestness2017deep; @kaplan2020scaling] in other domains. One might ask if, in 2025, there is any reason to be studying *model-based* approaches to certain manipulation problems when all you really need is a bigger GPU (TPU?) and some more data. 

In this essay/writeup, I want to make the case that in order for robotic manipulation systems to be deployable, it will require incorporating strong domain-knowledge priors about the structure of robotics problems into learning-based systems. I mean this in a broad sense—that what we need is a fusion of learning-based and model-based approaches. I want to sketch out a few interesting directions with this idea, and perhaps propose a longer-term vision for what my PhD might be spent studying and attempting. 

# 2 Why We (Still) Need Models

In short, the word that best describes my view here is *robustness*. The robustness of a robotic manipulation system is critical to its deployability, and I believe model-based approaches and priors are the most promising direction for imbuing learned systems with adequate levels of robustness. There are two ingredients in a successful learning method: good data and good priors. Robotics could use a bit more of both. I wrote [a blog post](https://thoughts.herbiewright.com/posts/free_lunch_in_robotics/) arguing that we already and should do more of intelligently using the inherent structure of robotics problems in our learning-based solutions (here I mean any system that has learning-based components, not just end-to-end learned systems). The structure I talk about in the post is 3D geometry and physics/dynamics. By exploiting this structure we can make our systems more robust and efficient. In the planning domain, a interesting paper that perhaps showcases this is the pix2pred [@athalye2024pixels] work that leverages VLMs to extract predicates from observations, allowing for much better robustness and efficiency compared to simply using a VLM to do the planning.

# 3 Some Potential Ideas

**Idea 1:** Using model-based systems to "guide" learned policies (perhaps similar to [@li2024language]). Or the other way around: using learned policies to "guide" model-based approaches to planning/control (the dumbest version is to use a learned policy to initialize MPC rollouts or something). Of course being able to do this requires being able to infer accurate models of the environment/task during inference-time, something that my first proposed project is meant to help with—and also think about uncertainty.

**Idea 2:** Using model-based systems to give useful information during the training of learned components. For example, if you had an accurate dynamics model, you could maybe infer contact points and use this information as input to a learning-based system. Of course, relatedly, RL these days always uses a model—the simulator! There are pretty mature simulation techniques out there, but not everything is modeled perfectly. Perhaps there are improvements to how the simulators work that might make RL produce more robust policies? Perhaps those same techniques could reduce the sim-to-real gap in real-world MPC applications?

**Idea 3:** Using model-based approaches to generate large amounts of better simulation data for learning-based approaches. This is related to the second idea. It would be along similar lines as the popular digital twin work in the field at the moment—again closely related to my first proposed project. If we could have a technique that could take in a video or something, and build a large amount of diverse environments, this could really help with data diversity in learned methods. I recently read [@li2025pin], which uses differentiable simulation to construct physically accurate single object models for a simple pushing task (digital twins), then they show you can perturb the physics parameters to generate digital cousins, which can act as domain randomization for learning methods.

# 4 The Future of the Field

When I try to predict where the field will be in 5 years, I do think that data will play a crucial part, however, I struggle to see the current large robotic datasets getting us anywhere near deployability with the current popular learning-based methods. I think understanding the modeling and structure within robotic manipulation problems (geometry, physics) is crucial to develop systems that can be as efficient and effective as possible with the data that they have. When I imagine what the state of the art manipulation system is in 5 years, I think it is a system that involves a lot of learning, but is closely integrated with either the explicit or implicit structure of its manipulation problems—maybe from physics/geometric constraints during inference-time planning or using accurate, diverse, simulation during training. 

# References




