---
title: Weekly Update
date: 2025 Oct 17
bibliography: 2025-10-17/refs.bib
link-citations: true
---

## 1 Last Time

Last time, we discussed a few different ideas; namely the VLM idea we have been talking about and some robust MPC stuff. This time, because my computer came, I didn't have any time to do any deep dives, as much of my time was spent setting up my new computer. However, now that I do have my computer, I want to start working on my original idea I pitched over the summer.

## 2 Towards my First Project Idea

### 2.1 The Pitch

Last time I had a high-level overview of the pitch:

> - **(Potential) Title:** *Diverse and Physically Stable Bayesian World Models for Manipulation*
> - **Motivation:** The ability to convert partial observations of a scene into reasonable estimates for the scene's dynamics is incredibly useful in robotics. In the case of robotic manipulation, this usually means reconstructing the geometry of each object in the scene along with some physics parameters. The reconstructions ought to be physically stable and capture the diversity/uncertainty from occlusion. Such diverse, physically stable reconstructions from a single RGBD image can be used downstream by controllers for robotic manipulation.
> - **Method:** We can do Bayesian reconstruction using a BundleSDF-like [@wen2023bundlesdf] ensemble. Use Amodal3R [@wu2025amodal3r] with some modulation as a prior, simplified probabilisitic ContactNets [@pfrommer2021contactnets] loss for physical stability. Then use likelihood from observation similar to the negative sampling in [@wright2024v].
> - **Experiments:** We can perform simple pushing tasks to verify that our model is *accurate* via multi-object pushing, and *diverse* via pushing with some occlusion that creates multiple different possible outcomes from the same push.

The key idea is to combine physical stability with diverse, probabilistic object reconstructions. There were two motivating examples I have for it:

![Motivating examples where diverse reconstruction affects physics and physical constraints](image.png)

Then, I also had a figure for the proposed method:

![overview of proposed method](image-1.png)

The mathematical formulation of things can be found in depth in my [original pitch](../2025-06-11_project_pitch).

Obviously, we didn't get around to discussing this last meeting, but there are a few changes since the original pitch; namely the idea to use Amodal3R + bounding box randomization to get diversity, which maybe has a similar insight to [@saund2021diverse], but in 2D instead of 3D.

**Question:** *How does this pitch sound? What would strengthen it?*

### 2.2 Initial Steps

The first step, in my estimation, is to get Amodal3R [@wu2025amodal3r] working on my new computer. There is [a github repo](https://github.com/Sm0kyWu/Amodal3R) for it, which I have cloned, but I did not have time to actually get around to resolving all the dependency things to try it out. I had a sort of roadmap set out last time, and here were the first few of the items:

> 1. Getting Amodal3R+registration running (Minghan probably already has this, but I will want it working for me)
>     1. Create a few example images
>     2. Get Amodal3R running on new GPU
>     3. Get registration working
>     4. Get Amodal3R working with randomized bounding boxes
> 2. Training a Bayesian ensemble implicit neural model with Amodal3R and depth supervision
>     1. Negative sampling
>     2. Bayesian ensemble
> ...

**Question:** *How do we feel about just going for it on this project?*

There are probably going to be problems that pop up along the way, but I would like to preemptively aim for maybe like IROS next year deadline

## 3 Brainstorming a MEAM 5170 Project

I think I might want to do something related to robust control for my MEAM 5170 final project. I am curious what you think. I can think of a few different directions to do, all of which are related to the particle-based stuff discussed previously:

- Do some sort of particle-based robust pushing by first having something like TRELLIS [@xiang2025structured] create multiple reconstructions and picking random guesses for center of mass and frictional and inertial properties, then doing the particle-based contact implicit control. I would have to use a very small number of particles, and it might be pretty slow.
  - This could be made more complicated by doing further sysID during the execution of the push.
- Do the particle-based contact implicit control, but do some sort of consensus (related to [@aydinoglu2024consensus]) with the QPs as well in order to be more efficient. I could also consider doing some sort of feedback thing.

The second bullet (and this whole section in general) is very related to the ESE talk this past week. 

Here is the formulation that I had a couple weeks ago for the particle-based robust MPC, but with the complimentarity constraint:
$$ \min_{u, \{x_i\}, \{\lambda\}} \frac{1}{m} \sum_{i=1}^m \left[\sum_{k=0}^{n-1} \left({x_k^{(i)}}^\top Q_k x_k^{(i)} + u_k^\top R_k u_k\right) + {x_n^{(i)}}^\top Q_n x_n^{(i)}\right] $$
$$ \text{s.t. } \; 0 \leq \lambda_k^{(i)} \perp g(x_k^{(i)}) \geq 0$$
$$ \;\;\;\;\;\; \text{and some linear constraints}  $$



<!-- [@drnach2021robust] -->

## 4 OpenAI API Keys

This section just has some info from when I briefly looked into OpenAI API keys. Here are some of the popular OpenAI models and their pricing:

| Model      | Input $/1M Token | Output $/1M Tokens |
| ---------- | ---------------: | -----------------: |
| GPT-5      |             1.25 |              10.00 |
| GPT-5 pro  |            15.00 |             120.00 |
| GPT-5 nano |             0.05 |               0.40 |
| GPT-4o     |             2.50 |              10.00 |

It should be noted that the token prices are for *text* tokens, and images would likely have different pricing. There seems to be [decent docs](https://platform.openai.com/docs/quickstart/build-your-application) on connecting to the API via python. I made an API key and tried to call the OpenAI API and got the following response:

```json
{
  "error": {
    "message": "You exceeded your current quota, please check your plan and billing details. For more information on this error, read the docs: https://platform.openai.com/docs/guides/error-codes/api-errors.",
    "type": "insufficient_quota",
    "param": null,
    "code": "insufficient_quota"
  }
}
```

But you can easily add payment methods and set usage limits in their dashboard:

![OpenAI dashboard for billing info](image-2.png)



## References

