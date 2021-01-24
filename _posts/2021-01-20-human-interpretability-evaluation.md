---
title: Paper Summary - An Evaluation of the Human-Interpretability of Explanation by Lage et. al. (2019)
tags: paper-summary
mathjax: true
excerpt: "Summarizing and providing my notes on the paper - *An Evaluation of the Human-Interpretability of Explanation* by Lage et. al. (2019)"
---

Lage, Isaac, et al. "An evaluation of the human-interpretability of explanation." *arXiv preprint arXiv:1902.00006* (2019). [<i class="far fa-file-pdf"></i>](https://arxiv.org/pdf/1902.00006.pdf)
{: .notice--info}

## Why I'm reading this paper

To obtain insights on how the interpretability of an explanation is evaluated, such that I can apply it to my own project on generating interpretable strategies for board games from RL models trained on those games.

## What is the paper trying to answer?

The paper is trying to find general design principles for explanations which are provided by explainable ML systems, in order to make them more human-interpretable. Prior work has not tried to investigate *what* makes an explanation useful to a human. If an explainable system outputs as an explanation a decision tree which is 5000 nodes wide, is it still interpretable? What if it was 5 nodes wide? What if the domains in which the explanation was provided were different? This paper addresses and attempts to answer these questions.

## Methodology

The paper focuses on analysing the effects of explanations embodied by *decision sets*. Decision sets (or rule sets) are simply lists of if-then rules specified using first-order logic. Reproducing Figure 1 from the paper below -

$$
(calm \lor nodding) \land sunny \rightarrow (spices \lor vegetables) \land grains \\
(rainy \land grumpy) \lor calm \rightarrow dairy \lor vegetables
$$

The variables in the LHS represent features of the input, and those in the RHS represent features of the output. In the context of medical decision support, we might imagine input features like `respiratory-illness == Yes`, `risk-depression == Yes` and `BMI` $\geq$ `0.2`, and output features like `Depression`, `Diabetes` and `Lung Cancer` (basically representing diagnoses). These examples were sourced from the paper by [Lakkaraju et. al., 2016](https://www-cs-faculty.stanford.edu/people/jure/pubs/interpretable-kdd16.pdf), so check out their paper for more details.

The authors conducted several experiments using human performance on certain tasks using decision set explanations. They chose three different dimensions along which to vary their explanations -

1. **Explanation size**: the total number of lines in the decision set, and the number of terms in the output clause
2. **Cognitive chunks**: a repeating clause in disjunctive normal form that occurs throughout the decision set. If it is explicitly names and listed as a separate line, it is called and explicit cognitive chunk. Else, it is called an implicit cognitive chunk.
3. **Repeated terms**: the number of times that input conditions repeated in the decision set.

They chose two different domains to ground the explanations in - recipe recommendation (low-risk) and medical diagnosis (high-risk). In both settings, the output was communicated to be for aliens, which is a clever way to negate the influence of real-world background knowledge on the tasks.

The tasks in question were of three types -

1. **Simulation**: given the system's explanation (presumably global), and a set of input observations, can you predict the output that the system would produce?
2. **Verification**: given the system's explanation, a set of input observations, and the system's output, do you think that the output is consistent with the input given the explanation?
3. **Counterfactual**: given the system's explanation, a set of input observations, the system's output and a perturbation to the input (basically a change to one dimension of the input), do you think the output would still be consistent with the perturbed input given the explanation?

The performance of the human subjects was measured using three metrics - response time, accuracy and subjective satisfaction (as self-reported using a 5-point Likert scale). These metrics were chosen since the authors believed it could be applied to a wide variety of tasks. They expect real domains to have much more specific metrics. For example, my project would have explanations evaluated by a gain in playing strength of the player receiving them.

The paper then describes in great detail the conditions of the experiment, the experimental interface and the participant demographics. They were sourced from Amazon Mechanical Turk. The details of how some participants were excluded from the study to account for the incentives of AMT itself are interesting. I wonder if there is a more formal study accounting for the differences in behaviour of or the validity of user studies done using populations from AMT as opposed to the standard WEIRD populations. Perhaps they overlap anyways.

The authors ran a Chi-squared test of independence for each demographic variable. I'm not sure why they did so. A Chi-squared test of independence checks if we can use the level of one variable to predict the other. In this case, perhaps demonstrating independence is meant to show that the sample of test subjects is randomly sampled?

## Results

There were six experiments in total - for each of the two domains and for each of the three kinds of variation in explanations. The authors reported a graph of the metrics across all six experiments. Significance was analysed using linear and logistic regression (where appropriate).

The main findings were -

* greater complexity results in longer response times, but the magnitude of the effect varies by the type of complexity: this isn't surprising to me. What was strange was that more chunks increased response times, and explicit chunks increased it in the recipe domain as opposed to implicit chunks. I would have predicted that implicit chunks would have increased response time.
* consistency of trends across domains
* consistency of trends across tasks
* consistency of trends across metrics

## Discussion

The paper shows that generating decision set-based explanations with fewer lines and implicit chunks can make for more easily parsed explanations. Additionally, framing the evaluation task in terms of simulation seems to be faster and presumably easier than the other tasks.

Unpacking the unexpected increase of response times in the presence of more cognitive chunks might be useful. Does it happen when there are a greater number of chunks overall? What about in different types of domains? I hypothesize that if the chunks used in the explanation correlate with the chunks already used by the subject, response time and accuracy will improve. This is where user modelling comes into the picture for generating better explanations.

The authors themselves pose a lot of these questions and more about the effect of the experimental interface used, which kind of explanations are best for what contexts, universals which exist independent of the nature of the explanation (decision sets, decision trees, pseudo-code etc.)

## Impressions

This is a very well-written paper. It is easily understandable and all the decisions made for the experiments have been well-justified. I probably won't use decision sets for my own project (I'm leaning more towards something like a DSL), but the rigour of the experiments done is admirable and very useful as a reference. The introduction and related work sections are very informative and have several useful references that I want to follow up on. The description of the tasks used to test interpretability are very useful and have made concrete a lot of previously vague notions for me. The metrics presented are interesting, and while I can think of more task-specific ones, I understand that the authors have specifically chosen them for their generality. I haven't yet verified whether the analysis of the data is sound, but I'm inclined to believe the results. An important caveat of this paper is that all explanations used in the experiments were hand-written, and not generated by an existing algorithm. This doesn't strike me as very problematic since, even if it doesn't match the capabilities of current explanation generators, can serve as a useful benchmark.
