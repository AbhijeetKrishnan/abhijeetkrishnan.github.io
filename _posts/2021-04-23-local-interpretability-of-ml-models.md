---
title: "Paper Summary - *Assessing the Local Interpretability of Machine Learning Models*
by Slack et. al. (2019)"
tags: paper-summary
mathjax: true
excerpt: "Summarizing and providing my notes on the paper - Assessing the Local Interpretability of
Machine Learning Models by Slack et. al. (2019)"
---

Slack, D., Friedler, S. A., Roy, C. D., & Scheidegger, C. (2019, January). Assessing the Local
Interpretability of Machine Learning Models. In *NeurIPS Workshop on Human-Centric Machine Learning
(HCML)*.
[<i class="far fa-file-pdf"></i>](https://par.nsf.gov/servlets/purl/10187139)
{: .notice--info}

## Paper Summary

This paper defines a metric to measure the <abbr title="local interpretability assesses whether
a single decision is interpretable, as opposed to global interpretability which assesses whether the
model as a whole is interpretable">local interpretability</abbr> of a model - the number of
operations the model requires to make a prediction. The metric is correlated with local
interpretability by measuring the latter with a human-subject study. Users were asked to predict the
output of a model given an input, and predict the output given counterfactual input i.e., slight
modifications of the input. The results show that the operation count metric *is* correlated with
local interpretability, and that decision trees and logistic regression models are more
interpretable than neural networks.

## Methodology

The metric defined to act as a proxy for interpretability is the runtime operation count.
Effectively it is the number of operations a human must carry out in order to simulate a single
prediction the model. This was performed by instrumenting the prediction operation for existing
trained models in Python's [`scikit-learn`](https://scikit-learn.org/) package.

## Experimental Setup

The authors ran a crowdsourced experiment $(N=1000)$ using the [Prolific](https://www.prolific.co/)
platform. Participants were asked to predict the output of a model on an input (by replicating the
actual calculations involved), and then asked to predict the output again for a slightly modified
version of the input. Three model types were used - decision trees (DT), logistic regression (LR)
and neural networks (NN). They were trained using `scikit-learn`. Training details are omitted here.

Users were trained in how to perform these calculations for each model type using a small fill in
the blank exercise conducted before the actual prediction task. The models were trained using
synthetic data to avoid the effects of domain knowledge on prediction.

The survey was administered through
[Qualtrics](https://www.qualtrics.com/education/student-and-faculty-research/). Hypotheses for the
study were pre-registered on the [Open Science Framework](https://osf.io/).

## Results and Analysis

Based on the number of correct responses for the two tasks of simulation and counterfactual
prediction for each of the three models, three hypotheses were made regarding local interpretability
of the models - $DT > NN$, $DT > LR$, $LR > NN$. p-values and confidence intervals were calculated
for each using [Fisher's Exact Test](https://en.wikipedia.org/wiki/Fisher%27s_exact_test).

The Fisher Exact Test gives exact p-values for $2 \times 2$ contingency tables, where samples from
$2$ different treatments can be classified in $2$ different ways. In the case of this paper, an
input from $2$ different models is classified into correct or incorrect based on the user's
response. Fisher's Exact Test then tells us how *likely* the particular assignment of values to each
cell are, with the null hypothesis being independence of all categories. It is a special case of the
chi-squared test but works for all sample sizes.
{: .notice}

The results show that decision trees are more simulatable and "what-if" locally explainable than
logistic regression or neural network models. However, the results did not find evidence for
logistic regression to be more locally interpretable than neural network models.

Visual comparison of the plots of accuracy vs. operation count across all three tasks
(simulatability, "what-if" local explainability and local interpretability, the last of which simply
measures how many users got both of the previous tasks right), shows that they *are* correlated.
Also graphed was the time taken vs. the operation count, and the accuracy vs. the time taken.

## Discussion

The authors point out how this metric may be used to assess the interpretability of a model without
a user study.

## Impressions

This is a neat paper which attempt to quantitatively validate a hypothesis that most people would
lend credence to being true. Operation count is of course a very crude way to approximate
interpretability, and is absolutely unfeasible for the mammoth network sizes that power today's
models. The paper also doesn't attempt to answer questions regarding interpretability for different
models with similar operation counts. Is a large decision tree equally as interpretable as a small
neural network? The graphs seem to show that the mean operation counts across the three model types
are significantly different. I would have liked to see how interpretability is affected when the
operation counts for different model types are similar.

I also question whether brute calculation of the model to produce outputs is how most people would
try to interpret a model. One can probably build some intuition from the feature weights and
response curves to be able to make accurate predictions for the smaller model types *without* having
to actually simulate the prediction function on paper. Nevertheless, it is a simple and good enough
lower bound to start asking questions about interpretability.
