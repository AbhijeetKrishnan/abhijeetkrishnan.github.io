---
title: "Paper Summary - *Distilling Deep Reinforcement Learning Policies in Soft Decision Trees* by
Coppens et. al. (2019)"
tags: paper-summary xrl-technique
mathjax: true
excerpt: "Summarizing and providing my notes on the paper - Distilling Deep Reinforcement Learning
Policies in Soft Decision Trees by Coppens et. al. (2019)"
---

Coppens, Y., Efthymiadis, K., Lenaerts, T., Nowé, A., Miller, T., Weber, R., & Magazzeni, D.
(2019, August). Distilling deep reinforcement learning policies in soft decision trees. In
*Proceedings of the IJCAI 2019 Workshop on Explainable Artificial Intelligence* (pp. 1-6).
[<i class="far fa-file-pdf"></i>](https://cris.vub.be/ws/portalfiles/portal/46718934/IJCAI_2019_XAI_WS_paper.pdf)
{: .notice--info}

## Paper Summary

This paper presents an XRL technique to explain the global policy of an agent by training a surrogate
tree-based model and visualizing its learned weights. A dataset of states and corresponding action
distributions is used to train a *soft decision tree* (SDT) which learns features by which it splits
the input down into different branches of the tree, with leaf nodes learning an action distribution.
The learned weights are visualized using a heat map, and the action distributions are graphed as aids
to analysing the policy. The authors provide some analysis of the policy of an agent trained on the
Mario AI benchmark and compare the SDT's performance to the original agent.

## Methodology

A trained agent is used to produce a dataset of $(s, \pi(s))$ records by exploration in the
environment. $\pi(s)$ is the action distribution at state $s$. This dataset is used to train an SDT.

Can an agent which learns a Q-function be used to output this action distribution? It would depend
on the action selection mechanism, but I think yes.
{: .notice--success}

An SDT is a hybrid tree-based model which combines binary decision trees with neural networks. A
depth is predetermined, which fixes the tree structure. Each node takes as input a state and outputs
a probability with which to pass it down to the right child node. It thus learns a hierarchy of
filters which assign an input state to a particular leaf node. The leaf nodes learn a softmax
distribution over the actions.

The filters are visualized using a heat map. These filters, in addition to graphs of the action
distributions at the leaf nodes, aid in analysing the policy.

## Experimental Setup

The authors used the Mario AI benchmark as a test environment. They trained the original agent using
actor-critic Proximal Policy Optimization (PPO). The policy of this agent was distilled to SDTs of
several depths. Average reward per episode was measured for each SDT model and compared to the
original PPO agent. Details of both model architectures and training can be found in the paper.

There should be a more concise term for *original agent*, or the 'agent for which explanations are
being provided'. A term I've seen used is *explanandum*, a Latin word for 'the phenomenon being
explained'. I would like to modify it to indicate that the phenomenon in question is an RL model. A
term used in biomimicry is *model* to denote the 'thing that is being mimicked', which communicates
this sentiment well. However, *model* already has a precise meaning in RL literature, so it might
not be wise to overload it with this additional meaning.
{: .notice--success}

## Results and Analysis

The authors use the heatmap of the SDT filters to deduce that features like 'what is directly in
front of Mario' are given more importance, while features like the floor are not considered important.

A visualization of an input state overlayed with the node filters on its path, along with a graph
of the action distribution of the leaf node and the PPO agent is shown. The authors use it to show
why low-reward coins are not given importance by the policy, and high-reward coins are. They also
show that the agent has learned to jump on enemies to kill them.

![Figure 3 from Coppens et. al. (2019)](/assets/images/coppens-fig-3.png "Figure 3 from Coppens et. al. (2019)")

The authors also visualize the action distribution of *all* leaf nodes in different graphs, showing
that the agent generally tends to move to the right, since those actions qualitatively have the
highest probabilities accorded to them.

The performance graph (in terms of average reward per episode) shows that SDT performance tends to
improve with increasing depth, but still remains quite inferior to the PPO agent's performance.

## Discussion

The authors discuss visualization of the learned filters in *non-spatial* settings i.e., settings
where the input features are not pixels. They note that the explanations are not in a symbolic form
which is presumably more human-readable. Rule-based ML techniques could be applied here, and user
tests could be conducted to determine the usefulness of the explanations.

They discuss the possibility of fitting an SDT directly on the data i.e., training an SDT-based RL
agent in an environment without mimicking an existing RL agent. Given that the SDTs underperformed
compared to the model agent, the authors question the resemblance of the learned policy to the model
policy and whether any insights drawn would be valid. They discuss improving the performance of the
SDT agent. Finally, they discuss adaptively changing the tree depth.

## Impressions

This is a small but neat paper which explains itself and analyses its limitations well. The actual
method is not a significant advancement (use an existing distillation technique, use very basic
visualization methods), but the otherwise the analysis is done well.

The problem of explaining the policy is reduced to the problem of explaining the learned filters in
the SDT, and the action distributions in the leaf nodes. Explaining filter weights is probably being
studied extensively in XAI and we should be able to draw on research there to explain these filters
in case of non-spatial environments.

The insights about the policy that were drawn didn't seem novel, and merely confirmed what was already
known. If there was a unique characteristic or bug in the model agent that was described and could
be explained by the SDT, that would be more convincing.

Additionally, since the filters are features learned by the SDT which mimics the model agent, it is
unclear if any explanation of these features is also a valid explanation of how the model agent
processes its input. If two very different processes lead to the same conclusion, are the explanations
for those processes equivalent? This probably depends on the use case of the explanations.

Lastly, a user evaluation of these explanations, or an additional analysis of a dissimilar domain
would have been useful to have.

*[SDT]: soft decision tree
*[PPO]: Proximal Policy Optimization
*[XAI]: explainable AI
