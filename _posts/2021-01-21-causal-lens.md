---
title: Paper Summary - *Explainable Reinforcement Learning Through a Causal Lens* by Madumal et. al. (2020)
tags: paper-summary xrl-technique
mathjax: true
excerpt: "Summarizing and providing my notes on the paper - Explainable Reinforcement Learning Through a Causal Lens by Madumal et. al. (2020)"
---

Madumal, P., Miller, T., Sonenberg, L., & Vetere, F. (2020, April). Explainable reinforcement learning through a causal lens. In *Proceedings of the AAAI Conference on Artificial Intelligence* (Vol. 34, No. 03, pp. 2493-2500). [<i class="far fa-file-pdf"></i>](https://ojs.aaai.org/index.php/AAAI/article/view/5631/5487)
{: .notice--info}

## My Objectives

To learn about a specific approach which generates explanations for decisions made by RL agents and use it for my own project.

## Paper Summary

This paper tries to develop an algorithm to generate explanations for decisions made by RL models. It introduces a new model based on *structural causal equations* which can be trained alongside the original RL model. This model approximates the policy of the original model. The paper presents algorithms to train this model and use it to generate explanations and perform task prediction. The authors evaluate the task prediction accuracy of this model. The authors perform a human subject study to evaluate the explanations generate by their model. They evaluate explanation satisfaction, accuracy of task prediction by the test subjects, and trust in the agent.

## Methodology

The paper introduces a new model to approximate the policy of a model-free RL agent called an *action influence model*. This model is inspired by *structural causal equations* and is designed to represent causal relationships between state variables in terms of actions i.e., how do actions influence the environment. This is basically equivalent to having an environment model, although the paper acknowledges this and mentions that the objective is to merely learn an approximation good enough for explanations. The environment model is necessary to provide counterfactual explanations - answering "Why not action B?" instead of just "Why action A?". The authors claim this makes for better explanations for humans.

I will intentionally try to be less formal and not use mathematical formalisms to describe the concepts presented in the paper.
{: .notice}

### Action Influence Model

The action influence model can be visualized as a DAG with nodes as the state variables and edges as the structural equation associated with an action. This structure represents how the state variables are causally related to each other. The sink nodes represent state variables which directly contribute to the agent's utility function. The source nodes, I believe, are what the authors call *exogenous* variables - representing external factors, but I'm not sure what a concrete example of this would be. The edges are labelled by an action and represent the set of structural equations which govern how the variable at the edge head changes under the influence of the variable at the edge tail when the action in the edge label is applied. More formally, if we have an edge $(X) \xrightarrow[]{A_s}(Y)$, then we will have a structural equation of the form $Y = f_{A_s}(X)$. This equation is learned using traditional function approximators like linear regression, decision trees and MLPs.

To make this description more concrete, I will try to explain it in terms of Figure 1 from the paper. The figure represents an action influence model learned in a StarCraft II domain. We can see the DAG of the state variables represented. The sink nodes $D_u$ and $D_b$ directly contribute to the agent's utility function. The edge labelled $A_s$ connects node $W$ to node $B$ and would contain a set of equations which describe how $B$ changes w.r.t $W$ when the action $A_s$ is taken.

It is important to note that the structure of the DAG is not learned but is created by hand. I think this has some important implications for how generalizable this technique is. The authors do not describe the design challenges of coming up with a useful DAG.
{: .notice--warning}

### Learning Structural Equations

The structural equations are learned from a dataset of experience frames. During the training of the original agent (for whose decisions the action influence model will provide explanations), we must generate records of the form $(s_t, a, r, s_{t+1})$, where $s_t$ is the state at time $t$, $a$ is the action taken by the agent, $r$ is the rewards obtained and $s_{t+1}$ is the successor state. This dataset can be generated during training (as in experience replay) or presumably post-training as well by executing the agent in an environment.

Given this dataset and the action influence model DAG, we use any function approximator to learn, for every edge $(X) \xrightarrow[]{A_s}(Y)$, the function $Y = f_{A_s}(X)$. The paper does this in minibatches drawn uniformly from the dataset. It uses as function approximators regression learners, decision trees and MLPs.

### Explanation Generation

This model generates local explanations i.e., explanations for a single decision taken by the agent (as opposed to global explanations which explain the overall policy of the agent). Given an action influence model $M$ and an agent action $a$  (and given instantiations of the state variables), we locate all edges in $M$ which are labelled with $a$ and find their head nodes. Starting here, we follow the causal chain till we reach the sink nodes. An explanation as defined in the paper is the set of ancestors of the reached sink nodes, and the set of head nodes. This is basically the immediate reason for doing an action and the immediate causes of rewards. This serves to heuristically simplify an explanation instead of providing the entire causal chain.

Perhaps a model of the explainee could serve to guide *how* to present the causal chain as an explanation.
{: .notice--success}

It seems the explanation is only defined in terms of the causal DAG, which has to be hand-designed. The structural causal equations serve only to *quantify* how much an action influences a causal relationship between two state variables, and this information is not used in the explanation. I wonder if adding this information would improve the explanation. As it stands, all informtion used in the explanation comes from the hand-written causal DAG alone. Every action can be assumed to have some influence i.e. we can label an edge in the causal DAG with every possible action, and learn separate sets of structural causal equations for each of them.
{: .notice--warning}

### Counterfactual Explanation Generation

A counterfactual explanation is the explanation of "Why not B?" given a decision A taken by the agent. A counterfactual explanation as defined in the paper is, given a model $M$, an action $a$ and a counterfactual action $b$, the difference between the causal paths taken from the head nodes of edges labelled $a$ and $b$. Basically, we follow the process of generating explanations for $a$ and $b$ using $M$ and return their difference.

### Task Prediction

This model can perform task prediction i.e., the task of predicting the action that would be selected by an agent given the current state $s_t$. To do this, it uses every structural equation that applies to an action in order to predict the successor state. Recall that a structural equation $Y = f_{A_s}(X)$ gives us the change in $Y$ in terms of $X$ when an action $A_s$ is applied.  We are in fact getting the value of $Y_{t+1}$ from $X_t$. If we obtain the changes in all state variables using all the structural equations that apply to $A_s$, we can obtain a complete description of the successor state $s_{t+1}$. Of course, this will be a prediction made by the model. We need to use this prediction to also predict the action that would have been taken by the original model.

It does so by computing the delta between the predicted successor state and the actual successor state. It selects the action whose structural equation has the highest difference between the predicted state variable value and the actual value i.e., it selects $\underset{A_s}{\operatorname{arg max}} f_{A_s}(X_{t+1}) - Y_{t+1}$ where $f$ represents the set of *all* structural equations in the action influence model.

{% capture notice-danger %}
This way of doing task prediction seems incorrect for two reasons -

1. Task prediction takes as input only the current state. Its output is a prediction of what the action the original model selects would be. This algorithm requires the actual successor state as input, which should be hidden from the algorithm.
2. Selecting the action which has the *maximum* difference between predicted successor state variable and actual successor state variable seems like selecting the *worst* possible action. Wouldn't a better way of doing this be to predict an entire successor state, obtain a measure of the delta between the predicted and actual successor, and select the action with *minimum* delta? The explanation for this method in the paper seems unconvincing -

> This is informed by the reasoning that the agent will try to follow the optimal policy, and the action with the biggest impact to correct the policy will be executed. The impact is measured by the above mentioned difference. This is itself an approximation, but is a useful guide for task prediction.
{% endcapture %}

<div class="notice--danger">{{ notice-danger | markdownify }}</div>

### Computational Evaluation

The paper presents a computational evaluation of the action influence model using task prediction for several RL benchmarks. It calculates the model's accuracy and runtime. It performs reasonably well, although these metrics are not too valuable for its purpose. The benchmarks also have small action spaces, which might misrepresent the capabilities of the algorithm.

## Experimental Setup

The paper hypothesizes two things -

1. Causal-model-based explanations build better mental models of the agent leading to a better **understanding** of its strategies.
2. Better understanding of an agent’s strategies promotes **trust** in the agent.

The authors use StarCraft II as their testbed. They use a reduced version of it with smaller action and state spaces. To test $H_1$, they use task prediction. They use a 5-point Likert scale to measure explanation satisfaction and trust ($H_2$).

The human subject study uses a 22 min. gameplay video of the RL agents (including baselines) playing against the in-game bot. The participants were sourced from Amazon Mechanical Turk. The study consists of 4 phases -

1. Collecting demographic information and training the participants to differentiate the agents' actions. Demographic information included sex, age self-rated gaming and StarCraft 2 experience.

The authors did not run a Chi-squared test for independence of each demographic variable, as in Lage et. al. (2019). As I didn't understand why it was done in that paper, I cannot judge whether it should have been done here.
{: .notice}

2. Gathering data for the explanation satisfaction survey by allowing subjects to query for "Why/Why not?" explanations from 5 different 15 sec. gameplay clips.
3. Gathering data for task prediction. The tasks being predicted are divided evenly between tasks which are similar to those the subjects were trained with, and novel tasks (in terms of the causal chain involved). Scores were given for correct, incorrect and "I don't know" responses and tallied.
4. Gathering data for the trust survey.

The authors describe how they avoided noise in the dataset and controlled for language by only recruiting participants from the US.

The independent variables of the experiment are the techniques used to generate explanations, including one prior work. The four different models are -
1. No explanation (N)
2. template 1 of Khan, Poupart, and Black (2009, p.3) (R)
3. action influence model with detailed explanations (using atomic actions in the causal graph) (D)

This is not adequately explained.
{: .notice--warning}

4. (presumably) the original action influence model (C)

It is unclear how Phase 2 of the study would work for subjects assigned to explanation techniques other than action influence models.
{: .notice--warning}

## Results & Analysis

The null hypothesis was $\mu_C = \mu_R = \mu_D = \mu_N$. The alternative hypotheses were -  
$H_1$: $\mu_C \geq \mu_R$  
$H_2$: $\mu_C \geq \mu_D$  
$H_3$: $\mu_C \geq \mu_N$

Basically, the proposed model was expected to outperform the other models when it came to task prediction scores obtained when the subjects used its explanations. A one-way ANOVA was conducted, and a significant p-value was obtained. Tukey multiple pairwise comparisons was used to obtain significances of differences between groups, and the obtained p-values were significant.

Correlation between the number of questions and the score was not significant.

Differences between the groups remained significant when accounting for only correct answers, only incorrect answers, and only "I don't know answers".

I'm not sure how exactly this division was accounted for.
{: .notice}

Pearson's Chi-Square was done as a non-parametric test on task prediction scores, and it showed significant results.

I'm not sure what the purpose of this test is here.
{: .notice}

The explanation quality data is analysed using a pairwise ANOVA, and the model outperforms the other on all metrics except "Understand".

The metrics are not sufficiently explained in terms of how the questions were presented to the subjects. How were explanations of the 'No explanation' model presented? Lastly, Table 3 does not report the differences with the detailed action influence model explanations (D).
{: .notice--warning}

The data on trust was similarly analysed using the metrics *confident*, *reliable*, *predictable* and *safe*. These were not statistically significant.

Here, too, it isn't clear in what sense these metrics are presented. What is being measured when a subject is asked to rate the safety or reliability of an agent? Are the subjects' notions of these terms consistent? 
{: .notice--warning}

A Pearson's correlation test was run to check for correlation between self-reported StarCraft II experience and task prediction scores. No significant correlation was found. The authors hypothesize that it could be because the reduced version of StarCraft II used as a testbed is sufficiently different that external experience couldn't influence the results.

The authors acknowledge the strong linearity assumption made for StarCraft II allowing linear regression to learn SCMs.

## Discussion

The authors reiterate the paper's contributions and briefly acknowledge the limitation that the causal model must be provided beforehand. They mention using explainee models to target explanations and adapting their model for continuous state variables (the ones in the StarCraft II domain were discrete).

## Impressions

This is an interesting paper and has a nice attempt at explanation generation for RL agents. I believe the issue of requiring a causal model is significant, and it deserved more explanation in the paper. How do we design a causal model for a different domain? It requires some amount of expert knowledge, which is precisely what model-based RL agents try to learn themselves. Perhaps this technique could be adapted to model-based RL agents: namely approximating their model using an action influence model and using the latter to generate explanations. I'm not sure why the paper restricted itself to model-free agents.

The action influence model also seems limited in its ability to represent the causal structure of a domain. It uses a DAG, which does not account for cyclic dependencies in domains (could one exist?). The authors acknowledge that the use of linear regression does not account for non-linear relationships between state variables, but I believe that could be solved by using a non-linear function approximator.

Lastly, the usage of satisfaction and trust metrics to measure explanation quality seems to incentivize explanations which sound good to humans but may not give them a good idea of how the agent works. Task prediction accuracy is a better metric, but then that constrains the model to be simulatable by a human, which is a big limitation. Perhaps a compromise would be accepting a certain score threshold.
