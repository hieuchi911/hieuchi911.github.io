---
layout: page
title: SPHRED for Dialog modeling
description: Rebuilding a Conditional Variational Framework for Dialog Generation
img: assets/img/work4_noncollapse.gif
importance: 4
category: work
---

This is a research project for my graduation thesis at International University, Vietnam National University - HCMC.

SPHRED is a combination of Conditional Variational Autoencoder and Hierarchical Recurrent Encoder Decoder for controllable dialog generation. 

To gain a better understanding of the VAE optimization, I used PCA to visualize in 2D the prior and posterior distributions while training, and observed that as posterior collapse happens (the 2 distributions move away from each other), the model starts to generate repeating tokens (greedy sampling).

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/work4_collapse.gif" title="posterior collapse" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/work4_collapse.png" title="repeated tokens" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    You can also have artistically styled 2/3 + 1/3 images, like these.
</div>

I later found that nucleus sampling, while prevent the model from generating repeating sequences, also resolves posterior collapse (the overlap of distributions was well maintained throughout the process). However, this disabled the controllability offered by CVAE.

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/work4_noncollapse.gif" title="non-posterior-collapse" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/work4_noncollapse.png" title="varied tokens" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    You can also have artistically styled 2/3 + 1/3 images, like these.
</div>

Watch my presentation on this project here:

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include video.liquid path="assets/video/presentation.mp4" class="img-fluid rounded z-depth-1" controls=true %}
    </div>
</div>
