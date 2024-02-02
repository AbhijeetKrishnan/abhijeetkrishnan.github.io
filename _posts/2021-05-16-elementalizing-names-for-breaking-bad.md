---
title: Elementalizing Names for Breaking Bad 
tags: breaking-bad graphs maximum-matching 
mathjax: true 
excerpt: "Attempting to use bipartite graphs to help Breaking Bad with its unique opening
credits."
---

## Motivation

[Breaking Bad](https://en.wikipedia.org/wiki/Breaking_Bad) is one of the finest television shows I've
seen. I recently binge-watched the entire five seasons of the show. In doing so, I noticed the
unique presentation of the opening credits where cast members have an elemental symbol from the
periodic table highlighted in their name, reflecting the importance of chemistry in the show (and
also calling back to the design of the title). I wondered how the show runners might have done this,
and thought about writing a program to do this "elementalization" automatically.

<img src="/assets/images/elemental_cast.png" alt="Examples of the cast names with highlighted elements" width="50%" height="50%"/>

## Problem

From observing how these elements are highlighted in the cast names, I made the following
observations -

* every name gets exactly one element
* the position of the element doesn't matter (could be in the first name, last name etc.)
* the same element can be assigned to multiple cast members (e.g. in S2E7 'Negro y Azul', `Ar` is
  assigned to both actor Aaron Paul and producer Stewart A. Lyons)
* elements don't cross name boundaries (e.g. `Ar` cannot be assigned to Cynthia Rodgers)
* case doesn't matter - wherever the symbol gets assigned, the first letter is capitalized, and the
  rest are in lowercase

The problem then is to assign elemental symbols (e.g. `H`, `Li`, `Na`) to a list of cast member
names if that symbol exists in their name in a valid way, which matches my earlier observations.
Some names might have multiple possible assignments, and others might have none. The program should
return all possible valid assignments for each name.

## Solution

As presented, the problem is easy to solve. Simply iterate through the list of names and for each,
check if every possible elemental symbol is present in it and return that list. I manually compiled
a list of elemental symbols in the periodic table from [Ptable](https://ptable.com/), which you can
find [here](/assets/docs/elements.txt). Since case is unimportant, all the cast member and elemental
symbol names are preprocessed to be in lowercase for simplicity. There might be names in which an
element can be assigned to multiple places in the name, however I chose to leave this to the person
responsible for the credits. The relevant snippet in Python is given below -

```python
from typing import List, Dict
from collections import defaultdict

def assign_elements(names: List[str], elements: List[str]) -> Dict[str, List[str]]:
    assignment = defaultdict(list)
    for name in names:
        for element in elements:
            if element in name:
                assignment[name].append(element)
    return assignment
```

For the first-billed [cast](https://www.imdb.com/title/tt0959621/) from the Pilot, we get the
following output -

```bash
{
    'bryan cranston': ['b', 'c', 'n', 'o', 's', 'cr', 'br', 'y', 'ra'], 
    'anna gunn': ['n', 'na', 'u'], 
    'aaron paul': ['n', 'o', 'p', 'ar', 'au', 'pa', 'u'], 
    'dean norris': ['n', 'o', 's', 'i', 'no'], 
    'betsy brandt': ['be', 'b', 'n', 's', 'br', 'y', 'nd', 'ra', 'ts'], 
    'rj mitte': ['te', 'i'], 
    'max arciniega': ['c', 'n', 'ar', 'ni', 'ga', 'in', 'i'], 
    'john koyama': ['h', 'n', 'o', 'k', 'y', 'am'], 
    'steven michael quezada': ['h', 'c', 'n', 's', 'v', 'te', 'i', 'u'], 
    'marius stan': ['n', 's', 'ar', 'i', 'ta', 'u'], 
    'aaron hill': ['h', 'n', 'o', 'ar', 'i'], 
    'gregory chase': ['h', 'c', 'o', 's', 'as', 'se', 'y', 're'], 
    'carmen serano': ['c', 'n', 'o', 's', 'ar', 'ca', 'se', 'er', 'ra', 'no'], 
    'evan bobrick': ['b', 'c', 'n', 'o', 'k', 'v', 'br', 'i'], 
    'roberta marquez seret': ['be', 'b', 'o', 's', 'ar', 'se', 'er', 'ta', 're', 'u']
}
```

We could stop here. However, it was interesting to me to think about the additional constraint of
restricting an element from being used for more than one cast member. It might become repetitive if
the common symbols like `H`, `S` and `O` are reused. What would an algorithm to obey this additional
constraint look like?

## Bipartite Graphs and Maximum Matching

We cannot simply use a greedy algorithm of assigning the first available symbol to a name. Consider
two hypothetical names *Br* and *B*. *Br* can use the symbols `B` and `Br`, while *B* can only use
the symbol `B`. If we first assign `B` to *Br*, then we won't have any symbol for *B*, even though
we would have had one had we used `Br` in the first place. We could think of backtracking and trying
different assignments if we encounter an impossibility, but we wouldn't know if there actually is an
impossibility or not until we exhaust all options, which could take a *long* time.

We can instead model this problem using a *bipartite graph*. On one side, we have a set of nodes,
each containing the name of a cast member. On the other side, we have another set of nodes, each
containing an elemental symbol. Using the previous solution, we can construct edges between these
two sets of nodes, connecting a name to every elemental symbol that can be assigned to it. We need
to choose a set of edges (which connect a name to an elemental symbol which can be assigned to it)
such that no two edges coincide (because that means we are assigning two elements to the same name)
and every name-node lies on an edge in the selection (because we don't want to leave a name
unassigned if we can help it).

This is exactly the problem of finding a [*maximum
matching*](https://en.wikipedia.org/wiki/Maximum_cardinality_matching) in a bipartite graph. There
are many well-known algorithms to solve this. We will use the [Hopcroft-Karp
algorithm](https://en.wikipedia.org/wiki/Hopcroft%E2%80%93Karp_algorithm) as implemented in the
[`networkx`](https://networkx.org/documentation/stable/reference/algorithms/bipartite.html) package.

```python
from typing import List, Dict

import networkx as nx

def assign_unique_elements(names: List[str], elements: List[str]) -> Dict[str, str]:
    graph = nx.Graph()
    graph.add_nodes_from(names, bipartite=0)
    graph.add_nodes_from(elements, bipartite=1)
    graph.add_edges_from([(name, ele) for ele in elements for name in names if ele in name])
    return nx.bipartite.maximum_matching(graph, top_nodes=names)
```

The output using the cast from the Pilot again is -

```bash
{
    'gregory chase': 'h', 
    'aaron hill': 'n', 
    'anna gunn': 'na', 
    'bryan cranston': 'b', 
    'aaron paul': 'o', 
    'roberta marquez seret': 'be', 
    'dean norris': 's', 
    'evan bobrick': 'c', 
    'betsy brandt': 'br', 
    'steven michael quezada': 'v', 
    'max arciniega': 'ar', 
    'marius stan': 'i', 
    'carmen serano': 'ca', 
    'rj mitte': 'te', 
    'john koyama': 'k', 
}
```

I can imagine the show runners might want to focus the more "dramatic" elemental symbols (i.e. the
two-character ones) on the first-billed cast displayed first (e.g. assigning `Br` to *Bryan
Cranston* instead of *Betsy Brandt*). I can also imagine one might want to iterate through multiple
assignments to find something more appropriate than the first one returned (I don't know how the
minds of TV show creators work). Refer to
[Uno, T. (1997)](https://link.springer.com/chapter/10.1007/3-540-63890-3_11)
which proposes an algorithm for this, since it isn't directly implemented in `networkx`.

## Conclusion

The usage of a classic graph algorithm to solve a problem I noticed in the world was satisfying to
me. I can think of a few additional questions to ask like -

* Which cast member can be mapped on to the *most* elements? What about the *least* number of
  elements?
* What is the longest name which cannot be mapped onto elements? Is this the same in names across
  various cultures?
