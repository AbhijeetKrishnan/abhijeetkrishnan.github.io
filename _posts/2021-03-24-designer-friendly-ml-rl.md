---
title: "Video Summary - *Designer-friendly Machine Learning and Reinforcement Learning for Video
Games* by Adith Swaminathan & Brian Broll (2021)"
tags: video-summary research
mathjax: true
excerpt: "Summarizing and providing my notes on the research talk - Designer-friendly Machine 
Learning and Reinforcement Learning for Video Games by Adith Swaminathan & Brian Broll (2021)"
---

Swaminathan, A., Broll, B. (2021, February 23-24). *Designer-friendly Machine Learning and
Reinforcement Learning for Video Games* [Research talk]. AI and Gaming Research Summit Day
1 Track 1 Part 2, Virtual. <https://www.microsoft.com/en-us/research/event/aiandgaming2021/>
[<i class="fab fa-youtube"></i>](https://youtu.be/rVvfJ1u5zDU)
{: .notice--info}

The theme of this talk was about how ML and RL can be used by game designers to make the process of
designing and testing games more painless and efficient. The speakers hypothesize that ML/RL will
play an increasing role in game dev and will move from spectacular, one-off demos (e.g., AlphaGo,
AlphaStar, OpenAI Five) to routine deployment. They recognize that game design goals are difficult
to specify as reward functions or labels. They identify three key ways in which ML can become more
accessible to game designers, and highlight one work for each.

## Interpret

ML and RL agents are black boxes. Their policy-making and inner workings need to be interpretable by
game designers so that they can appropriately respond to the agents' observed behaviour. 

The speakers highlight a portion of the work by [Broll et. al. (2019)](https://sites.google.com/view/customizing-scripted-agents/home)
in which the authors attempt to create imitation learning components to imitate human-like behaviors in
Minecraft. Evaluating the "human-likeness" of behaviour is time-consuming, since one has to manually
examine many different rollouts (episodes of play by the agent). The authors came up with a
visualization tool for a single rollout which could indicate actions that 'would have been taken'
(counterfactuals) were the agent in a different location of the state. This could be done with any
state dimension other than location. Playing through the rollout frame-by-frame, the speaker noted
that the actions stayed relatively stable when the enemies were far away from the agent, but the
actions changed sharply and erratically when the enemies surrounded the player. This indicated that
such situations were perhaps too few in the training set, and that more should be added.

Broadly, there need to be game design-centric visualization tools to help game designers
understand how agents behave. This could either inform how the agent behaviour needs to be changed,
if the agents are themselves part of the game, or how the game itself needs to be changed, if the
agents are being used merely for playtesting.

## Influence

AI agents are deployed in games for specific purposes. It would give a great deal of creative
expression to game designers if the behaviour of powerful agents could be customized to suit their
game design goals. Designers need to be provided with easily-used "knobs" which can tweak the
behaviour of agents in meaningful ways.

The speakers highlight the work by [Zhan et. al. (2020)](http://basketball-ai.com/). In this, they
apply the concept of *labeling functions' from weak supervision learning to the problem of imitation
learning. They use designer-provided labeling functions which can classify a basketball player's
trajectory on court using three metrics and three styles within those metrics (e.g., destination -
close by, medium, far away). The idea is to take as additional input these style settings and use it
to generate a trajectory. The labeling functions are applied to the generated trajectory to produce
an error with the original settings, which is backpropagated to update the policy.

Broadly, there needs to be more focus on providing easy-to-use tools for meaningfully changing the
behaviour of trained RL agents while not having to significantly alter the agent itself (e.g., train
a whole new agent). The speakers speculate that perhaps these control knobs requiring designer input
offloads too much work to the designer, and consider the possibility of having more abstract control
knobs.

## Interaction

The interaction loop between AI agents and game designers is hard to study. Agents may be deployed
for applications like playtesting, but the combination of how designers interpret and influence the
agents needs to be better observed.

The speakers present some preliminary efforts towards this by introducing a game environment geared
towards studying automated game balancing. This is called [0 AD](https://github.com/brollb/game-balancing-example-0ad)
and is an open-source real-time strategy game akin to Age of Empires that allows for writing custom
bots to explore the balance of the game. Playing multiple games against each other will reveal
effective strategies that will inform the designer of how they wish to change the game.

The speakers conclude by mentioning the biggest opportunity of this research area as being able to
do **automated game design and playtesting**. They mention the biggest challenge as doing
**player modeling in rich game environments**.
