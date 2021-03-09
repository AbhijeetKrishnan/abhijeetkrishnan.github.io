---
title: "Paper Summary - *Synthesizing Interpretable Strategies for Solving Puzzle Games* by Butler,
Torlak & Popović (2017)"
tags: paper-summary
mathjax: true
excerpt: "Summarizing and providing my notes on the paper - Synthesizing Interpretable Strategies
for Solving Puzzle Games* by Butler, Torlak & Popović (2017)"
---

Butler, E., Torlak, E., & Popović, Z. (2017, August). Synthesizing interpretable strategies for
solving puzzle games. In *Proceedings of the 12th International Conference on the Foundations of
Digital Games* (pp. 1-10).
[<i class="far fa-file-pdf"></i>](https://par.nsf.gov/servlets/purl/10049037)
{: .notice--info}

## Paper Summary

This work presents a method to automatically uncover user-interpretable strategies for solving a
logic-based puzzle game called Nonograms. The method involves using a DSL to describe a pattern-
condition-action rule which can be applied to solve states in the game. Sound rules in this format
are uncovered using an SMT solver, and the rules sets are optimized for coverage and conciseness.

## Methodology

### Nonograms

Nonograms is a popular deductive logic-based puzzle game similar to [Sudoku](https://en.wikipedia.org/wiki/Sudoku)
or [Kakuro](https://en.wikipedia.org/wiki/Kakuro). It involves
a square grid with integer hints provided for each row and column. The goal is to select the squares
to fill in such that the number of squares in each contiguous sequence matches the integer hint
provided. It involves examining the constraints on the blocks, both from the integer hints and based
on what has already been filled in and deducing which subsequent blocks need to be filled.

The [Luna Story](https://play.google.com/store/apps/details?id=com.healingjjam.picrossluna3&hl=en_US&gl=US)
series by Floralmong is an excellent mobile app for trying out Nonogram puzzles.

### DSL for Nonograms

The DSL is designed to be human-interpretable, and to encapsulate strategies for solving Nonograms
in the form of a pattern-condition-action rule.

The pattern contains constructs that allow parts of a state (a line in a Nonogram level, its integer
hints and currently filled/unfilled blocks) to be referenced later. It is basically a system to
allow binding the state to the given pattern. It is designed to include Nonogram-state concepts like
hints, blocks and gaps (sort of like state features). The bindings themselves have levels of
generality.

The condition describes *when* the action described in the rule can be applied. The action as is
designed in the paper only allows for filling in a block.

Finding interpretable rules in this DSL proceeds in 3 steps -

1. Specification Mining
2. Rule Synthesis
3. Rule-set Optimization

### Specification Mining

Given a training set consisting of Nonogram states $s$, an SMT solver is used (along with a specification
of Nonogram rules) to calculate the maximally filled state  $t$ that can be achieved from $s$. These
*transitions* $\langle s, t \rangle$ are the output from this stage.

### Rule Synthesis

In this phase, an SMT solver is used to find a DSL program which *includes* the transition
$\langle s, t \rangle$ and is sound with respect to the game rules. This means that the transition
holds for the rule condition and the action obeys the game rules. Limitations of the SMT solver like
requiring a finite bound for program size and soundness are addressed by iteratively increasing the
program size for the search.

The generated rules are modified to make them more general and concise. The former is achieved by
modifying the patterns and conditions of rules by brute-force enumeration. Patterns are modified by
replacing bindings with their more general versions and checking if the rule is still sound.
Conditions are modified by synthesizing a new program that covers strictly more states than the
generated one.

The latter goal of conciseness is achieved by using a designer-provided cost function which provides
a quantitative measure of the complexity of a rule. New rules are synthesized and kept if their cost
is less than that of the current rule.

### Rule-set Optimization

The rules obtained from rule synthesis are pruned by selecting a subset of $k$ rules which best
cover the states in the training set. Here, coverage is measured by the total number of cells filled.

## Experimental Setup

The testing data is obtained from commercial Nonogram puzzle books and digital games. The train data
is presumably obtained from a subset of this data with restricted line lengths. Crucially, all puzzles
are able to be solved by considering a line at a time and don't require any guesswork. Individual
states were obtained from the process of solving the puzzle using the SMT solver.

The paper does not mention the cost function used
{: .notice--warning}

The authors encoded *control rules* in the DSL using strategies sourced from puzzle books and
strategy guides for Nonograms. These serve as a benchmark for evaluating the rules recovered by the
system.

## Results and Analysis

The system was able to recover 9 out of the 14 control rules. The authors hypothesize that with
slight modifications, the system would be able to recover the missing rules as well. They note that
the existing system recovered rules which covered much of the same states as the missing control
rules.

The *coverage* of the control rule set and learned rule set are compared. Coverage is the number of
cells covered by applying the rules in the set to the transitions in the test set. The learned rule
set covers nearly $98\%$ of the transitions covered by both together.

This notion of coverage is not intuitive. I would think coverage would measure the proportion of
transitions to which a rule set *applies*. The notion in the paper is a measure of how many cells
are correctly covered using the rule set across a variety of states, and is more a measure of the
effectiveness of a rule set.
{: .notice--warning}

## Discussion

The authors clarify their goal as not generating human-interpretable strategies, but generating
strategies that humans are likely to use. Possible lines of investigation involve using player
solution traces for cost estimation.

They discuss how their work ties into puzzle game generation tools.

Lastly, they discuss the applicability of this work to other puzzle games.

## Impressions

This is an excellent paper, one that I've read earlier, and was responsible for instigating my
current project. The background material provided is excellently detailed, and helped me identify
my current project as being situated in *automated game analysis* as well.

The domain used doesn't seem to have a lot of potential for strategy, given that the authors
restricted themselves to Nonogram puzzles which could be solved line-by-line. Extending it to Sudoku
while preserving that constraint would be very restrictive, since most decent Sudoku puzzles require
some amount of guesswork and backtracking and cannot be solved using only deduction. This would
impact the availability of an oracle to generate transitions (*solved* states).

The state space representation is also very simple, since we only need a single line. However,
using more sophisticated state spaces for other domains would necessitate a better DSL, which is
a design task.

There is a lack of assessing the interpretability of the uncovered strategies. A robust user
evaluation would help alleviate concerns. Overall, the evaluation of the uncovered rules is rather
simplistic. It is possible that the rules don't make very much semantic sense to a human, or aren't
as intuitive, despite the increased "coverage". Providing some examples of the learned rules would
help develop a qualitative understanding of the type of rules learned, and conducting the aforementioned
human trial would add quantitative support.

This method involves generating a dataset of game states and the associated next best move, and
finding rules (DSL programs) which map onto them. Finding the next best move might be simple in
logic-based games like Nonograms and Sudoku where a solver can practically find the best squares to
fill (even with guesswork), but this is not so clear in games like chess, where the best move is
dependent on who's making it (i.e., the strength of the engine). Perhaps we could in fact generate
transitions using a particular engine and design a DSL to describe "chess strategies" and learn
them from the training set. I believe the DSL design is going to be a major bottleneck.

*[DSL]: domain-specific language
*[SMT]: satisfiability modulo theories
