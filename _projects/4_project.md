---
layout: page
title: SPHRED for Dialog modeling
description: Rebuilding a Conditional Variational Framework for Dialog Generation
img: assets/img/work4_noncollapse_distr.gif
importance: 4
category: research
---

<h5><b>Motivation</b></h5>

This is a research project for my graduation thesis at International University, Vietnam National University - HCMC. More information can be found [here](https://github.com/hieuchi911/sphred_from_vhred).

In this project, I sought to expand my technical foundations beyond the standardcurriculum by studying and reconstructing SPHRED, a combination of Conditional Variational Autoencoder and Hierarchical Recurrent Encoder Decoder for controllable dialog generation. Its architecture is shown below:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/work4_sphred.png" title="SPHRED architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    SPHRED architecture includes a Hierarchical Recurrent Encoder-Decoder (HRED, comprises of an encoder RNN, a context RNN, and a decoder RNN) for dialog generation and a conditional variational autoencoder for controllability.
</div>

<h5><b>Investigating VAE distributions changes in training</b></h5>

While replicating the original results, I employed PrincipalComponent Analysis (PCA) to visualize the approximate posterior and latent prior distributions during training to gain adeeper understanding of variational inference. This revealed a very intriguing pattern: as the model began generating repetitive tokens (text degeneration), the two distributions diverged.

<div class="row justify-content-sm-center">
    <div class="col-sm-5 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/work4_collapse_distr.gif" title="posterior collapse" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-7 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/work4_collapse_tokens.png" title="repeated tokens" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The prior and the approximate posterior distribution diverged (left), which happens as the model generates repeated tokens (right).
</div>

However, before text degeneration, the model were able to generate sound predictions, and the two distributions were able to come to an overlap. This agrees with the objective of minimizing the KL divergence between the prior distribution and the approximate posterior distribution.

<div class="row justify-content-sm-center">
    <div class="col-sm-5 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/work4_noncollapse_distr.gif" title="non-posterior-collapse" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-7 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/work4_noncollapse_tokens.png" title="varied tokens" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The prior and the approximate posterior distribution ovelapped (left), which happens as the model generates sound sequences (right).
</div>

<h5><b>Presentation</b></h5>

Watch my presentation and the demo on this project here:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include video.liquid loading="eager" path="assets/video/presentation.mp4" class="img-fluid rounded z-depth-1" controls=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include video.liquid loading="eager" path="assets/video/demo.mp4" class="img-fluid rounded z-depth-1" controls=true %}
    </div>
</div>
<div class="caption">
    A presentation on the model architecture, optimization, and results (left) and a demo on the model's dialog generation (right).
</div>
