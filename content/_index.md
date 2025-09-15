+++
title = "Lies We Can Trust: Quantifying Action Uncertainty with Inaccurate Stochastic Dynamics Through Conformalized Nonholonomic Lie Groups"
[extra]
display_title = "<em>Lies</em> We Can Trust: Quantifying Action Uncertainty with Inaccurate Stochastic Dynamics Through Conformalized Nonholonomic <em>Lie</em> Groups"
authors = [
    {name = "Luís Marques", url = "https://marquesluis.com/"},
    {name = "Maani Ghaffari", url = "https://curly.engin.umich.edu/"},
    {name = "Dmitry Berenson", url = "https://berenson.robotics.umich.edu/"}
]
venue = {name = "Under Review", url = ""}
buttons = [
    {name = "ArXiv", url = ""},
    {name = "PDF", url = ""},
    {name = "Code", url = "https://github.com/UM-ARM-Lab/claps_code"}
]
katex = true
large_card = true
favicon = false
+++

<div class="abstract-with-figure">

<div class="text-column">

We propose **C**onformal **L**ie-group **A**ction **P**rediction **S**ets (**CLAPS**), a symmetry-aware conformal prediction-based algorithm that constructs, for a given action, a set guaranteed to contain the resulting system configuration at a user-defined probability.
Our assurance holds under both aleatoric and epistemic uncertainty, non-asymptotically, and does not require strong assumptions about the true system dynamics, the uncertainty sources, or the quality of the approximate dynamics model.
Typically, uncertainty quantification is tackled by making strong assumptions about the error distribution or magnitude, or by relying on uncalibrated uncertainty estimates --- i.e., with no link to frequentist probabilities --- which are insufficient for safe control.
Recently, conformal prediction has emerged as a statistical framework capable of providing distribution-free probabilistic guarantees on test-time prediction accuracy.
While current conformal methods treat robots as Euclidean points, many systems have non-Euclidean configurations, e.g., some mobile robots have `$SE(2)$`.
In this work, we rigorously analyze configuration errors using Lie groups, extending previous Euclidean Space theoretical guarantees to `$SE(2)$`. Our experiments on a simulated Jetbot, and on a real MBot, suggest that by considering the configuration space's structure, our symmetry-informed nonconformity score leads to more volume-efficient prediction regions that represent the underlying uncertainty better than existing approaches.

</div>

<div class="figure-column">

{% figure(alt=["CLAPS Overview"] src=["./title_figure_manual.png"]) %}
**Title Figure.** Our proposed algorithm (**CLAPS**) constructs prediction regions `$\mathcal{C}^q$` (in C-Space) that are *marginally guaranteed* to contain the next *unknown system configuration* at a user-set probability `$(1-\alpha)$`. By considering the robot's symmetry, we can construct more *efficient* prediction regions.
{% end %}
</div>

</div>

# Problem Statement

**Problem.** TBD - sumamrize key assumpt, goals, ...

Our goal is to provide, for a given admissible action `$u_{des}$`, a C-Space prediction region `$C^q \subseteq \mathcal Q$` that provably contains the next true unknown system configuration `$q_1$` with, at
least, a user-defined probability `$(1-\alpha)$`, i.e.
`$ \mathbb{P}(q_1 \in \mathcal C^q ) \ge (1 − \alpha)$`,
where `$\alpha \in (0,1)$ is the user-set acceptable failure-probability.
Following CP literature, the likelihood is taken on average
over the test-time scenarios, not for a specific $q_1$, and we
assume the initial system state to be known (i.e., ˜s0 = s0).
While purely achieving (2) is trivial, e.g., by predicting the
entire space Cq = Q, we additionally want Cq to be as
tight/volume-efficient as possible to make it practical for
downstream robotic tasks such as safe control. This is a chal-
lenging problem. We consider both aleatoric and epistemic
uncertainty, and do not make strong assumptions about the fi-
delity of ˜f , or the nature of the stochastic disturbances. While
we make no claims about how efficient our prediction regions
are, we show they can be tighter than existing methods.


# CLAPS

TBD - explain method, include algorithm blocks

{% figure(alt=["CLAPS Method Diagram"] src=["./method_diagram3v2.png"] dark_invert=[true]) %}
**Method Figure.** **C**onformal **L**ie-Group **A**ction **P**rediction **S**ets | Offline: a dataset of state transitions is used jointly with an approximate dynamical model to derive a rigorous symmetry-aware probabilistic error bound on the configuration predictions. Online: our algorithm takes in a desired action `$u_{des}$` and computes a *calibrated C-Space prediction region* `$\mathcal{C}^q$` that is marginally guaranteed to contain the true configuration resulting from executing `$u_{des}$`.
{% end %}

# Experiments
TBD - explain

## JetBot Experiments (Simulation)

put table, explain c-space below

{{ figure(src = ["./cspace_videos/Isaac_Jetbot_over_confident_0300_3D_rotation.mp4", "./cspace_videos/Isaac_Jetbot_over_confident_0330_3D_rotation.mp4","./cspace_videos/Isaac_Jetbot_over_confident_0336_3D_rotation.mp4"], alt = ["C-space visualization - scenario 0300", "C-space visualization - scenario 0301"], dark_background=[true]) }}


{{ figure(alt=["LUCCa vs Baseline Performance Table"] src=["table_jetbot.svg"] dark_invert=[true] style="width:80%") }}

## MBot Experiments (Hardware)

epxlain

{{ figure(alt=["LUCCa in Action", "Baseline Comparison"] src=["./mbot_videos/mbot_clip1.mp4", "./mbot_videos/mbot_clip2.mp4", "./mbot_videos/mbot_clip3.mp4"] subcaption=["**LUCCa in Action** - Robot navigating with calibrated uncertainty estimates", "**Baseline Comparison** - Traditional approach without conformal calibration"]) }}

explain

put runtime in words.

<figure class="single-video-figure">
    <div class="single-video-wrapper">
        <iframe
            src="https://www.youtube.com/embed/hyddmrzfx7Y"
            title="MBot Hardware Demonstration"
            frameborder="0"
            allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
            allowfullscreen>
        </iframe>
    </div>
</figure>

<style>
.single-video-figure {
    max-width: 1000px;
    margin: 2rem auto;
    display: flex;
    flex-direction: column;
    align-items: center;
}

.single-video-wrapper {
    position: relative;
    width: 100%;
    height: 0;
    padding-bottom: 56.25%; /* 16:9 aspect ratio */
    border: 2px solid #e9ecef;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    overflow: hidden;
}

.single-video-wrapper iframe {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}

.single-video-figure figcaption {
    padding: 1rem;
    background: #f8f9fa;
    border-top: 1px solid #e9ecef;
    text-align: center;
    font-size: 0.9rem;
    line-height: 1.4;
    width: 100%;
    max-width: 1000px;
}

@media screen and (max-width: 768px) {
    .single-video-figure {
        margin: 1rem auto;
    }
}
</style>

# BibTeX <small><small>(cite this!)</small></small>

```
TBD

```
