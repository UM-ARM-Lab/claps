+++
title = "Lies We Can Trust"
[extra]
display_title = "<em>Lies</em> We Can Trust: Quantifying Action Uncertainty with Inaccurate Stochastic Dynamics Through Conformalized Nonholonomic <em>Lie</em> Groups"
authors = [
    {name = "Luís Marques", url = "https://marquesluis.com/"},
    {name = "Maani Ghaffari", url = "https://curly.engin.umich.edu/"},
    {name = "Dmitry Berenson", url = "https://berenson.robotics.umich.edu/"}
]
venue = {name = "Under Review", url = ""}
buttons = [
    {name = "ArXiv", url = "https://arxiv.org/abs/2512.10294"},
    {name = "PDF", url = "https://arxiv.org/pdf/2512.10294"},
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

{% problem_columns() %}
### Problem Setting
Let `$q\in \mathcal Q$` be the robot configuration, `$\dot q \in T_q \mathcal Q$` the generalized velocity, and `$s := (q,\dot q) \in T\mathcal Q$` the state. We consider holonomic and *nonholonomic* systems whose `$\mathcal Q$` is the Lie group `$SE(2)$` (unicycles, car-like robots, quadrotors, surface/underwater vehicles, satellites, quadrupeds’ COM, ...). The *unknown dynamics* evolve as
```
$$
s_{k+1} = f(s_k, u_k, w_k), \qquad w_k \sim P_{noise},
$$
```
where `$f$` is *unknown*, `$w_k$` is an iid disturbance drawn from an *unknown* distribution, and `$u_k \in \mathbb R^m$` is the control input. Inaccuracies
in modeling `$f$` may arise e.g., from domain shifts between fitting and deployment, and result in *epistemic uncertainty*. Additionally,
`$w_k$` introduces *aleatoric uncertainty*, and may represent external disturbances such as wind gusts, wheel slippage, or terrain bumps.
---
### Objective
For a given admissible action `$u_{des}$`, provide a C-Space prediction region `$C^q \subseteq \mathcal Q$` that contains the resulting (unknown) configuration `$q_1$` with probability at least `$(1-\alpha)$`:
```
$$
\mathbb{P}(q_1 \in \mathcal C^q ) \ge 1 - \alpha, \quad \alpha \in (0,1).
$$
```
where `$\alpha$` is the user-set acceptable failure probability. While purely achieving this goal is trivial, e.g., by predicting the entire space `$(C^q = \mathcal Q)$`, we additionally want `$C^q$` to be as *tight/volume-efficient* as possible, to make it practical for downstream robotic tasks such as *safe control*. We do not make strong assumptions about the fidelity of `$\tilde{f}$`, or the nature of the stochastic disturbances.
{% end %}

# CLAPS

**CLAPS** uses a dataset of state transitions `$(D_{cal})$` to *calibrate* the uncertainty estimates provided by approximate dynamics models.
**CLAPS** can be applied as a *post-hoc calibration layer* on top of existing Lie-algebraic Gaussian uncertainty estimators (e.g., Invariant EKF), turning their approximate covariances into *provably calibrated ones*.
By using a *symmetry-respective score metric*, our approach produces prediction regions that are more volume-efficient than existing conformal prediction baselines that treat the robot's configuration as Euclidean.

{% figure(alt=["CLAPS Method Diagram"] src=["./method_diagram3v2.png"] dark_invert=[true]) %}
**Method Figure.** **C**onformal **L**ie-Group **A**ction **P**rediction **S**ets | Offline: a dataset of state transitions is used jointly with an approximate dynamical model to derive a rigorous symmetry-aware probabilistic error bound on the configuration predictions. Online: our algorithm takes in a desired action `$u_{des}$` and computes a *calibrated C-Space prediction region* `$\mathcal{C}^q$` that is marginally guaranteed to contain the true configuration resulting from executing `$u_{des}$`.
{% end %}

The prediction region constructed by **CLAPS** `$(C^q \subseteq Q)$` can be used for probably-safe control in three main ways (for more details refer to Section `$\S$`V-C):
1. Configuration Check: a (sample) configuration `$g$` belongs in `$C^q$` if `$\sqrt{\log(\tilde{g}^{-1}g)^\top \tilde{\Sigma}^{-1}\log(\tilde{g}^{-1}g)} \le \chi^2_{\alpha}(\dim \mathfrak g)$` --- quick to evaluate in batch
2. C-space set: The `$C^q$` can be reconstructed by Alg. 2, for example to check if `$C^q \subseteq \mathcal Q_{safe}$`, for a known safe set `$\mathcal Q_{safe} \subseteq \mathcal Q$`.
3. Workspace set: `$C^q$` can be inflated by the robot's radius and mapped to the workspace `$(\mathbb R^2)$` to perform collision checks with known obstacles.

# Experiments
We compare **CLAPS** against seven baselines in both simulation (JetBot) and hardware (MBot) to demonstrate its improved *efficiency* and *representation quality*.
We model both systems as a second-order unicycles, and perform standard system identification to estimate the inertial properties. In all the experiments below we use `$\alpha=0.1$`.

**A) JetBot Experiments (Simulation)**<br>
&emsp;In Isaac Sim, we independently sampled additive perturbations to `$u_{des}$`, introducing aleatoric uncertainty. This leads to the well-known banana-shaped distributions seen below. Epistemic uncertainty arose from unmodeled effects (e.g., friction), and imperfections in the mass/inertia estimation.
The Figure below demonstrates **CLAPS**' ability to represent the underlying dynamics uncertainty of the unknown system (MC particles).
{% figure(alt=["Workspace method comparison plot"] src=["contour_comparison_val_isaac_0007_vs_val_isaac_0590_clear.png"] dark_invert=[true] style="width:80%") %}
**Workspace (`$\mathbb{R}^2$`) footprint**. Workspace marginalization of the C-Space regions generated by the methods, over two of the 625 JetBot validation trials. Left: lower linear and angular velocity. Right: higher velocity case.
InEKF+MLE has expected pose `$\tilde{g}_1$` shown as the gray dot. All other methods
have the same expected pose, which is represented by the blue dot. Both InEKF+2M and InEKF+MLE produce the same uncertainty covariance for all initial states and control inputs.
The Point Prediction (PP) methods generate large regions with boundaries lying
outside the plots’ margins. SS EKF, InEKF, InEKF+2M, and InEKF+MLE
are not guaranteed to contain the resulting configuration at the user-set
likelihood.
Qualitatively, CLAPS appears to more accurately represent the
underlying uncertainty distribution than the symmetry-unaware baselines. 
{% end %}

Quantitatively, **CLAPS** achieves the highest average Intersection over Union (IoU) with the MC particles, validating its alignment with the systems' uncertainty propagation, and **CLAPS** has a smaller C-space volume than all calibrated baselines in each of the 625 validation trials we tested. 

{{ figure(alt=["JetBot Performance Table"] src=["table_jetbot.svg"] dark_invert=[true] style="width:80%") }}

Below we visualize the C-space regions `$C^q$` constructed by the different methods in three of the 625 validation trials. The State Space (SS) baselines produce hyperellipsoids in configuration space, due to treating it as Euclidean. Instead, both the Invariant Kalman Filter (InEKF) and CLAPS produce symmetry-respective prediction regions, better capturing the underlying uncertainty.
While the uncertainty estimates provided by the InEKF are approximate, **CLAPS** provides provably calibrated prediction regions suitable for safe-control.

{{ figure(src = ["./cspace_videos/Isaac_Jetbot_over_confident_0300_3D_rotation.mp4", "./cspace_videos/Isaac_Jetbot_over_confident_0330_3D_rotation.mp4","./cspace_videos/Isaac_Jetbot_over_confident_0336_3D_rotation.mp4"], alt = ["C-space visualization - scenario 0300", "C-space visualization - scenario 0301"], dark_background=[true]) }}


**B) MBot Experiments (Hardware)**<br>
&emsp;We also validated our method on an MBot, a differential-drive vehicle shown below. 
Despite a relatively-small calibration dataset corresponding to `$\approx$`2 min of driving data `$(\lvert D_{cal}\rvert = 237)$`, our method provably satisfied the user-specified safety specifications, thanks to its *non-asymptotic guarantees*.
**CLAPS** uses `$D_{cal}$` to derive data-driven provable (probabilistic) bounds on the uncertainty arising from both model mismatch, and inherent stochasticity.

{{ figure(alt=["MBot data collection clips"] src=["./mbot_videos/mbot_clip1.mp4", "./mbot_videos/mbot_clip2.mp4", "./mbot_videos/mbot_clip3.mp4"] subcaption=["Data collection videos"]) }}

The system configuration and velocity were estimated using a motion capture system. Uncertainty in the resulting configuration arose due to inaccuracies in inertial property estimation, actuation delays, center-of-mass deviation from the body-fixed origin, ground-surface imperfections, friction, network jitter, etc.
The collection procedure of system transitions that make up `$D_{cal}$` and the validation set is shown below.
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

Our Python-implementation of **CLAPS** can run at 25 Hz, the sampling frequency of the MBot's sensors, making it serviceable for online use.

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
@misc{marques2025liestrustquantifyingaction,
      title={Lies We Can Trust: Quantifying Action Uncertainty with Inaccurate Stochastic Dynamics through Conformalized Nonholonomic Lie Groups}, 
      author={Luís Marques and Maani Ghaffari and Dmitry Berenson},
      year={2025},
      eprint={2512.10294},
      archivePrefix={arXiv},
      primaryClass={cs.RO},
      url={https://arxiv.org/abs/2512.10294}, 
}
```
