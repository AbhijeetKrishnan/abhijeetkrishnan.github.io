---
title: "Paper Summary - *Hierarchical and Interpretable Skill Acquisition in Multi-Task Reinforcement
Learning* by Shu, Xiong & Socher (2017)"
tags: paper-summary, hierarchical-rl
mathjax: true
excerpt: "Summarizing and providing my notes on the paper - Hierarchical and Interpretable Skill
Acquisition in Multi-Task Reinforcement Learning by Shu, Xiong & Socher (2017)"
---

Shu, T., Xiong, C., & Socher, R. (2017). Hierarchical and interpretable skill acquisition in
multi-task reinforcement learning. *arXiv preprint arXiv:1712.07294.*
[<i class="far fa-file-pdf"></i>](https://arxiv.org/pdf/1712.07294.pdf)
{: .notice--info}

## Paper Summary

This paper presents a technique to perform hierarchical reinforcement learning. It presents a method
to train a policy to use previously learned skills to learn more complex skills. The skills can be
interpreted in natural language. The authors use a custom Minecraft environment built using [Project
Malmo](https://www.microsoft.com/en-us/research/project/project-malmo/) as their testbed. They
conduct an ablation study using learning curves to demonstrate the impact of their algorithm
choices. They compare their hierarchical policy with a "flat" policy to demonstrate its generalizability.
They demonstrate its interpretability by displaying the hierarchical plan for several tasks in a
tree-based format.

## Methodology

The method presented in the paper trains a policy which learns to solve *tasks*. Tasks (denoted by $g$)
are "skills" which the agent can learn, and each has its associated reward function $R(s, g)$. In
the paper, tasks are constrained to a specific template $\langle \text{skill}, \text{item} \rangle$
e.g., $\langle \text{find}, \text{blue} \rangle$ which denote object manipulation tasks. Each task
can thus be represented in natural language.

The model is trained to solve tasks in a hierarchical fashion. It first learns to solve "base" level
tasks using the environment action set. The task set is then augmented with "higher-order" tasks
which can ostensibly be solved using the lower-order task policies, and a "higher-order" policy is
trained on this new task set, which constrains it to use as "actions" the tasks from the lower-order
policy. This process is repeated until all tasks have been learned. Figure 1 in the paper shows how
the task "stack blue" is learned by composing the lower-order tasks "get blue", "find blue" and "put
blue".

Concretely, the model defines the global policy of a level in the hierarchy (denoted by
$\pi_k$, with $k$ being the level) using the following sub-policies,

1. base policy: the global policy of the previous level i.e., $\pi_{k-1}$
2. instruction policy: informs the base policy which base task it needs to execute; defined using
conditionally independent distributions for $\text{skill}$ and $\text{item}$ ($\pi_k^{\text{instr}}(g'|s_t,g)$)
3. augmented flat policy: allows selection of base actions at any level to ensure that
the global policy is able to perform novel tasks that cannot be achieved by reusing the base policy
($\pi_k^{\text{aug}}(a|s_t, g)$)
4. switch policy: determine whether to use the base policy or the augmented flat policy $\pi_k^{\text{sw}}(e_t|s_t,g)$
where $e_t$ is a binary variable where $0$ selects the base policy, and $1$ selects the augmented
policy

The global policy $\pi_k(a_t\|s_t,g)$ can thus be denoted compactly as -

$$e_t \sim \pi_k^{\text{sw}}(e_t|s,g) \tag{switch policy}$$

$$g_t' \sim \pi_k^{\text{instr}} (g'_t|s_t,g) \tag{instruction policy}$$

$$a_t \sim \pi_k(a_t|s_t,g) = \pi_{k-1}(a_t|s_t,g')^{(1-e_t)}\pi_{k}^{\text{aug}}(a_t|s_t,g) \tag{global policy}$$

The global policy is augmented with the use of an STG (spatial temporal grammar) to capture temporal
relationships between tasks. If a particular order of task selection has been used in the past, the
STG will be able to provide it as a prior to the other policies. Concretely, the STG learns the
distribution of an alternate Markov chain from the sequence of $\langle e_t,g_t' \rangle$ tuples
conditioned on a specific task. giving us the transition probabilities $\rho_k(e_t,g_t'|e_{t-1},g_{t-1}',g)$
and the distribution of $\langle e_0,g_0' \rangle$ as $q_k(e_0,g_0'|g)$. These are used to augment
the switch and instruction policies of the respective level.

Learning this policy proceeds in two phases. First, tasks are learned only by sampling tasks
from the previous task set, ensuring that the global policy learns to connect tasks to previously
learned tasks. In the next phase, it is allowed to sample the full action set, allowing it to
discover new ways to complete tasks.

All policies are optimized using advantage actor-critic (A2C). The advantage functions used are
provided in the paper. Other training details to stabilize learning and increase episode efficiency
are also provided. The STG is trained using simple MLE.

## Experimental Setup

The authors use a custom room in Minecraft built using the Malmo platform as their environment. They
define the tasks as "find X", "get X", "put X" and "stack X" with 6 different block colours, amounting
to 24 different tasks. Each task has its own reward function also defined.

The authors adopt a specific skill acquisition order but stress than any alternate order would not
invalidate the conclusions.

The authors perform an ablation study with different versions of their model without certain features
and a baseline flat policy. They plot the learning curves for each of these models to compare average
reward gained and convergence rates. They also demonstrate the efficacy of their 2-phase curriculum
learning using this method.

The authors compare the performance of their policy vs. a flat policy trained on the first level of
tasks (i.e., "get X"). They use two rooms with differing number of objects for this.

Lastly, the authors present a visualization of the plans of several tasks generated by their policy.
They use it to claim interpretability.

## Results and Analysis

The ablation study confirms that the algorithm and training choices do indeed provide higher reward
compared to the flat policy and improve convergence rate. Their policy far outperforms the flat policy,
demonstrating the generalization capability of the model.

## Discussion

The authors admit the reliance on "weak human supervision" to define the order of skills to be learned
in each training stage, with future work involving how to increase the task set automatically.

## Impressions

This work is situated more in hierarchical RL than in XRL. The "explanations" or interpretability of
the policy was very briefly touched on in the paper, and the bulk of the focus was more on
demonstrating performance improvement. Regarding interpretability, it does seem like a good tool for
showing the hierarchy of the learned policy. However, I question whether a domain will have tasks
which neatly fit the $\langle \text{skill}, \text{item} \rangle$ template used here, or whether
the tasks will have a sensible hierarchy among them. We might have the policy learn a hierarchy for
tasks which make no sense to a human (which isn't a bad thing, since it helps debug the system).

I'm not very familiar with the field of hierarchical RL. It seems like a major authoring burden to
define a set of useful tasks, and an appropriate hierarchy of them for learning. The tasks need to
be composed in terms of the base action sets, and, if we want interpretability, need to be translated
for humans.

Regarding changing the order of the task sets, the authors simply mention -

> One may also alter the order, and the main conclusions shall still hold.

I would have liked to see the effects of reversing the order used in the paper on training and
performance.

*[STG]: spatial temporal grammar
