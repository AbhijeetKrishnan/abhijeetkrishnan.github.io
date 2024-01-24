---
title: My First Publication
tags: research deep-dive personal
mathjax: true
excerpt: "A retrospective on the problem-solving path that led to my first publication."
---

I submitted my first research paper to the [16th AIIDE](https://webdocs.cs.ualberta.ca/~santanad/aiide/) conference on May 28th, 2020 at 7:40 PM GMT. It had been nearly a year's worth of work that had gone into the paper. I would like to share the research journey which culminated in the final publication.

I began work in Spring 2019 when my advisor offered me the project due to their grant for it being funded by the [LAS](https://ncsu-las.org/). The original whitepaper was for modelling user mental models using a rule-based model (think [Prolog](https://skolemmachines.org/ThePrologTutorial/www/prolog_tutorial/pt_framer.html) statements) and attempting to "align" it to a correct model or understand how it differs. The choice of using a rule-based model was arbitrary and was motivated by my advisor's prior experience with them, as well as the belief that using a rule-based model would make the user models yield some explanatory power about the user. Since we are a rather games-focused lab, we were going to use games and players as our test bed.

The value to LAS from this project was in having a way for their intelligence officers to learn software faster since the results from our research could inform ways to accelerate their usage of internal software, provide shortcuts for oft-used operations and generally speed-up the learning process. The potential research results mentioned in the white paper were for general software interaction principles which could inform UI/UX design. The results I was hoping to get out of it, and my initial research direction, was a way to generate levels for games based on the learned player model, which could uniquely challenge the player.

Going into the project, I found myself clueless about a lot of things. I did not know how to model a player's mental model computationally (or what prior work existed). I did not know enough about rule-based systems I could use as the language of the player model. I had only a passing familiarity with different logic systems (e.g., epistemic, first-order, temporal) as a result of past AI courses I had taken. To remedy this, I began to trawl through the literature. My initial lit reviews led me to the fields of *student modelling* and *intelligent tutoring systems*, which I found a lot of relevant material in.

I was fortunate to have several professors and PhD students in adjacent labs in my department who had published in these fields. [Samiha Marwan](https://www.samihamarwan.com/) explained to me her research on knowledge components, which is a common way to model student learning in computer-assisted education research. [Dr. Noboru Matsuda](https://www.csc.ncsu.edu/people/nmatsud) had developed [SimStudent](http://www.simstudent.org/), which was a computational student model that could be "trained" like a student. This was very much like what I wanted to accomplish, and it used a rule-based representation for the artificial student model. From the paper, I discovered that it used an older student modelling system called FOIL, which could generalize over first-order logic statements provided to it, mimicking human induction. I could not find a working version of this system, so I tried to look for alternatives.

I also separately set up my test bed. I used a game called [Laserverse](https://gist.github.com/ryansalexander/9eef9fcacdee542b842f844629599052), which is a block-based puzzle game developed in [PuzzleScript](https://www.puzzlescript.net/) which centres around bouncing lasers off mirrors to open doors to the goal. The game was developed by UG researchers in the POEM Lab during a Summer REU project, which culminated in a [publication](http://ceur-ws.org/Vol-2282/EXAG_121.pdf). I would [previously worked](/assets/docs/CSC_791_Project_Report.pdf) to develop and ASP-based level generator for the game, so I was familiar with it. The authors had provided extensive documentation and were available in-person for help. I had to [fiddle a bit](https://github.com/AbhijeetKrishnan/PuzzleScript) with the PuzzleScript source code in order to log player actions taken in-game.

My foray into student modelling was very interesting, but ultimately did not yield anything useful. I learned a good bit about how student models in the earliest ITSs worked. However, there was no work that I could find which used a logic-based representation of a student model. Student modelling did make me refine my thinking about the project, however. In particular, the idea of constraint-based student models made me question if I could identify a player's actions as right or wrong, at least in the context of a puzzle game. This helped me course correct later in the project. I also toyed with coming up with a logic-based student model of my own, since my advisor has worked on [Ceptre](http://lambda-the-ultimate.org/node/5216), which is a linear logic-based language designed to model game rules. However, the feeling that "something like this must already exist" led me to keep looking for existing solutions. I also investigated some psychology papers for insight into how players form mental models of games they play.

I ran into the field of inductive logic programming (ILP). This field is concerned with developing methods to induce logical rules from sets of ground truth statements written in first-order logic. My biggest stumbling block here was a general unfamiliarity with the field. I found a very useful [textbook](https://www.springer.com/gp/book/9783540629276) on ILP which helped me navigate the ILP papers I had to read.

ILP attempts to solve the problem of generalization from examples (i.e., machine learning). Given a collection of positive and negative examples of an (as yet unknown) rule, can you generate a hypothesis which best fits this rule? Say we were given $(x,y)$ pairs $(1,2), (2, 4), (3, 8), (4, 16)$, would we be able to come up with the general rule $y = 2^x$? The general problem of forming a hypothesis based on training examples is well known in machine learning, but ILP attempts to work with training examples and hypotheses presented in symbolic forms, such as predicate logic.

The most modern ILP system I could find was [Metagol](https://github.com/metagol/metagol), written by [Andrew Cropper](http://andrewcropper.com/) for his PhD thesis. The system works using SWI-Prolog and was easy to use. It takes in positive and negative examples of a rule as input, along with some predicate definitions and optional background knowledge, and return a logic program which best describes the examples provided. As a concrete example, I am using directly the code provided in the README of the Metagol project.

```prolog
:- use_module('metagol').

%% metagol settings
body_pred(mother/2).
body_pred(father/2).

%% background knowledge
mother(ann,amy).
mother(ann,andy).
mother(amy,amelia).
mother(linda,gavin).
father(steve,amy).
father(steve,andy).
father(gavin,amelia).
father(andy,spongebob).

%% metarules
metarule([P,Q],[P,A,B],[[Q,A,B]]).
metarule([P,Q,R],[P,A,B],[[Q,A,B],[R,A,B]]).
metarule([P,Q,R],[P,A,B],[[Q,A,C],[R,C,B]]).

%% learning task
:-
  %% positive examples
  Pos = [
    grandparent(ann,amelia),
    grandparent(steve,amelia),
    grandparent(ann,spongebob),
    grandparent(steve,spongebob),
    grandparent(linda,amelia)
  ],
  %% negative examples
  Neg = [
    grandparent(amy,amelia)
  ],
  learn(Pos,Neg).
```

This produces the output -

```prolog
% clauses: 1
% clauses: 2
% clauses: 3
grandparent(A,B):-grandparent_1(A,C),grandparent_1(C,B).
grandparent_1(A,B):-mother(A,B).
grandparent_1(A,B):-father(A,B).
```

A novelty here are the *metarules*, which is a new concept introduced by Andrew Cropper himself for his PhD thesis. They define a kind of meta-predicate of how predicates can be combined. They constrain the sort of hypotheses that can be learned, and a set of simple metarules can be used to learn a wide variety of useful hypotheses.

I worked with ILP for a long time in attempting to get a solution for my player modelling problem. I even [contributed](https://github.com/metagol/metagol/pull/15) a few examples from a DeepMind [paper](https://arxiv.org/abs/1711.04574) on ILP to the Metagol repo. However, I found that Metagol required some very contrived metarules in order to learn simple rules, like movement. I needed to almost provide knowledge of the rule to be learned in the form of metarules (which Andrew explained to me was the biggest limitation of MIL). I also encountered problems with my test bed since I would need a Prolog-based representation of Laserverse to work with Metagol. My prior experience in building a logic-based level generator for the game had taught me that working with logic is *very* cumbersome and fiddly, since expressing mechanics for the game in logic is hard and debugging them when things aren't working is even harder (and annoying, given the lack of sophisticated development tools). I considered the possibility of using a simple game already written in Prolog as a test bed instead.

I tried a lot to get Metagol to learn the mechanics of the game. As an aid, I tried to break down the problem I was solving more formally. I realized it involved recording player actions and the resulting changes in game state, taking this sequence of $S_0 - A_1 - S_1 - A_2 - ...$ and attempting to learn rules about the player - which actions they take in certain states, long-term patterns in their actions and so on. It slowly began to dawn on me that this problem was fundamentally unsupervised, whereas ILP required labelled examples of what matched the rule and what did not. Although there were ILP approaches which could do unsupervised learning, I began to look for alternative approaches, bolstered by the thought that since the problem *felt* so tractable, there must be some prior work on it.

I did attempt a lot to solve it myself, but could not come up with a satisfactory solution, mostly because I was unable to find a way to detect long-term patterns in a sequence. I investigated some data mining approaches but could not find any applicable technique. I finally hit the jackpot when I stumbled upon *action model learning* (AML), a field dedicated to solving exactly what my problem entailed. AML is situated in the automated planning community and is a method to reconstruct a planning domain from examples of action traces. A planning domain is a rule-based description of an environment (usually in PDDL), and an action trace is exactly the $S_0 - A_1 - S_1 - A_2 - ...$ sequence I'd earlier described.

One limitation I found was that AML systems try only to learn the predicates of an *action*, rather than any rules about long-term trends in the actions taken. In the context of games, they would only learn the mechanics of an action rather than deduce which sequence of steps the player tends to perform when confronted with a specific obstacle. I decided to reframe the problem given this limitation as trying to learn a player's mental model *of their knowledge of a game's mechanics* and leave the long-term learning for future work.

I found [FAMA](https://github.com/daineto/meta-planning) as the most modern AML system, and corresponded with the authors for help getting it working (I ran into memory overflow issues frequently, and they were very helpful in their responses). I did encounter papers for older AML systems, but could not easily find their source codes online (although I managed to find some by emailing the respective authors when writing the paper for my [written prelim](/assets/docs/Written_Prelim_Paper.pdf)). I identified Sokoban as an alternative to Laserverse as a block-based puzzle game which fortunately also had a PDDL representation. I also attempted to understand the working of FAMA from the paper and found that it compiled the AML problem into one of planning. It uses a cleverly constructed planning domain for which when a solution plan is found, can be turned into the solution for the AML problem. I faced much more difficulty in understanding the working of older AML systems like ARMS and SLAF, and since I wasn't able to use them, did not pursue the endeavour.

I spent the next few weeks reading the FAMA paper, pursuing some promising leads mentioned there (e.g., human-aware planning), and trying to learn action models using it. I realized during this time that the learned action models themselves were meaningless as a player model, even if they were learned using action traces from human players. The learning process of FAMA is nothing like how a human learns, and so we have no evidence for the learned player model resembling a human's knowledge of the mechanics in any way. Basically, there was no quantitative metric for accuracy or precision, nor did we have a ground truth - the player's *actual* knowledge of the game's mechanics represented in PDDL.

In addition, we didn't have anything useful we would be able to learn from the PDDL player model. I was able to adapt an evaluation metric from the FAMA paper to produce a "proficiency score", which measured how well a player knew the game's mechanics (which again wasn't calibrated against any ground truth). I also speculated that the player model of the game's mechanics could be used as an actual game by supplying it as the input domain to a planner. This had some artistic merit.

I attempted to find a way to calibrate these scores, or to find evidence that the learned player models resembled a human player model, but any solution would require a user study. I found it difficult to organize one due to the COVID-19 pandemic hitting at the time, so opted to mention that we would do so in a future work.

Separately, an undergraduate researcher I was working with came up with a simplified AML algorithm (dubbed *Blackout*) to learn actions from play traces. He used simple principles to verify that the learned predicates matched the play traces and used some novel "invariant"-based heuristics to eliminate spurious predicates. The algorithm shares similarities with the earliest works on AML, and the invariant-based heuristics do not have a formal proof for their correctness. However, quantitative measures of *Blackout*'s accuracy and precision found it comparable to FAMA, and *Blackout* far exceeded it in terms of performance, so we believed we had a publishable result.

I reframed the problem further to present the use of AML as a novel, *domain-agnostic* player modelling technique which could learn a rule-based player model, which described the player's knowledge of a game's mechanics. I described the use of AML to do player modelling and described the results of experiments comparing FAMA with *Blackout*. This became the final draft of the paper which I intended to submit for publication.

My advisor provided a lot of valuable advice in re-writing the paper for publication. First was the problem of length - my paper exceeded a page limit of 6 + 1 by quite a bit. They helped me find the key findings of my paper and discard the irrelevant bits. They also taught me to prop up the positive results of my work more - I had been very critical of the lack of experimental correlation of the learned mental models with actual players and had mentioned a slew of other shortcomings (games are difficult to represent in PDDL, real-time games are probably impossible to etc.) and had devoted a lot of page space to it.

The reviewer feedback was nerve-wracking to receive but was in the end very helpful. The feedback seemed mostly positive, although there were concerns regarding the correlation of player models to actual humans (which I expected). One reviewer's comments made me think about the problem much more generally, leading to my rewrite of it for the written prelim. Overall, I was satisfied with the reviewer comments, and felt that I provided a good rebuttal. I was slightly disappointed with having a poster presentation instead of an oral, however.

The actual presentation during AIIDE was uneventful. A digital poster presentation is not the best way to interact with people, and I would have been much more comfortable with the person physically in front of me while presenting. I was also not able to network and socialize as well as I would have liked. A few people did stop by the Discord channel dedicated to my paper to ask questions, and at least one seemed to want to incorporate my work in their own research.

The main thing to do to further this work is develop a robust evaluation metric for measuring the accuracy of a learned player model. I have suggested one in my written paper and will try implementing it when possible. I believe AML methods can be augmented with other features derived from psychological insights of mental model formation to better align them to an actual player. I am working on methods to interpret RL-models trained on board games, so the results from this work could be repurposed to "interpret" a neural network-based player model into a rule-based model.

The journey was very instructive, and I am happy to have published my first paper. I found that the difficulty of research lies in dealing with a lot of unknown pieces while simultaneously trying to combine them into something useful. I have not improved my sense of any problem space better (since I'm not continuing work in player modelling or AML), but I believe my research skills and process have been sharpened. Specifically, I believe an evaluation-first approach to solving a problem is very useful, and that starting to write your paper from the very first day leads to more productivity.

*[LAS]: Laboratory for Analytic Sciences
*[FOIL]: first-order inductive logic
*[ITS]: interactive tutoring system
*[MIL]: meta-inductive learning
*[AML]: action model learning
*[RL]: reinforcement learning
*[AIIDE]: Artificial Intelligence and Interactive Digital Entertainment
