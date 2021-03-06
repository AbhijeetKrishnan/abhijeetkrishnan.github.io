---
title: "Paper Summary - *Improving Robot Controller Transparency Through Autonomous Policy Explanation* by Hayes & Shah (2017)"
tags: paper-summary xrl-technique
mathjax: true
excerpt: "Summarizing and providing my notes on the paper - Improving Robot Controller Transparency Through Autonomous Policy Explanation by Hayes & Shah (2017)"
---

Hayes, B., & Shah, J. A. (2017, March). Improving robot controller transparency through autonomous policy explanation. In *2017 12th ACM/IEEE International Conference on Human-Robot Interaction (HRI* (pp. 303-312). IEEE. [<i class="far fa-file-pdf"></i>](http://interactive.mit.edu/sites/default/files/documents/hayes-hri17.pdf)
{: .notice--info}

## My Objectives

To learn about an XRL technique that I could use for my project.

## Paper Summary

This paper presents a method to extract explanations from a policy. It requires learning an MDP of
the policy using a dataset of real or simulated transitions. The method uses a hand-authored mapping
from state observations to state predicates (*communicable predicate set*). Given these, it can
provide explanations in the form of DNF formulas for 3 different query types -

1. When do you do {action}?

    Provide a general state description for the states in which you do {action}

2. What do you do when {state}?

    Provide the action taken by the policy in states which match {state}

3. Why didn't you do {other-action}?

    Provide a description of why {other-action} wasn't taken in the current state. This is simply
    counterfactual reasoning as described further in Madumal et. al. (2019).

The DNF formulas generated are combined with a templating system to generate natural language explanation
to human users. The focus in the paper is on using the explanations to aid debugging agent behaviour.
The authors provide examples of generated explanations in 3 different domains and contrast it with
an expert explanation.

## Methodology

### Learning the agent policy

In this step, a dataset consisting of $(s, a, s')$ tuples is generated by running the agent in the
environment. The dataset is used to approximate the agent policy and learn a function to predict
which action the agent will take in a given state.

The method of learning the policy is not specified in the paper. Presumably, we can use any method
to do so, ranging from crude frequency-based approximations to more sophisticated methods.
{: .notice--warning}

### Communicable Predicate Sets

These are a collection of hand-authored predicates which provide relevant state information. They
may or may not map directly onto the state features provided as input to the agent. They are a set of
first-order logic clauses which can be either $\text{True}$ or $\text{False}$ in a given state. An
example of these in the CartPole domain are "pole falling left/right", "pole standing up", "cart
moving left/right" etc.

These predicates are used to produce a DNF formula which represents a boolean function that is true
for a given set of states, and false for another. If these groups of sets have some common property, say, if
they are the sets in which a particular action is frequently taken, then the resulting DNF formula
becomes the explanation for *why* that action was taken in terms of the communicable predicates. If
we further apply the templates based on the query and predicate, we can generate a natural language
explanation. For e.g., if we have predicates $\text{idle}/0$ and $\text{running}/0$ indicating
whether the robot is active/idle, and if the factory line is running/stationary respectively, a DNF
formula $(\text{idle} \land \lnot \text{running} \lor (\lnot \text{idle}))$ would beget the
explanation "*The robot is idle* and *the factory line is not running* or *the robot is active*".

The DNF formula is generated from a target set of states $S$ and an excluded set of states $\overline{S}$.
The problem is equivalent to finding the minimal set cover for the states, and is solved using the
Quine-McClusky algorithm (although an approximate algorithm like ESPRESSO can also be used in case
of large state spaces).

The resulting DNF formula can be used to generate explanations for 3 different query types. The
authors provide algorithms for each, and they involve different ways to construct the sets $S$ and
$\overline{S}$ to generate the descriptions.

1. When do you do {action}?

    Using the learned agent policy, the environment's state space is enumerated (or sampled, if large)
    and $S$ is constructed from every state where the target action is taken, and $\overline{S}$ from
    the states where it isn't.

2. What do you do when {state}?

    Given an input state description $s$ as a partial subset of the communicable predicates, we identify
    the states where the description holds true. For each of these states, we find the action taken
    using the learned policy ($\pi(s) = a$) and use it to group them ($S[a] \leftarrow S[a] \cup s$).
    We then return the reason why that action was taken in that group of states $S[a]$ and not in
    any others.

3. Why didn't you do {other-action}?

    Counterfactual explanations are generated by finding the set of states where the query action
    is the one executed by the learned policy. The algorithm in the paper scans the set of states
    a certain distance away from the current state. It generates a reason for why the query action
    was taken in those states, why the actually executed action was taken in the current state, and
    returns the difference between the two.

## Experimental Setup

The authors conduct experiments using 3 different environments with different agents. For each
environment, policy explanations were generated in response to queries of all three types.

1. GridWorld (discrete, planning)

    The authors trained a tabular $\epsilon$-greedy Q-learning agent.

2. CartPole (continuous)

    The authors trained a neural network-based agent using Q-learning.

3. Robot Inspection (multi-agent, complex, dynamic)

    The authors wrote a traditional conditional-logic based agent for this environment. It is unclear
    whether the environment was simulated or physical.

The authors have not described how the agent policy was approximated, how the distance metric was
implemented or what the templating system looked like.
{: .notice--warning}

## Results and Analysis

The authors generate a policy summary for each domain and compare it to a policy summary written by
a human expert in terms of the communicable predicate sets for the domain. The comparison is
qualitative and presented in terms of "closeness" to the expert summary.

The authors present an example of a counterfactual explanation generated for the robot inspection
domain.

The authors do not describe what a policy summary is, nor how it is generated. Presumably, it is the
set of explanations for queries regarding when an action is performed, for all possible actions.
{: .notice--warning}

## Discussion

The authors use evidence of their method being able to generate explanations in a variety of
environments and for a variety of agents as confirmation of their hypothesis. They present qualitative
arguments for how it can improve policy understanding and aid in debugging agent behaviour.

The authors propose as future work -

1. using parsed natural language descriptions of valid task strategies to map advice down to state
regions for biasing exploration during policy learning

    This is unclear, but since it deals with a better way to learn the agent policy, it is not too
    important to understand since any SOTA method will work.
    {: .notice}

2. explaining human behaviour using inverse RL

## Impressions

### Unnecessary discussion of logging

This paper devotes an entire section to describe how they add logging to an existing agent code in
order to obtain the $(s, a, s')$ tuples used to learn the agent policy. This is unnecessary since
most RL setups utilize an environment which includes an interface to input the action and receive
the reward and the next state. This logging is already in-built into most RL libraries in use today
like OpenAI Gym. It is a solved problem.

### Lack of explanation evaluation

This paper does not have a robust evaluation of the generated explanations. The focus of the paper
being using the explanations to understand and debug agent behaviour, it seems strange that tasks
which involve predicting agent behaviour (to test understanding) or debugging are not evaluated using
human studies. I'm not sure what task the latter would involve. An idea is to provide an agent which
behaves contrary to a specification and have users query it for explanations until the cause is found.
Their experience can be recorded using qualitative surveys.

Additionally, it seems unlikely that explanations with large communicable predicate sets will be
very explainable. Even the policy descriptions presented in the paper seemed to border on being
confusing and overly verbose. Given that the environments used in the experiments seemed very basic,
with small state and action spaces, it is very likely that this problem will only get bigger as the
environments get more complex.

### Hand-authored communicable predicate sets

The explanations rely heavily on using meaningful predicates which capture relevant information in
the state. In certain planning-based domains, it is possible to have a 1:1 mapping between the state
features and the predicates. In other domains, it is likely that a lot of authorial design and
creativity will be needed to come up with a predicate set that is -

1. representative enough of the original state to enable generation of meaningful explanations
2. meaningful to an end user

I find it a possibility that with a poorly chosen predicate set, we might get either overly short
(or empty) or extremely long explanations due to the Quine-McClusky algorithm being unable to effectively
minimize the given expression.

Additionally, for domains which use raw features like pixel data or audio, it is cumbersome to design
features which could effectively represent such states. If used as is, the explanations would be
meaningless to any human (e.g., I moved up because pixel (1, 1) and pixel (2, 1) and not pixel (3, 2) etc.)

### Focus on natural language explanation

I found the focus on providing natural language queries and explanation unnecessary. Neither does
the paper need it, nor does it actually use it in any meaningful way. Template-based string
generation is a very crude way of doing it and not worth focusing on. The queries are also seemingly
provided in terms of just the parameters since there is no mention of a parsing and identification
algorithm from natural language.

Additionally, explanations need not be in purely natural language. They can comprise of any multimedia.
They are akin to the presentation logic for a narrative in narrative generation. The model for an
explanation (the DNF formula) should be able to generate explanations in any desired format.

A list of other minor issues I found in the paper are -

1. The environment model is stated as required to be learned but is seemingly not used anywhere else.
It is possible I have misunderstood something somewhere.
2. The behaviour model $G = (V, E)$ input to the explanation generation algorithms is not adequately
described beforehand.