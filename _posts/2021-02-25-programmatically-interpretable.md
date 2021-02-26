---
title: "Paper Summary - *Programmatically Interpretable Reinforcement Learning* by Verma et. al. (2018)"
tags: paper-summary xrl-technique
mathjax: true
excerpt: "Summarizing and providing my notes on the paper - Programmatically Interpretable
Reinforcement Learning by Verma et. al. (2018)"
---

Verma, A., Murali, V., Singh, R., Kohli, P., & Chaudhuri, S. (2018, July). Programmatically
interpretable reinforcement learning. In *International Conference on Machine Learning*
(pp. 5045-5054). PMLR.
[<i class="far fa-file-pdf"></i>](http://proceedings.mlr.press/v80/verma18a/verma18a.pdf)
{: .notice--info}

## Paper Summary

The paper presents an XRL technique (NDPS) to imitate a DRL agent. The mimic is represented symbolically as
a program in a DSL. The authors describe how to use NDPS and demonstrate it on the TORCS environment.
They measure the mimic's reward and conduct an ablation study to justify their algorithm choices.
They qualitatively appeal to their interpretability and smoothness of policies produced by NDPS. They
demonstrate NDPS agents being more generalizable and show how its policies can be verified.

## Methodology

The authors define a DSL for describing the space of all policies. The policy DSL defines a policy
using a functional language based on side-effect-free combinators. It basically involves operations
over the *histories* of an agent, where operations can be any common operations over observations
and actions (like addition). The language also allows for numerical constants.

They use a policy *sketch*, which is a grammar defined over the policy DSL which constrains the
space of programs that can be produced. The policy sketch is hand-authored and defined per
environment. It helps reduce the program space to speed up search.

NDPS is described using the below algorithm, reproduced from Algorithm 1 in the paper -

> Input: POMDP $M$, neural policy $e_{\mathcal{N}}$, sketch $\mathcal{S}$
>
> $\mathcal{H} \leftarrow \mathtt{create\\\_histories}(e\_{\mathcal{N}}, M)$  
> $e \leftarrow \mathtt{initialize}(e_{\mathcal{N}}, \mathcal{H}, M, \mathcal{S})$  
> $R \leftarrow \mathtt{collect\\\_reward}(e, M)$  
> **repeat**  
> $\qquad (e', R') \leftarrow (e, R)$  
> $\qquad \mathcal{H} \leftarrow \mathtt{update\\\_histories}(e, e_{\mathcal{N}}, M, \mathcal{H})$  
> $\qquad \mathcal{E} \leftarrow \mathtt{neighbourhood\\\_pool}(e)$  
> $\qquad e \leftarrow \underset{e' \in \mathcal{E}}{argmin} \sum_{h \in \mathcal{H}} || e'(h) - e_{\mathcal{N}}(h)||$  
> $\qquad R \leftarrow \mathtt{collect\\\_reward}(e, M)$  
> **until** $R' \geq R$  
> **Output:** $e'$  

- $M$ is the environment modelled as a POMDP $(\mathcal{S}, \mathcal{A}, \mathcal{O}, T(\cdot\|s,a), Z(\cdot\|s), r, Init, \gamma$).
- $e_{\mathcal{N}}$ is a DRL agent trained on $M$ which serves as a *neural oracle*. NDPS tries to
generate programs which imitate $e_{\mathcal{N}}$.
- $\mathtt{create\\\_histories}$ runs $e_{\mathcal{N}}$ in $M$ to generate a sample set of trajectories.
- $\mathtt{initialize}$ generates a set of sample program policies using *template enumeration* and
*parameter optimization*, simulates each of them using $M$ and returns the one which achieves the
highest reward.

The authors do not describe how they've done the template enumeration and parameter optimization.
Presumably, this involves generating strings from the CFG and optimizing the constants in them using
SGD.
{: .notice--warning}

- $\mathtt{collect\\\_reward}$ is simply to represent the total reward received by the agent over an
episode
- $\mathtt{update\\\_histories}$ uses a heuristic to select "interesting" observations from the trajectory
of $e$. Corresponding actions for each observation are obtained using $e_{\mathcal{N}}$, and are
used to augment $\mathcal{H}$ (input augmentation). This is done to ensure the subsequent distance
between $e$ and $e_{\mathcal{N}}$ is greater, as the suboptimal states which $e$ visits that
$e_{\mathcal{N}}$ never would is considered for the distance measurement.

The authors repeatedly mention using a heuristic to select "interesting" observations from the
trajectory, without really elaborating on what makes these inputs interesting. The objective should
be to select inputs $h$ for which $|| e'(h) - e_{\mathcal{N}}(h)||$ is as high as possible.
{: .notice--warning}

- $\mathtt{neighbourhood\\\_pool}$ generates a set of sample program policies similarly to $\mathtt{initialize}$
- the selected policy is the one which has the least distance from $e_{\mathcal{N}}$ as measured
using the distance metric

## Experimental Setup

The authors evaluate NDPS using the TORCS environment. They generate a programmatic policy to drive
cars around the simulated racetrack. They use input from only 5 sensors as opposed to the full 89
sensors available. They also use only the *Practice Mode* setting of this environment, which is also
the environment in which their DDPG-based neural oracle is trained. The output from the policy is
floating-point values for the steering angle and acceleration.

The sketches used for this domain provide a basic structure of a proportional-integral-derivative
(PID) program. The $fold$ calculation, a basic operation of the policy DSL, is constrained to the
last 5 observations of the history, and has precedent in the automatic error reset strategy of discrete
PID controllers.

$\mathtt{update\_histories}$ simply captures all inputs from the program agent trajectory.

The authors also evaluate NDPS in three classic control environments. However, the results are present
in an appendix which do not appear in this paper.

The authors compare the performance of agents designed to measure the impact of various choices in
NDPS, namely -

- *DRL*. The neural oracle trained using DDPG
- *Naive*. A program synthesized without the *policy oracle*
- *NoAug*. A program synthesized without using *input augmentation*
- *NoSketch*. A program synthesized without sketch guidance
- *NoIF*. A program synthesized using a stricter sketch which doesn't permit conditional branching
- *NDPS*. A program synthesized using NDPS as described in the paper

Performance is evaluated using 2 different tracks in the TORCS environment. 

## Results and Analysis

The results show that all the components produce better lap times and higher rewards. They even lead
to agents which can complete the lap as opposed to agents like *Naive*, *NoAug* and *NoSketch* where
the generated agents could not complete a lap in 12 hours.

Interpretability of the policy is appealed to qualitatively due to the human-readable nature of it.

The NDPS policy was claimed to lead to "smoother" behaviour in terms of the lower variance of its
steering actions.

Statistical significance of the differences in these metrics was not reported.
{: .notice--warning}

The authors conduct a study using a *partially observable* variant of TORCS where a random sample
of $k$ sensors are made defective. Defective sensors are *blocked* with a certain fixed probability,
meaning they will return either the correct reading or `null`. They compare the behaviour of DRL
and NDPS in this environment. For many different combinations of blocked sensor and blocking
probability, NDPS travels farther than DRL before crashing. The authors use this as evidence that
NDPS agents are more robust to missing/noisy features.

To measure transferability, the authors measured the performance of NDPS and DRL on previously unseen
tracks, similar to the ones they had been trained on. Only NDPS was able to complete the race
successfully while DRL crashed before completion.

The authors show that they are able to prove bounds on the smoothness of the acceleration actions output
by the generated policy. They can also prove bounds on the action values.

## Discussion

The authors mention that these bounds are irrelevant in the case of the TORCS domain since the
action values get clipped much inside the bounds. They mention that the paper only dealt with
symbolic inputs and admit the challenges of extending it to perceptual inputs. They also mention
considering only deterministic policies, and the potential to extend to stochastic policies in future.

## Impressions

I liked this paper. The idea of using a DSL to define the policy and learn it using a DRL oracle is
similar to my vague idea of using a DSL to characterize a "strategy" and learn it from a DRL agent
trained on the task (I got this idea from Butler et. al. (2017)). The policy DSL seems very general
and in practice is probably going to necessitate a high-quality sketch.

The sketch design and creation is going to be a pain point, and as with all hand-authored elements
of ML papers, it hardly receives much attention here. There would need to be a balance between
constraining the program space enough for practical search and creating rigid rules which could
probably be replicated by hand. It is also going to be a challenge to design this for varying state
and action representations. The ones used in TORCS are simple 1-D vectors.

Interpretability of the policies generated by NDPS is very questionable to a layperson. Even the
policy presented in the paper (Figure 2) is borderline unreadable. There is no qualitative testing
of this done on the lines of Madumal et. al. (2019) or Lage et. al. (2019). The nature of the
defined policy DSL means the policies are always going to be lengthy strings of observations and
operators.

Perhaps we could allow the generated policies to only be applicable in certain states. If the
applicability condition could be constrained (in terms of length) and the policy length itself
could be constrained (there would be limits on the value of the action, but we can counter that
by learning these rules from (state, action) pairs with high estimated value), then we might have
more interpretable sub-policies.

*[NDPS]: neurally-directed program search
*[DSL]: domain-specific language
*[DDOG]: Deep Deterministic Policy Gradients
