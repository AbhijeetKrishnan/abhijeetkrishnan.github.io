---
title: Paper Summary - *Policy Distillation* by Rusu et. al. (2016)
tags: paper-summary
mathjax: true
excerpt: "Summarizing and providing my notes on the paper - Policy Distillation by Rusu et. al. (2016)"
---

Rusu, A. A., Colmenarejo, S. G., Gulcehre, C., Desjardins, G., Kirkpatrick, J., Pascanu, R., ... & Hadsell, R. (2015).
Policy distillation. *arXiv preprint arXiv:1511.06295*. [<i class="far fa-file-pdf"></i>](https://arxiv.org/pdf/1511.06295))
{: .notice--info}

## My Objectives

To learn about a method which can be used to extract a policy from a larger RL agent network and use
it to train a smaller network which potentially has similar performance or brings other value. I hope
to use a model which can provide explanations regarding the learned policy.

## Paper Summary

The paper presents *policy distillation* as a novel technique to learn the policy of a trained DQN
agent using a smaller student model. It experiments with 3 different loss functions in learning the
policy using the same model. It demonstrates how learning using a compressed model preserves performance.
Finally, it demonstrates the utility of this technique for multi-task RL by learning a single policy
from multiple single-task DQNs.

## Methodology

Policy distillation attempts to transfer the $Q$-function of the teacher model $T$ to the student
model $S$. This implies that the teacher model must be able to output $Q$-values given the input
observation.

If we use neural networks, they must classify the observed state into an action. The preceding layer
before softmax must therefore be indicative of a $Q$-function since it is a state-dependent
distribution over the actions. It is probably not necessarily correlated with the actual expected
rewards, it could be very unstable and unbounded.
{: .notice}

Using $T$, we generate a dataset $\mathcal{D}^T$ with records of the form $(s_i, \mathbf{q_i})$, where $s_i$ is the
input state observation (possibly a sequence of state observations) and $\mathbf{q_i}$ is the predicted
vector of $Q$-values output by $T$. The student model $S$ is then trained using supervised regression
to mimic the predictions of $T$. The paper experiments with 3 different loss functions to train the
student model which have different properties.

1. negative log-likelihood loss (NLL)

    The authors hypothesize that training a student model to predict only the single best action
    may be problematic because there could be multiple actions with similar $Q$-values. This loss
    function optimizes for predicting the highest valued action from the teacher.

    $$
        \mathcal{L}_{NLL}(\mathcal{D}^T, \theta_S) = -\sum_{i=1}^{|\mathcal{D}|} \log P(a_i = a_{i, \text{best}} | x_i, \theta_S)
    $$

2. mean-squared-error loss (MSE)

    The authors hypothesize that directly approximating the $Q$-function of $T$ using $S$ is problematic
    because the $Q$-function is not bounded and can be unstable. It is moreover computationally
    challenging to solve the $Q$-value evaluation problem. This loss function optimizes for predicting
    the teacher's $Q$-values accurately.

    $$
        \mathcal{L}_{MSE}(\mathcal{D}^T, \theta_S) = -\sum_{i=1}^{|\mathcal{D}|} ||\mathbf{q_i}^T - \mathbf{q_i}^S||^2_2
    $$

3. Kullback-Leibler divergence (KL)

    The authors follow the distillation setup of Hinton et. al. (2014) in using KL-divergence with
    a softmax temperature $\tau$. This "sharpens" the embeddings before the softmax layer allowing
    the student to learn more easily. The authors diverge here from a traditional classification
    setting where the temperature is raised to soften the distribution of softmax, allowing more
    secondary knowledge to be learned.

    The sharpening here is very similar to sharpening done with images, such that it is easier to
    threshold them later. Sharpening the $Q$-values is so that it becomes easier to select the correct
    action i.e., the one taken by $T$. I'm not sure how softening it helps with learning "secondary"
    knowledge, but I can refer to Hinton's paper for more details.
    {: .notice}

    $$
        \mathcal{L}_{KL}(\mathcal{D}^T, \theta_S) = -\sum_{i=1}^{|\mathcal{D}|} \text{softmax}(\frac{\mathbf{q_i}^T}{\tau}) \ln \frac{\text{softmax}(\frac{\mathbf{q_i}^T}{\tau})}{\text{softmax}(\frac{\mathbf{q_i}^S}{\tau})}
    $$

For multi-task policy distillation, the paper recommends using single-task networks trained separately
on each task and generating a combined dataset which the student model learns from. The authors
experiment with KL and NLL loss functions and also compare the performance of a multi-agent distillation
agent with a multi-agent DQN agent (i.e., a DQN agent trained using the same combined dataset).

## Experimental Setup

The authors use a number of games from the Atari benchmarks, selecting games along the spectrum for
which SOTA methods perform compared to human benchmarks i.e., games where they achieve superhuman
performance (e.g., Breakout, Space Invaders) to games where they achieve below-human performance
(e.g., Q*bert, Ms. Pacman).

To avoid issues of memorization of starting states, the authors used starting states generated from
expert-human play.

The authors first use the same model for teacher and student and compare the performance of the
different loss functions. They then use compressed versions of the teacher network as the student
and compare their performance. Finally, they train a multi-game distilled network using two different
loss functions and compare their performance to the single-game agents, and a multi-game DQN agent.
DQN agents were used as the teacher models for most of the experiments. Metrics used were the score
achieved in the game.

Finally, the authors experimented with an online policy distillation procedure where teacher model
checkpoints from training were used periodically to train the distilled network.

## Results & Analysis

KL-divergence achieved the best performance among the loss functions. 25%, 7% and 5% compressed versions
achieved comparable (and sometimes better) performance to the teacher DQN. The multi-game distilled
network achieved a geometric mean of 89.3% when comparing the scores it achieved to the scores
achieved by single-task DQNs trained individually on various games. The distilled model outperformed
the multi-task DQN model, with NLL and KL showing similar performance on games where superhuman scores
had been achieved, but with KL outperforming NLL on games where sub-human scores were the SOTA.

For online policy distillation, the authors observed that the distilled agent was much more stable
than the teacher while achieving similar performance across all games.

## Discussion

The authors hypothesize the reasons for various things in the paper. One example is their discussion
on loss functions and the difficulty in learning the $Q$-function of a teacher. Regarding multi-task
learning, they hypothesize that multi-game DQN learning fails to achieve single-game performance
because of interference between the different policies, different reward scaling and the inherent
instability of learning value functions.

The authors hypothesize that the reason MSE fails to produce good distilled agents is because many
$Q$-values are similar for many actions. For NLL, they hypothesize that minimizing the NLL could
amplify the noise inherent in the teacher's learning process.

## Impressions

This method seems similar to how models were learned in both Madumal et. al. (2019) and Liu et. al.
(2018). It might of course be the paper that influenced or initiated the use of policy distillation
in all those later papers. The main take-away seems to be the use of KL-divergence as the loss function.
Potentially, reducing the model size might also help. However, the key to me is to generate useful
explanations, which requires a model amenable to doing so. The training of that model might very well
have to change depending on its nature. Policy distillation obviously applies to models which update
weights using gradient descent (which models don't nowadays?) and also require a teacher model which
produces $Q$-value vectors (if using KL-divergence).
