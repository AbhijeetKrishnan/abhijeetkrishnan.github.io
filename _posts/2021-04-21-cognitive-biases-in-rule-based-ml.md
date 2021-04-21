---
title: "Paper Summary - *A review of possible effects of cognitive biases on
interpretation of rule-based machine learning models* by Kliegr, Bahník and Fürnkranz (2021)"
tags: paper-summary biases xrl
mathjax: true
excerpt: "Summarizing and providing my notes on the paper - A review of possible effects of 
cognitive biases on interpretation of rule-based machine learning models by Kliegr, Bahník and 
Fürnkranz (2021)"
---

Kliegr, T., Bahník, Š., & Fürnkranz, J. (2021). A review of possible effects of cognitive biases on
interpretation of rule-based machine learning models. *Artificial Intelligence*, 103458.
[<i class="far fa-file-pdf"></i>](https://arxiv.org/pdf/1804.02969.pdf)
{: .notice--info}

## Paper Summary

This paper identifies various human cognitive biases from literature that could have an impact in
the interpretation of rule-based explanations of AI systems. For each bias identified, it provides an
explanation, its hypothesized impact on rule interpretation and a debiasing technique informed by
literature. The paper offers a set of recommendations for rule-learning algorithms and software to
help reduce the chances of users misinterpreting their rules. Finally, the paper concludes with a
discussion of its limitations and future work.

## Methodology

The authors conducted a literature review of cognitive science and rule-based learning research.
Using personal judgment, they identified the relevant biases and debiasing techniques, and
hypothesized the impact each bias might have had on the rule interpretation.

A summary of the full list of biases identified, as well as their impact on rule interpretation and
associated debiasing technique can be seen in the figure from the paper below.

![List of biases](/assets/images/kliegr_biases.jpg)

## Discussion

The authors extensively discuss the limitations of their work. First, the need for validation of
these biases through human-subject experiments. The biases have been identified as relating to the
interpretation of rules based on the authors' judgment. This needs to be experimentally validated.
There is also a lack of research in validating these biases in the context of rule interpretation.

Next, the authors discuss the effect of domain knowledge in interpreting rules, as well as
individual differences between users that can cause rules to be interpreted differently. They also
point to the lack of research on the efficacy of debiasing techniques and the need to explore them
further.

Lastly, they discuss extending the scope of this research beyond biases to misinterpretation arising
from cognitive load and remembrance. They also discuss applying these principles to forms of
explanation beyond logical rules.

## Impressions

This is a very well written paper, and a much needed one based on the lack of human-centred
evaluation of explanations I identified in previous papers I've summarized. It states its
assumptions and limitations very clearly, and is well-researched, with each bias and debiasing
technique drawing on a lot of relevant papers in cognitive science. The directions for future work
are analyzed well, and the biases themselves seem convincing in that I can visualize myself falling
for them when interpreting rules.

I see the logical next step for the paper to be investigating the effect of specific biases on rule
interpretation. For example, take the bias related to the misinterpretation of AND. The debiasing
technique suggested is to use AND to connect propositions and not categories (don't use it to say
Linda is a bank teller AND active in the feminist movement, say Linda is a bank teller and Linda is
active in the feminist movement). The effect of the bias and debiasing technique could be
quantitatively explored in a rule-based interpretation context. The same study methodology could be
applied to all biases and debiasing techniques presented in the paper. This would give us a insight
into which biases we should worry most about when designing explainability techniques for AI systems.