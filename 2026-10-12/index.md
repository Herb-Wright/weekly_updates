---
title: Herbie's Weekly Update 🗓️
date: 2026 October 12
link-citations: true
---

## 1. Center of Mass

My first experiment here was to shift the center of mass along the vertical stem of the "T". I originally did a couple runs that looked promising, where ARCtIC seemd to outperform ARCtIC-D:

![Push "T" with **center-of-mass** uncertainty](image-1.png)

I generated new configs and tried to reproduce and I got:

![Weekend push "T"; center of mass uncertainty part 2](image.png)

Where the difference kind of just disappears. The time-to-goal is also not signficant.

## 2. Changing the Shape

My second experiment is about changing the shape of the "T", but by randomizing the stem. Here is what draws from the uncertainty distribution look like:

![Draws from the uncertainty distribution](image-2.png)

I tested this distribution on the nominal and version of the experiment where the stem is shifted 2 cm in either direction. Here are the results:

![Results](image-3.png)

As you can see, not much difference

## 3. Shape Scale

Finally, here are some shape scale results. They don't look promising at $N=50$, as there is way too much noise:

![Shape scale part 1 (N=50)](image-5.png)

But, they almost look promising in a later $N=200$ run I did of just the nominal and 0.8x version:

![Shape scale part 2 (N=200)](image-4.png)

Note that 86% to 77% is significant at $N=200$.